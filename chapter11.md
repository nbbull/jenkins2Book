### 前言 {#前言}

TesterHome有人专门加了我QQ问安全测试这个话题，所以这篇准备先聊聊持续交付中的安全测试。  
现在信息安全已经上升到了国家战略的高度，特别是今年《中华人民共和国网络安全法》颁布后，用户隐私通过国家立法的方式被严格要求保护，另外一方面安全灰产行业风起云涌，形成了一个巨大的地下产业链条和破坏能力。在此背景下，越来越多的互联网公司也开始组建自己的安全职能部门，但也会发现很多公司的软件并没有经过专门的安全测试便运行在互联网上，它们携带着各类安全漏洞直接暴露在公众面前，其中一些漏洞甚至直指软件所承载的核心敏感信息或业务逻辑。这里既有企业安全意识的问题，也有安全人才缺乏的原因，目前软件测试团队大多数人的视野仍还停留在功能验收，或性能自动化等传统领域，对于安全测试，则往往会感觉水太深了不知从何下手。

### 互联网公司的安全团队 {#互联网公司的安全团队}

在软件测试的这些领域里，安全测试确实是一个比较特别的科目。在几年前，企业招聘安全人员一般都是放在运维部门，主要工作还是扫描和服务器加固等，搭建IDS/IPS/WAF等安全设备进行对抗，这些安全设备说起来很唬人，市场也卖的很好，但是如果脱离业务光依靠安全设备堆叠，真正挡住了多少黑客攻击？估计抓的最多的还是漫无目的的蠕虫。  
本人几年前有幸从无到有进行了安全团队的组建，从自己做渗透测试到现在逐渐组建了10人左右的专业安全团队，慢慢也接触和熟悉了这个方向，覆盖领域从应用安全逐渐扩展到运维安全、安全风控、安全合规等各个方面，并自主进行了WAF和IDS等系统的研发。  
这个方向确实有不少其独特的地方，但放下其他几个不表，至少在应用安全这个方向，“神秘的安全测试人员”（一般我们也喜欢招聘有渗透经验的白帽子们）不光是名字跟软件测试人员一样都有“测试”二字，所做的事情在本质上也是跟软件测试人员有很多相通。

### 持续交付中的安全测试 {#持续交付中的安全测试}

回到持续交付这个话题，在过往的软件研发过程中，安全测试（类似的也包括性能、APP专项等测试）由于其专业性，一般是作为软件开发的较末环节开始手工执行和验收。但持续交付/devops的大潮提高了速度并扩张了规模，让安全和性能等专项团队也面临着新的挑战。为确保快速开发和新功能部署，安全团队必须确保安全评估的频率，既要保证安全风险最小化，同时也要考虑安全团队有限资源的可持续性，权衡DevOps速度与现有安全要求的需求也在安全行业内催生了一个名为DevSecOps的模型（类似的还有DevTestOps的概念,不详细展述）。  
即使在持续集成（CI）和持续交付（CD）过程中还不能实现完整的自动化软件安全评估（人工的渗透测试依然不可缺少），但这个方向仍然有不少可以自动化实施的实践。其中自动化的静态代码扫描，依赖组件扫描以及初级的渗透测试都可以比较容易的在交付流水线中实现，参见下面截图（为了pipeline的运行效率，相互不依赖几个集成测试环节可以并发执行）  
[![](https://testerhome.com/uploads/photo/2017/5f105bac-d552-4d8a-95e2-c837b137567b.png!large)](https://testerhome.com/uploads/photo/2017/5f105bac-d552-4d8a-95e2-c837b137567b.png!large)

### 代码安全检查 {#代码安全检查}

利用静态代码扫描工具对代码在编译之前进行扫描，并在静态代码层面上发现各种问题，其中包括安全问题。部分工具列表：  
[![](https://testerhome.com/uploads/photo/2017/d6411dbf-d2e8-4150-8ffa-007f4aafe0f2.png!large)](https://testerhome.com/uploads/photo/2017/d6411dbf-d2e8-4150-8ffa-007f4aafe0f2.png!large)  
为了方便与sonarQube集成，我们使用的是FindbugSecurity,根据企业实际情况定制了规则，在持续构建的过程中，会进行代码静态安全检查。  
[![](https://testerhome.com/uploads/photo/2017/d3c540e8-5442-454a-8df1-82cdd1beebd7.png!large)](https://testerhome.com/uploads/photo/2017/d3c540e8-5442-454a-8df1-82cdd1beebd7.png!large)  
通过检查可以很快发现代码中sql注入，XSS、密码硬编码存储、不安全配置等问题。  
[![](https://testerhome.com/uploads/photo/2017/e37daa28-c4bd-4ecb-a67d-4341714569be.png!large)](https://testerhome.com/uploads/photo/2017/e37daa28-c4bd-4ecb-a67d-4341714569be.png!large)

pipeline中只需要集成sonarqube的代码检查即可，通过sonar的质量阈判断pipeline是否通过。

```groovy
stage('静态检查') {
           when { 
               expression
                   {return isCA} 
           }
           steps {
               echo "starting codeAnalyze with SonarQube......"
               //sonar:sonar.QualityGate should pass
               withSonarQubeEnv('SonarQube') {
                 //固定使用项目根目录${basedir}下的pom.xml进行代码检查  
                 sh "mvn -f pom.xml clean compile sonar:sonar"
               }
               script {
               timeout(10) { 
                   def qg = waitForQualityGate() 
                       if (qg.status != 'OK') {
                           error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
                       }
                   }
               }
           }
       }
```

### 依赖组件安全检查 {#依赖组件安全检查}

由于当前服务器应用依赖的第三方的库和框架越来越多、越来越复杂，比如SSL、Spring、Rails、Hibernate、.Net，以及各种第三方认证系统等。而且系统开发的时候一般选定某个版本后在很长一段时间内都不会更新，因为更新的成本一般都比较高。但是往往这些依赖为了添加新的功能和修复各种当前的问题——当然包括安全问题，却会经常更新。开源项目的安全问题只要被发现以后，通常都会被公布到网上去，比如CVE、CWE等，导致很多人都可能利用它去攻击使用这些依赖的系统。比如说这两年出尽风头的万年漏洞王 Struts2每次在0DAY漏洞爆发后，都造成了大量的网站沦陷，对整个互联网行业都造成了严重恐慌。  
依赖组件检查就是通过扫描当前系统使用到的所有第三方依赖，并和网上公布的安全漏洞库进行比较，如果当前某个第三方依赖存在某种危险级别（需要自己定义）的漏洞，就立即发出警告（比如通过pipeline阻止发布等）来通知开发人员或者系统管理员，从而在最短的时间内修复这个问题，防止攻击，避免或者减少损失。  
部分工具列表：  
[![](https://testerhome.com/uploads/photo/2017/d18fe027-dc54-44b7-a9d4-1514aba8c788.png!large)](https://testerhome.com/uploads/photo/2017/d18fe027-dc54-44b7-a9d4-1514aba8c788.png!large)

我们使用Dependency-Check来做组件检查，先在Jenkins中安装OWASP Dependency-Check Plugin插件，并在pipiline中增加对应stage

```groovy
stage('依赖安全检查') {
    when { 
        expression
            {return isDC} 
    }
     steps{
          //指定检测**/lib/*.jar的组件
         dependencyCheckAnalyzer datadir: '', hintsFile: '', includeCsvReports: false, includeHtmlReports: false, includeJsonReports: false, isAutoupdateDisabled: false, outdir: '', scanpath: '**/lib/*.jar', skipOnScmChange: false, skipOnUpstreamChange: false, suppressionFile: '', zipExtensions: ''
        //有高级别组件漏洞时，fail掉pipeline
        dependencyCheckPublisher canComputeNew: false, defaultEncoding: '', failedTotalHigh: '0', healthy: '', pattern: '', unHealthy: ''

     }
 }
```

组件检查结果如下  
[![](https://testerhome.com/uploads/photo/2017/8ede0fff-7a91-43d9-909f-336af151c4eb.png!large)](https://testerhome.com/uploads/photo/2017/8ede0fff-7a91-43d9-909f-336af151c4eb.png!large)  
重点关注存在高危漏洞的组件  
[![](https://testerhome.com/uploads/photo/2017/49cffe55-2dc9-4625-87b0-02f0a4578fcd.png!large)](https://testerhome.com/uploads/photo/2017/49cffe55-2dc9-4625-87b0-02f0a4578fcd.png!large)  
安全漏洞详情  
[![](https://testerhome.com/uploads/photo/2017/8f4a8041-d072-4f4b-8b1a-63e1589ff21e.png!large)](https://testerhome.com/uploads/photo/2017/8f4a8041-d072-4f4b-8b1a-63e1589ff21e.png!large)  
补充：由于与国外网络的访问不稳定，建议在本地搭建 NVD 镜像（美国国家信息安全漏洞库）并定期与官方库进行同步。

### 安全渗透扫描 {#安全渗透扫描}

此类方法一般适用在web安全测试领域，针对Web应用的安全扫描工具非常多，其中OWASP ZAP是免费软件里面最为常用的。部分工具列表：  
[![](https://testerhome.com/uploads/photo/2017/d8d8b691-6904-498f-9906-d08167681784.png!large)](https://testerhome.com/uploads/photo/2017/d8d8b691-6904-498f-9906-d08167681784.png!large)  
web安全扫描一般分为两种类型：主动扫描和被动扫描。  
**主动扫描**  
主动扫描是首先给定需要扫描的系统地址，扫描工具通过某种方式访问这个地址，如使用各种已知漏洞模型进行访问，并根据系统返回的结果判定系统存在哪些漏洞；或者在访问请求中嵌入各种随机数据（模糊测试）进行一些简单的渗透性测试和弱口令测试等。主动扫描覆盖测试面会比较大，但缺点是完成一次全面扫描非常耗时\(一般都需要几个小时\)，所以我们没有选用这种方式在pipeline中进行集成，更合适的方式可能还是搞个job在半夜的时候做定时扫描然后第二天来查看扫描报告。  
**被动扫描**  
被动扫描的基本原理就是设置扫描工具为一个Proxy Server，功能测试通过这个代理服务访问系统，扫描工具可以截获所有的交互数据并进行分析，通过与已知安全问题进行模式匹配，从而发现系统中可能的安全缺陷。在实践中，为了更容易地集成到CI，如果有比较完善的Web自动化测试用例，一般会在运行自动化功能测试的时候使用被动扫描方法，从而实现持续安全扫描。  
[![](https://testerhome.com/uploads/photo/2017/7a2e8143-506d-4e65-b214-437b32ead12c.png!large)](https://testerhome.com/uploads/photo/2017/7a2e8143-506d-4e65-b214-437b32ead12c.png!large)  
被动扫描的具体实现也非常简单，在下载安装好ZAP后，后续的过程都可以在pipeline上进行组织。  
1.配置WebDriver,为其设置代理到ZAP的地址  
2.启动ZAP并运行web自动化测试：zapStart build -Dzap.proxy=localhost:7070，执行web自动化测试脚本  
3.生成安全测试报告：zapReport  
4.关闭ZAP：zapStop  
_备注：如果要执行主动扫描可以使用命令“zapStart zapScan”，在执行过程中使用“zapScanStatus”查看状态，当扫描完成后，生成安全扫描报告并关闭ZAP“zapReport zapStop”。_  
需要先在gradle脚本里配置好security-zap插件：[https://github.com/wmaintw/security-zap](https://github.com/wmaintw/security-zap)  
安全测试报告：  
[![](https://testerhome.com/uploads/photo/2017/6cded446-6614-4aad-80fb-1ed11b1b8b58.png!large)](https://testerhome.com/uploads/photo/2017/6cded446-6614-4aad-80fb-1ed11b1b8b58.png!large)

相对于代码安全检查和依赖组件检查，安全渗透扫描适用的场景较少，在web自动化用例不是太丰富的情况效果比较有限，不过还是可以作为一个安全渗透的冒烟测试，在专业渗透测试工程师资源有限的情况下提供一个安全基线的检查。

### 结语 {#结语}

安全测试是个非常复杂的过程，依靠自动化扫描器并不能发现所有的安全问题，但是它可以在较小投入的情况下持续发现大部分系统的基础安全问题。如果需要更高级别的安全保障，人工渗透性测试和威胁建模等必不可少，但成本也是相对较高的。  
安全测试绝不应该是测试工程师的禁区，不管是在测试思想还是工程实践上，安全测试都脱离不了持续交付和敏捷的软件工程体系。相信未来安全测试，也肯定会和软件开发测试的过程结合的越来越紧密，而不再是现在这样只限于白帽子这个小众的圈子。  
所以这里致所有测试同仁们，让我们也开始做安全测试吧。

