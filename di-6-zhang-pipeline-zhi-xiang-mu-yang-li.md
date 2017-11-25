### 项目说明 {#项目说明}

本文将以一个微服务项目的具体pipeline样例进行脚本编写说明。一条完整的pipeline交付流水线通常会包括代码获取、单元测试、静态检查、打包部署、接口层测试、UI层测试、性能专项测试（可能还有安全、APP等专项）、人工验收等研发测试环节，还会包括灰度发布、正式发布等发布环节。

补充说明：  
1.此项目的部署还是使用传统虚拟机服务器的方式，暂未采用docker容器，docker容器与pipeline的结合后面会有其他专题进行说明。  
2.灰度发布、正式发布等发布环节，由于涉及到线上发布系统对接，该项目暂未包括，后续专题展开。  
3.采用jenkins官方推荐的declarative pipeline方式实现。

### 交付流水线（BlueOcean） {#交付流水线（BlueOcean）}

Jenkins UI从2006年-2016年，几乎没有变化。为了适应Jenkins Pipeline和 Freestyle jobs任务，Jenkins推出了BlueOcean UI，其目的就是让程序员执行任务时，降低工作流程的复杂度和提升工作流程的清晰度  
BlueOcean目前为止还是作为一个插件，需要Jenkins版本2.7.x以上,按照jenkins社区的规划，未来也许会逐渐取代原有的jenkins界面。  
当然目前功能上还不是太成熟，期待未来更加强大吧，不过UI界面上确实非常漂亮。

**参数化构建界面**  
[![](https://testerhome.com/uploads/photo/2017/a2dcb5ec-02ae-4ea4-978f-a236069890c2.png!large)](https://testerhome.com/uploads/photo/2017/a2dcb5ec-02ae-4ea4-978f-a236069890c2.png!large)

**交付流水线界面**  
[![](https://testerhome.com/uploads/photo/2017/bee06982-e9a4-452f-8d5d-b80645ece23e.png!large)](https://testerhome.com/uploads/photo/2017/bee06982-e9a4-452f-8d5d-b80645ece23e.png!large)

### 脚本详解 {#脚本详解}

Pipeline支持两种语法：Declarative Pipeline（在Pipeline 2.5中引入，结构化方式）和Scripted Pipeline，两者都支持建立连续输送Pipeline。  
为与BlueOcean脚本编辑器兼容，通常建议使用Declarative Pipeline的方式进行编写,从jenkins社区的动向来看，很明显这种语法结构也会是未来的趋势。  
下面的样例以Declarative Pipeline的方式进行详细讲解，关键步骤都增加了注解和说明，部分公司敏感信息做了脱敏处理。

```groovy
#!groovy
pipeline {
    //在任何可用的代理上执行Pipeline
    agent any
    //参数化变量，目前只支持[booleanParam, choice, credentials, file, text, password, run, string]这几种参数类型，其他高级参数化类型还需等待社区支持。
    parameters {
    //git代码路径【参数值对外隐藏】
    string(name:'repoUrl', defaultValue: 'git@git.*****.com:*****/*****.git', description: 'git代码路径')
    //repoBranch参数后续替换成git parameter不再依赖手工输入,JENKINS-46451【git parameters目前还不支持pipeline】
    string(name:'repoBranch', defaultValue: 'master', description: 'git分支名称')
    //pom.xml的相对路径
    string(name:'pomPath', defaultValue: 'pom.xml', description: 'pom.xml的相对路径')
    //war包的相对路径
    string(name:'warLocation', defaultValue: 'rpc/war/target/*.war', description: 'war包的相对路径 ')
    //服务器参数采用了组合方式，避免多次选择，使用docker为更佳实践【参数值对外隐藏】
    choice(name: 'server',choices:'192.168.1.107,9090,*****,*****\n192.168.1.60,9090,*****,*****', description: '测试服务器列表选择(IP,JettyPort,Name,Passwd)')
    //测试服务器的dubbo服务端口
    string(name:'dubboPort', defaultValue: '31100', description: '测试服务器的dubbo服务端口')
    //单元测试代码覆盖率要求，各项目视要求调整参数
    string(name:'lineCoverage', defaultValue: '20', description: '单元测试代码覆盖率要求(%)，小于此值pipeline将会失败！')
    //若勾选在pipelie完成后会邮件通知测试人员进行验收
    booleanParam(name: 'isCommitQA',description: '是否邮件通知测试人员进行人工验收',defaultValue: false )
    }
    //环境变量，初始确定后一般不需更改
    tools {
        maven 'maven3'
        jdk   'jdk8'
    }
    //常量参数，初始确定后一般不需更改
    environment{
        //git服务全系统只读账号cred_id【参数值对外隐藏】
        CRED_ID='*****-****-****-****-*********'
        //测试人员邮箱地址【参数值对外隐藏】
        QA_EMAIL='*****@*****.com'
        //接口测试（网络层）的job名，一般由测试人员编写
        ITEST_JOBNAME='Guahao_InterfaceTest_ExpertPatient'
    }
    options {
        //保持构建的最大个数
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }
    //定期检查开发代码更新，工作日每晚4点做daily build
    triggers {
        pollSCM('H 4 * * 1-5')
    }
   //pipeline运行结果通知给触发者
    post{
        success{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run success\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
        failure{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                body: "${BUILD_USER}'s pineline  '${JOB_NAME}' (${BUILD_NUMBER}) run failure\n请及时前往${env.BUILD_URL}进行查看"
                }
            }

        }
        unstable{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})结果",
                body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run unstable\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
    }

    //pipeline的各个阶段场景
    stages {
        stage('代码获取') {
            steps {
            //根据param.server分割获取参数,包括IP,jettyPort,username,password
            script {
                def split=params.server.split(",")
                serverIP=split[0]
                jettyPort=split[1]
                serverName=split[2]
                serverPasswd=split[3]
            }
              echo "starting fetchCode from ${params.repoUrl}......"
              // Get some code from a GitHub repository
              git credentialsId:CRED_ID, url:params.repoUrl, branch:params.repoBranch
            }
        }
        stage('单元测试') {
            steps {
              echo "starting unitTest......"
              //注入jacoco插件配置,clean test执行单元测试代码. All tests should pass.
              sh "mvn org.jacoco:jacoco-maven-plugin:prepare-agent -f ${params.pomPath} clean test -Dautoconfig.skip=true -Dmaven.test.skip=false -Dmaven.test.failure.ignore=true"
              junit '**/target/surefire-reports/*.xml'
              //配置单元测试覆盖率要求，未达到要求pipeline将会fail,code coverage.LineCoverage>20%.
              jacoco changeBuildStatus: true, maximumLineCoverage:"${params.lineCoverage}"
            }
        }
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

        stage('部署测试环境') { 
            steps {
                echo "starting deploy to ${serverIP}......"
                //编译和打包
                sh "mvn  -f ${params.pomPath} clean package -Dautoconfig.skip=true -Dmaven.test.skip=true"
                archiveArtifacts warLocation
                script {
                    wrap([$class: 'BuildUser']) {
                    //发布war包到指定服务器，虚拟机文件目录通过shell脚本初始化建立，所以目录是固定的
                    sh "sshpass -p ${serverPasswd} scp ${params.warLocation} ${serverName}@${serverIP}:htdocs/war"
                    //这里增加了一个小功能，在服务器上记录了基本部署信息，方便多人使用一套环境时问题排查，storge in {WORKSPACE}/deploy.log  & remoteServer:htdocs/war
                    Date date = new Date()
                    def deploylog="${date.toString()},${BUILD_USER} use pipeline  '${JOB_NAME}(${BUILD_NUMBER})' deploy branch ${params.repoBranch} to server ${serverIP}"
                    println deploylog
                    sh "echo ${deploylog} >>${WORKSPACE}/deploy.log"
                    sh "sshpass -p ${serverPasswd} scp ${WORKSPACE}/deploy.log ${serverName}@${serverIP}:htdocs/war"
                    //jetty restart，重启jetty
                    sh "sshpass -p ${serverPasswd} ssh ${serverName}@${serverIP} 'bin/jettyrestart.sh' "
                    }
                }
            }
        }

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

        stage('UI自动化测试') { 
             steps{
             echo "starting UITest......"
             //这个项目不需要UI层测试，UI自动化与接口测试的pipeline脚本类似
             }
         }

        stage('性能自动化测试 ') { 
            steps{
                 echo "starting performanceTest......"
                //视项目需要增加性能的冒烟测试，具体实现后续专文阐述
                }
        }

        stage('通知人工验收'){
            steps{
                script{
                    wrap([$class: 'BuildUser']) {
                    if(params.isCommitQA==false){
                        echo "不需要通知测试人员人工验收"
                    }else{
                        //邮件通知测试人员人工验收
                         mail to: "${QA_EMAIL}",
                         subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})人工验收通知",
                         body: "${BUILD_USER}提交的PineLine '${JOB_NAME}' (${BUILD_NUMBER})进入人工验收环节\n请及时前往${env.BUILD_URL}进行测试验收"
                    }

                    }
                }
            }
        }

        // stage('发布系统') { 
        //     steps{
        //         echo "starting deploy......"
        //     //    TODO发布环节后续专题阐述
        //     }
        // }
    }
```



 当然，这个pipeline脚本只是针对该项目的场景展开并尽可能做到通用（在本公司大多数的项目还是适用的，只需要对参数化参数进行调整），部分项目会根据自己项目的情况进行裁剪和微调，比如一些耗时长的stage会使用parallel并发的方式进行等等。[![](https://testerhome.com/uploads/photo/2017/d9c58118-dd2b-4243-a816-71708a2f1948.png!large)  
](https://testerhome.com/uploads/photo/2017/d9c58118-dd2b-4243-a816-71708a2f1948.png!large)

```groovy

```



