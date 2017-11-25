### 前言 {#前言}

公司此前用的一直是的SonarQube5.1（2015年版本，为兼容jdk6和jdk7的项目一直没有升级），最近为了pipeline的集成刚刚升级到了最新的SonarQube6.5版本。  
网上对SonarQube6的介绍比较少，这里重点先介绍下SonarQube6以后的一些新增特性。  
1.代码问题重新分级，将问题分为bug、漏洞、坏味道；将代码检查结果从可靠性、安全性、可维护性几个角度进行问题分类和风险分级。  
2.更丰富的代码检查规则，更友好的问题处理曲线展示，更清晰的质量阈和代码规则定制。  
3.支持webhook功能，可与jenkins等持续集成平台完美对接，在检查完后通过webhook的方式实时反馈检查结果，控制pipeline状态\(**划重点：为devops量身定制的特性**）。  
4.不再支持原来需要输入系统用户名密码，数据库用户密码的认证方式，统一修改成token方式，参数配置更简便也更安全。  
5.横向扩展的能力以及更快的代码检查速度。  
从总体发展趋势来说，除了代码检查能力以外，对devops的支持能力正成为SonarQube升级的一个重大方向。个人认为不足之处是对multi-maven项目的支持还有待加强，踩坑不少。

### SonarQube简介 {#SonarQube简介}

SonarQube是一个用于代码质量管理的开源平台，用于管理Java源代码的质量。通过插件机制，SonarQube可以集成不同的测试工具，代码分析工具，以及持续集成工具，比如pmd-cpd、checkstyle、findbugs、Jenkins。通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。同时 SonarQube还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用 SonarQube。此外，SonarQube的插件还可以对 Java 以外的其他编程语言提供支持，对国际化以及报告文档化也有良好的支持。  
[![](https://testerhome.com/uploads/photo/2017/7b58815c-33de-4567-82fc-09c526aa8633.png!large)](https://testerhome.com/uploads/photo/2017/7b58815c-33de-4567-82fc-09c526aa8633.png!large)

### SonarQube安装 {#SonarQube安装}

SonarQube安装和使用这块，网上介绍的文档已经非常多了，这里不再作为重点进行赘述.  
新使用的同学可以参考官方安装指导手册：[https://docs.sonarqube.org/display/SONAR/Installing+the+Server](https://docs.sonarqube.org/display/SONAR/Installing+the+Server)  
对于像我这样的SonarQube老用户来说，升级的过程倒是有必要特别提下，SonarQube最近有两个里程碑版本： 4.5.7 LTS和 5.6 LTS，这两个版本均做了大的重构，如果小于该版本号的在升级时注意必须先升级到该LTS版本，再往上继续升级。  
Example 1 : 4.2 -&gt; 6.5, migration path is 4.2 -&gt; 4.5.7 LTS -&gt; 5.6 LTS -&gt; 6.5  
Example 2 : 5.1 -&gt; 5.6, migration path is 5.1 -&gt; 5.6

### 与LDAP集成 {#与LDAP集成}

如果公司有LDAP统一用户认证授权系统，在SonarQube上进行相关集成是非常有必要的。  
1.安装ladp插件，入口：Administration &gt; System &gt; Update Center，找到ladp plugin安装即可。  
2.配置ladp，vi conf/sonar.properties，增加ldap相关的配置参数  
[![](https://testerhome.com/uploads/photo/2017/66c9ce6c-5981-4fe9-83a7-ac0ac48e58a7.png!large)](https://testerhome.com/uploads/photo/2017/66c9ce6c-5981-4fe9-83a7-ac0ac48e58a7.png!large)  
3.重启sonar生效

### 命令行方式运行 {#命令行方式运行}

**1.分析Maven项目**  
maven全局设置,修改setting.xml配置

```
<settings>
    <pluginGroups>
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    <profiles>
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <!-- Optional URL to server. Default value is http://localhost:9000 -->
                <sonar.host.url>
                  http://myserver:9000
                </sonar.host.url>
            </properties>
        </profile>
     </profiles>
</settings>
```

执行maven指令：mvn clean verify sonar:sonar -Dsonar.login=\[my analysis token\] -Dsonar.java.binaries=\[basedir\]  
sonar.java.binaries等属性也可配置在项目的pom.xml或构建机器的setting.xml里。  
**2.分析Gradle项目**  
配置gradle全局属性~/.gradle/gradle.properties.

```
systemProp.sonar.host.url=http://localhost:9000
```

gradle clean compileJava sonarqube -x test -Dsonar.projectKey=\[my analysis token\] -Dsonar.java.binaries=\[basedir\]  
sonar.java.binaries等属性也可配置在项目的build.gradle或构建机器的gradle.properties里。  
详见官方文档:  
maven:[https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven)  
gradle:[https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Gradle](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Gradle)

### 与Eclipse集成 {#与Eclipse集成}

**1.安装sonar的eclipse插件**  
点击Help -&gt; Install New Software,将弹出Install对话框。 复制地址[http://dist.sonar-ide.codehaus.org/eclipse/](http://dist.sonar-ide.codehaus.org/eclipse/)到Work with栏并回车，将显示可用的插件和组件列表，如下图  
[![](https://testerhome.com/uploads/photo/2017/04f6bcbb-a09d-4672-bdc4-f6a90894295f.png!large)](https://testerhome.com/uploads/photo/2017/04f6bcbb-a09d-4672-bdc4-f6a90894295f.png!large)  
**2.配置eclispe的本地/远程Sonar服务器**  
点击Window-&gt;Preferences-&gt;SonarQube-&gt;Servers  
预置的访问本地Sonar服务器的地址为[http://localhost:9000/](http://localhost:9000/)，你可以修改、删除或者新增一个地址  
[![](https://testerhome.com/uploads/photo/2017/91ea7912-3ee6-47d6-aacf-63897ef8896e.png!large)](https://testerhome.com/uploads/photo/2017/91ea7912-3ee6-47d6-aacf-63897ef8896e.png!large)  
**3.链接你的项目**  
Sonar服务器配置完成后，下一步是将你的Eclipse项目链接到Sonar服务器，并利用Sonar服务器进行分析。  
首先，在Project Explorer中右键单击项目，然后点击Configure-&gt; Associate with SonarQueb.  
若项目在sonar中从未运行和创建，跳过这一步。  
最后执行mvn clean verify sonar:sonar 运行即可  
[![](https://testerhome.com/uploads/photo/2017/45231500-cbf1-40b0-9775-ac4b54106a3e.png!large)](https://testerhome.com/uploads/photo/2017/45231500-cbf1-40b0-9775-ac4b54106a3e.png!large)  
参考链接：[http://docs.sonarqube.org/display/SONAR/SonarQube+in+Eclipse](http://docs.sonarqube.org/display/SONAR/SonarQube+in+Eclipse)

### 与Jenkins Job集成 {#与Jenkins Job集成}

在jenkins的插件管理中选择安装sonar jenkins plugin，该插件可以使项目每次构建都调用sonar进行代码度量。  
进入配置页面对sonar插件进行配置，如下图，sonar6只需要配置token即可。  
[![](https://testerhome.com/uploads/photo/2017/e0531e7c-1d01-46b8-b39c-012d5c92a4eb.png!large)](https://testerhome.com/uploads/photo/2017/e0531e7c-1d01-46b8-b39c-012d5c92a4eb.png!large)  
创建静态代码检查的job，以maven项目为例，执行mvn clean verify sonar:sonar即可执行sonar代码检查，完成后对应的job里会有链接直接跳转到sonar的检查结果。  
[![](https://testerhome.com/uploads/photo/2017/745ac39e-0b6f-4468-8850-8a27605f84f2.png!large)](https://testerhome.com/uploads/photo/2017/745ac39e-0b6f-4468-8850-8a27605f84f2.png!large)  
参考链接：[https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins)

### 与Jenkins Pipeline集成 {#与Jenkins Pipeline集成}

**1.在pipeline中执行代码检查**  
SonarQube 5.2版本以后，开始支持与jenkins pipeline的集成，从而正式融入到以jenkins pipeline为核心的持续交付工具链.  
使用pipeline中的“withSonarQubeEnv”块可以选择jenkins中配置好的的SonarQube server，前面的pipeline文章的项目样例里其实已经有相关代码，再次举例如下

```groovy
withSonarQubeEnv('SonarQube') {
   //固定使用项目根目录${basedir}下的pom.xml进行代码检查
   sh "mvn -f pom.xml clean compile sonar:sonar"
 }
```

**2.sonar质量阈未通过时中止pipeline**  
SonarQube 6.2版本以后开始支持webhook功能，利用此特性可以与jenkins完美结合，在检查完成后通过异步回调的方式告知jenkins检查结果，从而让jenkins可以控制pipeline执行状态。  
sonarqube需要先配置webhook地址：\[Your Jenkins instance\]/sonarqube-webhook/，然后编写jenkins pipeline脚本如下。

```groovy
stage('静态检查') {
           steps {
               echo "starting codeAnalyze with SonarQube......"
               //sonar:sonar.QualityGate should pass
               withSonarQubeEnv('SonarQube') {
                 //固定使用项目根目录${basedir}下的pom.xml进行代码检查
                 sh "mvn -f pom.xml clean compile sonar:sonar"
               }
               script {
               timeout(10) { 
                   //利用sonar webhook功能通知pipeline代码检测结果，未通过质量阈，pipeline将会fail
                   def qg = waitForQualityGate() 
                       if (qg.status != 'OK') {
                           error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
                       }
                   }
               }
           }
       }
```

### 一些问题和解决方案 {#一些问题和解决方案}

**问题1：maven版本支持**  
From maven-sonar-plugin 3.1, Maven &lt; 3.0 is no longer supported.If using Maven prior to 3.0, you should use maven-sonar-plugin 3.0.2.  
maven-sonar-plugin3.1以后，sonar分析只支持maven3版本，maven2不再支持。  
**问题2：sonar在创建项目时，不允许maven项目里的module被重复执行代码检查【sonar5】。**  
\[ERROR\] Module "com.greenline.eops:greenline-eops-biz" is already part of project "com.greenline.eops:greenline-eops"，出现此问题的原因有几个  
1.不同的项目存在重名的module  
比如说项目中的module是从其他项目中copy，pom.xml里的artifactId等坐标标识未作更新。  
**解决方案：**  
这类项目需要强制开发修改名称，互相不要重名。  
2.multi-module项目里多个pom.xml重复引用相同的module  
某个multi-module项目的目录结构如下

```
--base  pom.xml
--biz   pom.xml
--web   pom.xml
--hessian   pom.xml
--mq-client   pom.xml
```

在主项目下，存在web,hessian,mq-client等多个module子项目，每个module子项目里都会引用biz模块，base目录里放的是parent pom.xml  
当web在sonar运行过代码检查后，再运行hessian,因为biz里的module已被运行过，就会出现此类错误。  
**解决方案：**  
i.在multi-module项目的项目根目录${basedir}下，放置parent pom.xml,并引用所有子module。（推荐）

```
--biz   pom.xml
--web   pom.xml
--hessian   pom.xml
--mq-client   pom.xml
pom.xml
```

ii.mvn命令运行sonar时增加--project !module参数，过滤已被运行过的module。（不建议使用）  
**问题3：还是multi-module项目的问题，parent module因找不到sonar.java.binaries里的编译文件报错\[升级sonar6以后的坑\]**  
sonar6为了提升检测速度，在检测前会检查所有module的sonar.java.binaries里定义的编译文件是否存在（默认是/target/classes），parent层因为没有实际项目代码也就不会产生class编译文件检查抛错。  
sonar社区很早就有人对此机制提出异议，搞不懂为啥现在还没有解决，sonarqube对multi-module的支持确实比较糟糕。  
[http://sonarqube-archive.15.x6.nabble.com/Maven-Build-Fails-when-no-files-in-target-classes-directory-td5030353.html](http://sonarqube-archive.15.x6.nabble.com/Maven-Build-Fails-when-no-files-in-target-classes-directory-td5030353.html)  
[![](https://testerhome.com/uploads/photo/2017/26285d42-3927-4d3c-b11a-5733a2566b8c.png!large)](https://testerhome.com/uploads/photo/2017/26285d42-3927-4d3c-b11a-5733a2566b8c.png!large)  
**解决方案：**  
目前我的解决办法是把sonar.java.binaries配置成项目根目录${basedir}，sonar会自动遍历获取根目录下所有class文件生成分析结果。不过这种方式对jacoco和junit的报告文件获取有一定影响（又是一个坑）,大家有更好的解决方案也可以和我讨论。  
为减少对项目的侵入，可在编译机里的maven全局配置setting.xml里设置，也可通过命令参数传入，配置如下：

```
<sonar.java.binaries>${basedir}</sonar.java.binaries>
```



