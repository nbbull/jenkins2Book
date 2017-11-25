### Jacoco简介 {#Jacoco简介}

Jacoco是一个开源的覆盖率工具。Jacoco可以嵌入到Ant 、Maven中，并提供了EclEmma Eclipse插件,也可以使用JavaAgent技术监控Java程序。很多第三方的工具提供了对Jacoco的集成，如sonar、Jenkins等。  
官网地址：[http://www.eclemma.org/jacoco/](http://www.eclemma.org/jacoco/)

### 支持的集成工具 {#支持的集成工具}

Jacoco团队提供了如下的一些集成工具的支持：  
Java API  
[http://www.eclemma.org/jacoco/trunk/doc/api/index.html](http://www.eclemma.org/jacoco/trunk/doc/api/index.html)  
Command Line  
[http://www.eclemma.org/jacoco/trunk/doc/agent.html](http://www.eclemma.org/jacoco/trunk/doc/agent.html)  
Apache Ant  
[http://www.eclemma.org/jacoco/trunk/doc/ant.html](http://www.eclemma.org/jacoco/trunk/doc/ant.html)  
Apache Maven  
[http://www.eclemma.org/jacoco/trunk/doc/maven.html](http://www.eclemma.org/jacoco/trunk/doc/maven.html)  
Eclipse EclDmma Plugin  
[http://www.eclemma.org/](http://www.eclemma.org/)

### Jacoco与Eclipse集成 {#Jacoco与Eclipse集成}

打开 Eclipse 的软件市场，在其中搜索 EclEmma，找到后完成安装，如下图所示：  
[![](https://testerhome.com/uploads/photo/2017/6d769276-29cb-478f-96a2-733af1ac5f11.png!large)](https://testerhome.com/uploads/photo/2017/6d769276-29cb-478f-96a2-733af1ac5f11.png!large)  
安装完成后，Eclipse 的工具条里会多出下面这样一个图标：  
[![](https://testerhome.com/uploads/photo/2017/3961e750-f8ef-4685-a76f-287a40016654.png!large)](https://testerhome.com/uploads/photo/2017/3961e750-f8ef-4685-a76f-287a40016654.png!large)

### Jacoco与jenkins集成 {#Jacoco与jenkins集成}

**安装jacoco插件**  
[![](https://testerhome.com/uploads/photo/2017/1634a98a-8f68-43e3-977c-3f08677853f2.png!large)](https://testerhome.com/uploads/photo/2017/1634a98a-8f68-43e3-977c-3f08677853f2.png!large)  
**Jenkins中构建参数**  
关键maven参数：

```
mvn org.jacoco:jacoco-maven-plugin:prepare-agent  clean  package  -Dautoconfig.skip=true   -Dmaven.test.skip=false  -Dmaven.test.failure.ignore=true
```

org.jacoco:jacoco-maven-plugin:prepare-agent：命令行引用jacoco-maven-plugin插件，减少对开发源码的依赖。  
-Dmaven.test.skip=false:启用代码中的单元测试，开发代码中一般默认是关闭的。  
-Dmaven.test.failure.ignore=true：忽略失败的单元测试用例继续执行。  
**配置jacoco插件**  
在“Addpost-build action”中选择“Reccord Jacoco coverage report”  
配置文件路径：  
[![](https://testerhome.com/uploads/photo/2017/dc58f58e-1221-4cbc-a754-7adf1ea57043.png!large)](https://testerhome.com/uploads/photo/2017/dc58f58e-1221-4cbc-a754-7adf1ea57043.png!large)  
Path to exec files ：代码覆盖率统计文件位置；  
Path to class directorie：classes文件位置；  
Path to source directories：源码文件位置；  
根据需要填写覆盖率要求；  
[![](https://testerhome.com/uploads/photo/2017/b45f91f1-063b-4976-b52e-3d886eadf97c.png!large)](https://testerhome.com/uploads/photo/2017/b45f91f1-063b-4976-b52e-3d886eadf97c.png!large)  
**Jacoco覆盖率报告**  
[![](https://testerhome.com/uploads/photo/2017/de55eac4-c465-42c0-9555-90ac582b1a2e.png!large)](https://testerhome.com/uploads/photo/2017/de55eac4-c465-42c0-9555-90ac582b1a2e.png!large)

### Jacoco与Jenkins Pipeline集成 {#Jacoco与Jenkins Pipeline集成}

可视项目和团队情况，增加对测试覆盖率的要求，比如下面例子就是当代码覆盖率低于70%时，这个阶段将会fail掉。

```groovy
stage('单元测试') {
steps {
echo "starting unitTest......"
//clean test. All tests should pass.
sh "mvn org.jacoco:jacoco-maven-plugin:prepare-agent -f pom.xml clean test -Dautoconfig.skip=true -Dmaven.test.skip=false -Dmaven.test.failure.ignore=true"
junit '**/target/surefire-reports/*.xml'
//code coverage.LineCoverage>70%.
jacoco changeBuildStatus: true, maximumLineCoverage:70
}
}
```

### Jacoco与SonarQube集成 {#Jacoco与SonarQube集成}

**jacoco report报告路径配置**  
[![](https://testerhome.com/uploads/photo/2017/08453dab-d264-4d35-869d-51aa6deec884.png!large)](https://testerhome.com/uploads/photo/2017/08453dab-d264-4d35-869d-51aa6deec884.png!large)  
代码覆盖率统计数据  
[![](https://testerhome.com/uploads/photo/2017/e88a5839-9fd4-43ff-9999-5086090edce9.png!large)](https://testerhome.com/uploads/photo/2017/e88a5839-9fd4-43ff-9999-5086090edce9.png!large)

### 一些问题和解决方案 {#一些问题和解决方案}

坑1：使用了反射的单元测试用例执行报错：java.lang.NoSuchMethodException: com.greenline.expertpatient.model.po.EventRecordPO.set$jacocoData\(\[Z\)  
**解决办法：**  
To collect execution data JaCoCo instruments the classes under test which adds two members to the classes: A private static field $jacocoData and a private static method $jacocoInit\(\). Both members are marked as synthetic.  
Please change your code to ignore synthetic members. This is a good practice anyways as also the Java compiler creates synthetic members in certain situation.  
修改使用反射的测试用例，加个判断  
if\(!fields\[i\].isSynthetic\(\)\){  
//Or whatever processing you are doing here with your fields.  
}  
参考链接：[http://www.eclemma.org/jacoco/trunk/doc/faq.html](http://www.eclemma.org/jacoco/trunk/doc/faq.html)

坑2：multi-module maven项目，sonarQube只会检查指定parent module目录里的jacoco.exec覆盖率统计文件，而不会检查其他子module目录，即使在sonar里把sonar.jacoco.reportPaths设置成\*\*/\*\*.exec也不行（万马奔腾而过...）。  
**解决办法1：**  
jenkins jacoco plugin里没这个问题，代码覆盖率数据和报告可在jenkins上直接查看，sonarqube的问题等待社区后续完善。  
**解决办法2：**  
设置jacoco的destFile属性，合并所有的jacoco.exec报告到multiModuleProjectDirectory目录

```
<jacoco.destFile>${maven.multiModuleProjectDirectory}/target/jacoco.exec</jacoco.destFile>
```

${maven.multiModuleProjectDirectory}参数需要maven 3.3.1 以上版本支持。  
参考链接：[https://stackoverflow.com/questions/13031219/how-to-configure-multi-module-maven-sonar-jacoco-to-give-merged-coverage-rep](https://stackoverflow.com/questions/13031219/how-to-configure-multi-module-maven-sonar-jacoco-to-give-merged-coverage-rep)

