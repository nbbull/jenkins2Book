### 前言 {#前言}

这半个月基本都在出差以及各种公司业务上的事情，难得有空闲整理一些测试技术上的事情。周末有些空闲码一篇填下坑，持续交付/持续集成这一系列文章不仅仅是想在坛子里和同行者做些分享，对个人也是一种自我思考和鞭策。总体来说我觉得这个论坛氛围目前还比较清爽，希望在人气快速提升的同时能保持初心，坚持做一个单纯技术分享交流的平台。

### 分层的自动化测试 {#分层的自动化测试}

5~10年前，我们接触的自动化测试更关注的是UI层的自动化测试，Mercury的WinRunner/QTP是那个时代商业性自动化测试产品的典型代表，在那个时代大家单纯想的都是能用一个自动化操作的工具替代人力的点击，商业化或是私有化框架大行其道。  
而分层的自动化测试倡导产品的不同阶段（层次）都需要自动化测试。在《google软件测试之道》中，在google 70%的投入为单元测试（小型测试），20%为接口/集成测试（中型测试），10%为UI层的自动化测试（大型测试），也就是大家熟悉的金字塔模型，越往上自动化实现难度越大，投入产生的收益也越低（需要强调的是，UI层的自动化测试作为最接近用户操作的测试，仍然有其存在的意义和场景）。  
[![](https://testerhome.com/uploads/photo/2017/21a0e5f3-d217-4145-9f4c-069c3d193e71.png!large)](https://testerhome.com/uploads/photo/2017/21a0e5f3-d217-4145-9f4c-069c3d193e71.png!large)

### 接口测试的意义 {#接口测试的意义}

接口测试是验证两个或多个模块应用之间的交互（通常是采用接口的方式），测试的重点是要检查数据的交换，传递和控制管理过程，还包括处理的次数。  
接口测试的核心战略在于：以保证系统的正确和稳定为核心，以持续集成为手段，提高测试效率，提升用户体验，降低产品研发成本。  
接口测试要为代码的编写保驾护航，增强开发人员和测试人员的自信，让隐含的 BUG提前暴露出来，要让开发人员在第一时间修复 BUG，要让业务测试人员在测试的时候更加顺手，最大限度得减少底层 BUG 的出现数量，要让产品研发的流程更加敏捷，要缩短产品的研发周期，最后在产品上线以后，要让用户用得更加顺畅，要让用户感觉产品服务零缺陷。  
不同于单元测试，接口测试本质上还是一种黑盒的测试，所以非常适合专职测试工程师去参与和覆盖。

### 接口测试框架选型 {#接口测试框架选型}

1.目前接口测试框架的选型，最常见的方法是采用jmeter,soapUI,postman,robotframework等UI化的接口测试框架来做。  
好处是业务测试人员可以不用或很少写测试代码，入门门槛低，前几年有很多公司都曾经开发过类似的测试框架，有前端有后端，专职的测试开发人员维护，业务测试人员只需要知道怎么操作而不需要参与具体coding。  
这种方法看起来非常高大上，但实际的问题是执行过程中主要的工作变成了测试框架的维护，非常依赖专职测试开发人员的设计和开发能力，每增加一种新的接口协议（比如dubbo、hessian或者内部自定义的协议）就需要在框架上增加支持；更致命的是一旦核心测试开发人员出现流动，就很容易造成整个接口测试体系的崩塌；另外对业务测试人员的技能成长也并不公平，个人已面试过太多只会使用某大公司XXX测试框架却完全不了解具体实现方式的工程师。  
《google软件测试之道》中早已有过预言，保密和私有化的基础测试设施并不能获得想象中的好处，这种方式意味着昂贵和迟缓，即使在公司内部的不同项目之间也很难做到复用。未来的测试基础设施必然是建立在共享代码和开源框架的基础上，测试开发人员需要更多的利用开源项目并为之贡献。  
最近重读了一次这本四五年前几乎改变软件测试行业的书籍，发现里面的预言都是如此准确，当然也可以认为国内整个行业都正参照google的方式在进行演变。  
[![](https://testerhome.com/uploads/photo/2017/6bc1be67-b7b7-41d9-8606-52cf3a28c05d.png!large)](https://testerhome.com/uploads/photo/2017/6bc1be67-b7b7-41d9-8606-52cf3a28c05d.png!large)  
2.使用junit、testng等java接口框架，直接编写测试代码去测试，同时对一些重复性的工作抽象建立基础库或方法。  
有点类似于单元测试，这种方法扩展性好实现灵活，作为程序员可以用代码实现灵活的场景组织和功能，只要稍微二次开发一下，但需要测试工程师有一定的编码基础。　  
这种方式在前几年实施的难度还是比较大，因为在市场上要找到懂java代码的测试工程师都寥寥无几，但在对测试工程师开发能力要求越来越多的今天实施难度已没有想象中困难，java/python等语言的编码能力也已成为我们团队招聘时的基本要求。  
另外提下，这里使用java而不用其他语言的原因，主要是团队的技术储备java是强项,拥有丰富的开源测试库，而且一般互联网公司的产品基本都是采用java框架进行开发，和开发团队技术栈保持一致非常有必要性。

### 我们封装的接口测试框架 {#我们封装的接口测试框架}

有很多公司做了各种不同的接口框架，都是基于自己公司的业务基础设计开发。我们基于自己的业务特点（为避免广告嫌疑，尽量我不提及具体公司的信息），也封装了自己的接口测试框架gtest-framework，在开发人员的单元测试中也正逐渐使用。  
gtest-framework要做的事情：  
1.前置数据准备和自动清理。  
2.常见接口协议的实现和封装。  
3.依赖注入配置方式的支持。  
4.如文件、图片、xml、字符等各类通用处理方法的集成。  
5.断言方式的扩展等。  
[![](https://testerhome.com/uploads/photo/2017/030e92d1-1c89-4324-bc00-0a8fd1c755e3.png!large)](https://testerhome.com/uploads/photo/2017/030e92d1-1c89-4324-bc00-0a8fd1c755e3.png!large)

### 接口测试关键实践 {#接口测试关键实践}

**1.数据准备**  
接口测试的数据准备，一般是指数据库的数据准备，有时候还包括文件和缓存的数据准备。具体实现可以从下面几个方面去考虑  
（1）硬编码的方式准备测试数据，在写测试代码的时候，使用到什么数据就插入什么数据。为了避免数据重复，很多人会习惯于使用随机字符或随机数（这种方法可能造成测试用例不稳定，尽量避免）。  
（2）可以直接通过调用其他API的方式准备测试数据，这种情况在测试最上层服务的时候比较有用，比如测试购买商品，就需要准备要购买的商品数据，购买商品的用户数据，这个时候，可以直接调用生成商品的api和生成用户的api直接生成测试数据。此方法实现简单，但前提是需要具备相应的api并且此api功能正确。  
（3）使用excel或xml准备测试数据，这种准备测试数据的方式，主要针对对象数据的准备，比如可以将一条商品数据对应excel中的一条数据，因为一般开发都会使用pojo映射，而在准备测试数据的时候，这些pojo对象属性的设置往往是重复和大工作量的，用excel或XML方式准备，则可以减少在代码当中重复去准备这些数据。  
一般我们使用的是2/3两种方式，其中3这种方式主要利用dbunit、spring-test、unitils等测试框架的特性经二次开发增加自定义注解，很轻松的导入excel或xml格式的文件并在测试完成后对数据进行自动回滚。

```java
/** 
* @ClassName: TestJdbcDataSet 
* @Description: 采用自定义TestDataSet注解方式准备测试数据，推荐。
* @author Cay.Jiang   
* @date 2017年7月10日 上午9:10:29 
*  
*/
public class TestJdbcDataSet extends BaseCase{
    Map<String, Object> args = new HashMap<String, Object>();
    @Test
    @TestDataSet(locations={"/tmp/domaininfo.xls"},dsNames={"mysqlDataSource"})
    public void test01_mysql(){
        args.put("selfdomain", "baidupc2");
        List<Map<String, Object>> result=JdbcUtil.queryData(mysqlJdbcTemplate, "domaininfo", args);
        System.out.println(result);
        assertEquals("合作商接入名称",result.get(0).get("remark"));
    }
}
```

上面代码中的/tmp/domaininfo.xls参见：domaininfo.xls ，其中Excel格式以Sheet名为表名，第一行定义了字段名称，其余行为对应的数据。  
[![](https://testerhome.com/uploads/photo/2017/b06e7a03-aa15-4370-a668-b07f52b757d5.png!large)](https://testerhome.com/uploads/photo/2017/b06e7a03-aa15-4370-a668-b07f52b757d5.png!large)  
多数据集：  
[@TestDataDataSet](https://testerhome.com/TestDataDataSet)\(locations={"Data1.xls","Data2.xls"},dsNames={"dsNameA","dsNameB"}\)，Data1.xls的数据会插入dsNameA所指的数据库中，Data2.xls的数据会插入dsNameB所指的数据库中  
**2.断言**  
常见的断言方式有JUnit自带的Assert和Hamcrest。JUnit自带的断言方法功能十分有限只能满足最基本的需求。Hamcrest相对来讲功能丰富一些，但是该库已经多年不更新。而且Hamcrest和JUnit自带的断言方法一样，有个致命的缺点，就是当一个case中有多个断言时，如果其中一个断言失败，那么在它之后的断言都不会执行。这里向大家推荐一款新的断言神器AssertJ。  
AseertJ: 号称流式断言。什么是流式，常见的断言器一条断言语句只能对实际值断言一个校验点，而AseertJ支持一条断言语句对实际值同时断言多个校验点，这样使得断言的语句更加简洁适合阅读。AseertJ还支持一次性执行所有断言，然后收集所有失败的断言一起反馈。当然除此之外AseertJ还有很多其他特性，可以参考官方文档慢慢挖掘。下面将举例说明一下AseertJ的优势：

```java
public class TestCase extends BaseCase{
    UserProfileBO user = new UserProfileBO();
    @Before
    public void init(){
        user.setAddress("杭州");
        user.setMobile("1386800000");
        user.setUserName("测试账号");
    }

    /*
     * JUnit 内置的断言
     * 
     * 1、其中一个断言失败后，后面所有断言将不会执行。
     * 2、支持的断言方法较少
     * 
     */
    @Test
    public void testAssertJUnit(){

        Assert.assertEquals("地址",user.getAddress(),"宁波");
        Assert.assertEquals("手机",user.getMobile(),"13868000000");
        Assert.assertEquals("手机",user.getUserName(),"测试");

        Assert.assertNotNull(user.getMobile());
        Assert.assertTrue(user.getMobile().startsWith("138"));
        Assert.assertTrue(user.getMobile().length() == 11);

    }

    /*
     * Hamcres 断言
     * 
     * 1、其中一个断言失败后，后面所有断言将不会执行。
     * 2、支持的断言方法丰富，但是已经多年不更新。
     * 
     */
    @Test 
    public void testHamcrestMatchers() {  

        MatcherAssert.assertThat(user.getAddress(), equalTo("宁波"));  
        MatcherAssert.assertThat(user.getMobile(), equalTo("13868000000"));  
        MatcherAssert.assertThat(user.getUserName(), equalTo("测试"));  

        MatcherAssert.assertThat(user.getMobile(), allOf(is(nullValue()),startsWith("136")));  
    }

    /*
     * AssertJ 断言
     * 
     * 1、支持所有断言执行后，失败断言统一反馈。
     * 2、支持的断言方法丰富。
     * 3、支持流式断言，方便阅读。
     * 
     */
    @Test
    public void testAssertJ(){

        //断言集合，执行所有断言后，失败断言统一反馈。
        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(user.getAddress().equals("宁波"));
            softly.assertThat(user.getMobile().equals("13868000000"));
            softly.assertThat(user.getUserName().equals("测试"));
        });

        //流式断言
        Assertions.assertThat(user.getMobile())
            .isNotNull()
            .startsWith("136")
            .hasSize(11);

    }
}
```

**3.jenkins集成接口测试**  
（1）设置测试代码的仓库地址及身份信息  
[![](https://testerhome.com/uploads/photo/2017/a58f0249-37ef-4a44-81e3-bd347bf95827.png!large)](https://testerhome.com/uploads/photo/2017/a58f0249-37ef-4a44-81e3-bd347bf95827.png!large)  
（2）设置maven运行参数  
希望执行部分接口用例，可以通过-Dtest=XXX\(测试类名\)的方式执行指定的case，多个类名用逗号“，”隔开  
[![](https://testerhome.com/uploads/photo/2017/414e7ed0-7bda-4867-8463-2090e4a9f0e9.png!large)](https://testerhome.com/uploads/photo/2017/414e7ed0-7bda-4867-8463-2090e4a9f0e9.png!large)  
（3）设置Job执行机制，下图表示每天定时执行  
[![](https://testerhome.com/uploads/photo/2017/000f0f4a-d6b4-4af2-ade8-99e52647ab29.png!large)](https://testerhome.com/uploads/photo/2017/000f0f4a-d6b4-4af2-ade8-99e52647ab29.png!large)  
**4.pipeline参数注入**  
前面写pipeline的时候提过，我们的接口测试代码是需要支持外部参数注入的，比如测试的服务地址，不同的分支代码可能部署在不同测试服务器上，需要在pipeline中通过参数化的方式驱动对不同的服务器进行接口测试。  
这里我们可以使用maven的-D（Properties属性）来实现，举例如下：  
\(1\)dubbo使用properties配置文件，但具体参数使用${key}占位符方式打包替换  
[![](https://testerhome.com/uploads/photo/2017/d2be473f-bdf4-4752-961e-caa45b4dba6e.png!large)](https://testerhome.com/uploads/photo/2017/d2be473f-bdf4-4752-961e-caa45b4dba6e.png!large)  
\(2\)maven的pom文件中指定对应配置文件中的参数值\(此处指定的参数值会被通过maven -D传递过来的参数值覆盖\)  
[![](https://testerhome.com/uploads/photo/2017/f7cb32ba-8464-4921-9482-0fa900fc0691.png!large)](https://testerhome.com/uploads/photo/2017/f7cb32ba-8464-4921-9482-0fa900fc0691.png!large)  
此处注意：还需要启动resources的filter过滤器  
\(3\)使用maven命令行设置属性值  
[![](https://testerhome.com/uploads/photo/2017/cf4bd7e5-098f-41b6-869b-1356c63fcb65.png!large)](https://testerhome.com/uploads/photo/2017/cf4bd7e5-098f-41b6-869b-1356c63fcb65.png!large)  
并对该值进行参数化支持pipeline传参  
[![](https://testerhome.com/uploads/photo/2017/d40074e0-f6be-43f1-b01a-b59ff49f1fee.png!large)](https://testerhome.com/uploads/photo/2017/d40074e0-f6be-43f1-b01a-b59ff49f1fee.png!large)  
**5.pipeline代码实现**

```groovy
stage('接口自动化测试') {
      steps{
          echo "starting interfaceTest......"
          script {
           //为确保jetty启动完成，加了一个判断，确保jetty服务器启动可以访问后再执行接口层测试。
           timeout(5) {
               waitUntil {
                  try {
                      //确保jetty服务的端口启动成功
                      sh "nc -z ${serverIP} ${jettyPort}"
                      //sh "wget -q http://${serverIP}:${jettyPort} -O /dev/null"
                      return true
                  } catch (exception) {
                      return false
                      }
                  }
              }
          //将参数IP和Port传入到接口测试的job，需要确保接口测试的job参数可注入
           build job: ITEST_JOBNAME, parameters: [string(name: "dubbourl", value: "${serverIP}:${params.dubboPort}")]
          }
      }
  }
```

### 测试代码规范\(仅供参考\) {#测试代码规范(仅供参考)}

1.测试项目命名规范  
接口测试：  
一般需要独立测试项目，测试项目的命名规则为：“test-“+被测试的项目名，如test-kano  
单元测试：  
不需要重建独立测试项目，和开发代码放在同一项目即可。  
2.测试目录定义规范  
测试代码统一放在测试项目的“src/test/java”下。  
测试配置文件统一放置在“src/test/resources”下。  
3.包名定义规范  
与被测试项目中的包名一致  
4.测试类命名规范  
测试类的命名规则是：以Test开头，以它要测试的对象的名称结尾，例如  
Test+被测试的业务、Test+被测试的接口、Test+被测试的类  
另外一种方式是：以Test结尾，以它要测试的对象的名称开头，例如  
被测试的业务+Test、被测试的接口+Test、被测试的类+Test  
视个人习惯而定，为了case定位方便，目前测试团队一般用第一种。  
5.测试用例命名规范  
测试用例的命名规则是：test+用例操作\_条件状态，统一使用lowerCamelCase风格，必须遵从驼峰形式。  
单词的约定与测试类命名同  
6.接口测试代码常见约束  
（1）数据清理和构造  
[@BeforeClass](https://testerhome.com/BeforeClass)[@Before](https://testerhome.com/Before)中做数据准备等相关操作  
--加载测试类以前需要加载所有测试用例共同的场景数据，同时在运行单个测试用例的时候加载特别的测试数据  
[@AfterClass](https://testerhome.com/AfterClass)[@After](https://testerhome.com/After)中做测试数据清理等相关操作  
--在执行完相关测试以后清理用例现场  
（2）断言  
--不要做无谓的断言  
在测试模式下, 有时会情不自禁的滥用断言. 这种做法会导致维护更困难, 需要极力避免. 仅对测试方法名指示的特性进行明确测试,因为对于一般性代码而言, 保证测试代码尽可能少是一个重要目标  
--使用显式断言方式  
应该总是优先使用 assertEquals\(a, b\) 而不是 assertTrue\(a == b\), 因为前者会给出更有意义的测试失败信息. 在事先不确定输入值的情况下, 这条规则尤为重要  
--断言的参数顺序要合适  
（3）测试用例保持独立  
--确保测试代码独立于项目代码之外  
--为了保证测试稳定可靠且便于维护, 测试用例之间决不能有相互依赖, 也不能依赖执行的先后次序.  
（4）测试代码要考虑错误处理  
--如果前面的代码执行失败, 后续语句会导致代码崩溃, 剩下的测试都无法执行. 任何时候都要为测试失败做好准备, 避免单个失败的测试项中断整个测试套件的执行  
--不要写自己的catch代码块，即只有test失败的情况，不应该存在catch情况

### 结语 {#结语}

接口测试是一个非常庞杂的体系，很难用一篇文章阐述清楚，在持续交付体系中扮演着非常重要的角色。  
其他实践如接口测试覆盖率统计、标准协议页面化测试平台（提供给没有编码能力的产品或测试人员使用）、maven项目骨架建立标准化测试工程、dubbo/hessian/restful/webservice等接口协议具体实现等等限于篇幅也没有做具体阐述。  
不过接口测试实在已经不是一个新鲜的话题，在论坛里已经看到很多人对接口测试各种实现方法做了介绍，本人也从中学习和参考了很多，所以这块的介绍就先这样草草结尾吧![](https://twemoji.b0.upaiyun.com/2/svg/1f602.svg "😂")，希望能提供大家一些有益的参考。

