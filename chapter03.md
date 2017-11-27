# 第3章 Pipeline使用之语法详解

## 3.1 概述

在本章中，我们重点介绍Pipeline的语法，从Pipeline插件2.5版开始，Pipeline支持两种独立的语法结构：Declarative Pipeline和Scripted Pipeline，两者都支持建立连续传送的Pipeline。

如“入门指南”所述，Pipeline最基本的部分是“step”，step告诉Jenkins要做什么，并且作为Declarative Pipeline和Scripted Pipeline语法的基本构建块。

为与BlueOcean编辑器兼容，通常建议使用Declarative Pipeline的方式进行编写,这种语法结构也会是Jenkins Pipeline未来发展的趋势。

## 3.2 Declarative Pipeline

从Pipeline2.5版本以后，Jenkins Pipeline新增了一种新的语法类型Declarative Pipeline（声明式Pipeline），它在Pipeline系统之上提供了一种更加简单和有意义的语法。

所有有效的Declarative Pipeline必须包含在一个pipeline块内，例如：

```groovy
pipeline { 
/* insert Declarative Pipeline here */ 
    }
```

Declarative Pipeline遵循与Groovy相同的语法规则，但有以下几点例外：

* Pipeline的顶层必须是块，具体来说就是：pipeline { }。

* 不用分号作为语句分隔符，每个声明必须独立一行。

* 块里只能包含Sections（章节）、Directives（指令）、 Steps（步骤）或赋值语句。

* 属性引用以无参方法的方式调用。例如，输入被视为input（）。

### 3.2.1 Sections（章节）

Declarative Pipeline里的Sections通常包含一个或多个Directives或Steps。

#### agent

agent指定整个Pipeline或特定stage在Jenkins环境中执行的位置。在pipeline代码块的顶层agent必须进行定义，但在stage级使用是可选的。

| **需要** | 是 |
| :--- | :--- |
| **参数** | 见参数说明 |
| **允许** | 在pipeline顶层代码块或每个stage级代码块中 |

##### 参数列表

为实现Pipeline可能拥有的各种用例，agent支持几种不同类型的参数。这些参数可以应用于pipeline块的顶层，也可以应用在每个stage指令内。

**any**

在任何可用的agent 上执行Pipeline或stage。例如：agent any  
**none**

当在pipeline块的顶层使用none时，将不会为整个Pipeline运行分配全局agent ，每个stage部分将需要定义其自己的agent。

**label**

提供label标签名称，在Jenkins环境中可用的agent上执行Pipeline或stage。

例如：agent { label 'my-defined-label' }

**node**

agent { node { label 'labelName' } }，等同于 agent { label 'labelName' }，但node也允许其他选项（如customWorkspace）。

**docker**

定义此参数时，执行Pipeline或stage时会动态提供一个docker节点去运行基于Docker的Pipelines。docker还可以接受一个args参数，直接传递给docker run指令调用。

例如：agent { docker 'maven:3-alpine' }或

```groovy
agent {
    docker {
        image 'maven:3-alpine'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
```

**dockerfile**

使用从Dockerfile仓库中包含的dockerfile创建镜像文件来构建执行Pipeline或stage。为了使用此选项，Jenkinsfile必须从Multibranch Pipeline或“Pipeline from SCM"中加载。

默认目录是在Dockerfile仓库的根目录：agent { dockerfile true }。如果Dockerfile需在另一个目录中建立，可使用dir选项：agent { dockerfile { dir 'someSubDir' } }。

还可以通过docker build ...使用additionalBuildArgs选项，如agent { dockerfile { additionalBuildArgs '--build-arg foo=bar' } }。

##### 通用选项

这些是可以应用于两个或多个agent中的选项。除非明确定义，否则非必需。

**label**

string字符串。标记在哪里运行pipeline或stage

此选项适用于node，docker和dockerfile，并且在node中是必需的。  
**customWorkspace**

string字符串。自定义运行的工作空间,它可以是相对路径，在这种情况下，自定义工作区将位于node节点工作空间的根目录下，也可以是绝对路径。例如：

```groovy
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```

**reuseNode**  
一个布尔值，默认为false。如果为true，则在同一工作空间中，此选项适用于docker和dockerfile，并且仅在独立stage中使用agent时才有效。

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent { docker 'maven:3-alpine' } ①
    stages {
        stage('Example Build') {
            steps {
                sh 'mvn -B clean verify'
            }
        }
    }
}
```

**①**使用‘maven:3-alpine’的镜像创建容器，执行pipeline的所有步骤。

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent none ①
    stages {
        stage('Example Build') {
            agent { docker 'maven:3-alpine' } ②
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('Example Test') {
            agent { docker 'openjdk:8-jre' } ③
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

**①**agent none在Pipeline顶层定义，表示将不会为整个Pipeline运行分配全局agent，每个stage需自己设置agent。

**②**使用‘maven:3-alpine’的镜像创建容器，执行此阶段中的步骤。

**③**使用‘openjdk:8-jre’的镜像创建容器，执行此阶段中的步骤。

#### post

定义Pipeline或stage运行结束后的操作。post支持以下类型的代码块：always，changed，failure，success，unstable和aborted。这些代码块允许在Pipeline或stage运行结束时执行相关步骤，具体取决于Pipeline的运行状态。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 在pipeline顶层代码块或每个stage级代码块中 |

##### 参数列表

**always**

结束时运行，无论Pipeline运行的完成状态如何。

**changed**

只有当前Pipeline运行的状态与先前完成的Pipeline的状态不同时，才能运行。

**failure**

只有当前Pipeline处于“失败”状态时才运行，通常用红色指示的Web UI表示。

**success**

只有当前Pipeline具有“成功”状态时才运行，通常用蓝色或绿色指示的Web UI表示。

**unstable**

只有当前Pipeline具有“不稳定”状态，一般由测试失败，代码违例等引起，才能运行。通常用黄色指示的Web UI表示。

**aborted**

只有当前Pipeline处于“中止”状态时，才会运行，通常是由于Pipeline被手动中止。通常用灰色指示的Web UI表示。

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    post { ①
        always { ②
            echo 'I will always say Hello again!'
        }
    }
}
```

①post章节通常会放在pipeline末端。

②post代码块里包括steps章节的内容。

#### stages

包含一个或多个stage的序列，Pipeline的大部分工作在此执行。建议stages至少包含至少一个stage指令，用于连接各个交付过程，如构建，测试和部署等。

| **需要** | 是 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages { ①
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**①**stages章节通常跟随在agent,options等指令后面。

#### steps

steps包含一个或多个在stage块中执行的step序列。

| **需要** | 是 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 在每个stage代码块内。 |

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages { 
        stage('Example') {
            steps {①
                echo 'Hello World'
            }
        }
    }
}
```

**① steps章节必须包括一个或多个step。**

### 3.2.2 Directives （指令）

#### environment

environment指令指定一系列键值对，这些键值对将被定义为所有step或stage中step的环境变量，具体取决于environment指令在Pipeline中的位置。

该指令支持一种特殊的方法credentials\(\)，可通过标识符访问Jenkins环境中预定义好的Credential凭证。

对于“Secret Text”类型的凭据，credentials\(\)方法需确保指定的环境变量包含Secret Text内容，对于“Standard username and password"”类型的凭证，指定的环境变量需要设置为username:password。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 在pipeline块内或stage指令内。 |

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
environment
pipeline {
    agent any
    environment { ①
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { ②
                AN_ACCESS_KEY = credentials('my-prefined-secret-text') ③
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

**①**environment指令放在pipeline顶级块中，将适用pipeline所有步骤。

**②**environment指令放在stage中，给定的环境变量将只适用该stage中的步骤。

**③**environment块中使用credentials\(\)方法，可以访问Jenkins环境中预定义的凭证。

#### options

options指令允许在Pipeline内配置Pipeline专用选项。Pipeline本身提供了许多选项，例如buildDiscarder，它们也可以由Jenkins插件提供，例如 timestamps。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 参数列表

**buildDiscarder**

pipeline保持构建的最大个数。例如：

options{buildDiscarder\(logRotator\(numToKeepStr: '1'\)\)}

**disableConcurrentBuilds**

不允许并行执行Pipeline,可用于防止同时访问共享资源等。例如：

options {disableConcurrentBuilds\(\)}

**skipDefaultCheckout**

默认跳过来自源代码控制的代码。例如：

options {skipDefaultCheckout\(\)}

**skipStagesAfterUnstable**

一旦构建状态进入了“Unstable”状态，就跳过此stage。例如：

options {skipStagesAfterUnstable\(\)}

**timeout**  
设置Pipeline运行的超时时间。例如：

options {timeout\(time: 1, unit: 'HOURS'\)}F

**retry**

失败后，重试整个Pipeline的次数。例如：

options {retry\(3\)}

**timestamps**

预定义由Pipeline生成的所有控制台输出时间。例如：

options {timestamps\(\)}

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    options { 
        timeout(time: 1, unit: 'HOURS') ①
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**①**设置pipeline全局的超时时间为1小时，超时后将会自动终止pipeline运行。

#### parameters

parameters指令提供用户在触发Pipeline时的参数列表。这些参数值通过params对象可用于Pipeline步骤，具体用法如下

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 参数列表

**string**

string类型的参数, 例如:

```groovy
parameters { 
string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '')
 }
```

**booleanParam**

boolean类型的参数, 例如:

```groovy
parameters {
 booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') 
}
```

截至发稿，Jenkins社区目前已支持\[booleanParam, choice, credentials, file, text, password, run, string\]这几种参数类型，其他高级参数化类型也在陆续完善中。

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
            }
        }
    }
}
```

#### triggers

triggers指令定义了Pipeline自动化触发的方式。对于与源代码集成的Pipeline，如GitHub或BitBucket，triggers可能不需要基于webhook的集成也已经存在。目前只有两个可用的触发器：cron、pollSCM和upstream。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 只能有一次，在pipeline代码块内。 |

**cron**

接受一个cron风格的字符串来定义Pipeline触发的常规间隔，例如：

triggers {cron\('H 4/\* 0 0 1-5'\)}

**pollSCM**  
接受一个cron风格的字符串来定义Jenkins检查SCM源更改的常规间隔。如果存在新的更改，则Pipeline将被重新触发。例如：triggers {pollSCM\('H 4/\* 0 0 1-5'\)}

**upstream**

可接受多个job名称以及一个threshold设置参数。任何一个job以符合threshold条件完成后，均可以触发Pipeline的运行。举例：{ upstream\(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS\) }

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    triggers {
        cron('H 4/* 0 0 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### stage

stage指令包含在stages中，包含step、agent（可选）或其他特定包含于stage中的指令。实际上，Pipeline完成的所有实际工作都包含在一个或多个stage指令中。

| **需要** | 至少一个 |
| :--- | :--- |
| **参数** | 一个强制参数，一个标识stage名称的字符串。 |
| **允许** | 在stages章节内。 |

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### tools

通过tools可自动安装工具，并放置环境变量到PATH。如果agent none，这将被忽略。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 在pipeline块内或stage指令内。 |

**支持的Tools**

maven

jdk

gradle

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1' ①
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

**①**调用的tool必须被预置在Jenkins中，可通过**Manage Jenkins**→**Global Tool Configuration配置。**

#### when

when指令允许Pipeline根据给定的条件确定是否执行该阶段。when指令必须至少包含一个条件，如果when指令包含多个条件，则只有所有子条件返回true时才会执行stage，这与子条件嵌套在allOf相同（见下面的例子）。

更复杂的条件结构可使用嵌套条件：not，allOf或anyOf，嵌套条件可以嵌套到任意深度。

| **需要** | 否 |
| :--- | :--- |
| **参数** | 无 |
| **允许** | 在stage指令内。 |

##### 内置条件

**branch**

当正在构建的分支与给出的分支模式匹配时执行，例如：when { branch 'master' }。请注意，这仅适用于multibranch Pipeline。

**environment**

当指定的环境变量设置为指定值时执行，例如： when { environment name: 'DEPLOY\_TO', value: 'production' }

**expression**

当指定的Groovy表达式求值为true时执行，例如： when { expression { return params.DEBUG\_BUILD } }

**not**

当嵌套条件为false时执行。必须包含一个条件。例如：when { not { branch 'master' } }

**allOf**

当所有嵌套条件都为true时执行。必须至少包含一个条件。例如：when { allOf { branch 'master'; environment name: 'DEPLOY\_TO', value: 'production' } }

**anyOf**

当至少一个嵌套条件为真时执行。必须至少包含一个条件。例如：when { anyOf { branch 'master'; branch 'staging' } }

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                expression { BRANCH_NAME ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

### 3.2.3 Parallel\(并行\)

Declarative Pipeline的stages中可能包含多个嵌套的stage, 对相互不存在依赖的stage可以通过并行的方式执行，以提升pipeline的运行效率。

另外，通过在某个stage中设置“failFast true”，可实现当这个stage运行失败的时候，强迫所有parallel stages中止运行（详见下面的例子）。

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
            }
        }
    }
}
```

### 3.2.4 Steps（步骤）

Declarative Pipeline可使用Pipeline Steps手册中的所有可用步骤，以及以下仅在Declarative Pipeline中支持的步骤。

Pipeline Stepsreference：[https://jenkins.io/doc/pipeline/steps/](https://jenkins.io/doc/pipeline/steps/)

#### script

script步骤中可以引用script Pipeline语句，并在Declarative Pipeline中执行。对于大多数用例，script在Declarative Pipeline中的步骤不是必须的，但它可以提供一个有用的加强。

##### 样例

```groovy
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
                script {
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```

## 3.3 Scripted Pipeline

Groovy脚本不一定适合所有使用者，因此Jenkins创建了Declarative Pipeline，为编写Jenkins Pipeline提供了一种更简单、更有意义的语法。但是不可否认，由于脚本化的pipeline是基于groovy的一种DSL语言，所以与Declarative pipeline相比为Jenkins用户提供了更巨大的灵活性和可扩展性。

### 3.3.1 流程控制

Pipeline脚本同其它脚本语言一样，从上至下顺序执行，它的流程控制取决于Groovy表达式，如if/else条件语句，举例如下：

```groovy
//Jenkinsfile (Scripted Pipeline)
node {
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
```

Pipeline脚本流程控制的另一种方式是Groovy的异常处理机制。当任何一个步骤因各种原因而出现异常时，都必须在Groovy中使用try/catch/finally语句块进行处理，举例如下：

```groovy
//Jenkinsfile (Scripted Pipeline)
node {
    stage('Example') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw
        }
    }
}
```

### 3.3.2 Steps

如本章开始所言，pipeline最核心和基本的部分就是“step”。从根本上来说，steps是作为Declarative pipeline和Scripted pipeline语法的最基本的语句块，来告诉jenkins应该执行什么操作。

Scripted pipeline不再专门将steps作为它的语法的一部分来介绍，但是在Pipeline Steps reference这篇文档中对pipeline及其插件涉及的steps做了很详细的介绍。如有需要可参考Jenkins官网对该部分的介绍。

### 3.3.3 与普通Groovy的区别

由于pipeline的一些个性化需求，比如在重新启动Jenkins后要求pipeline脚本仍然可以运行，那么pipeline脚本必须将相关数据做序列化，然而这一点 Groovy并不能完美的支持。

### 3.3.4 Declarative Pipeline和Scripted Pipeline的比较

共同点：

两者都是pipeline代码的持久实现，都能够使用pipeline内置的插件或者插件提供的steps，两者都可以利用共享库扩展。

区别：

两者不同之处在于语法和灵活性。Declarative Pipeline对用户来说，语法更严格，有固定的组织结构，更容易生成代码段，使其成为用户更理想的选择。但是Scripted pipeline更加灵活，因为Groovy本身只能对结构和语法进行限制，对于更复杂的pipeline来说，用户可以根据自己的业务进行灵活的实现和扩展。

## 3.4 小结

Pipeline语法是使用Jenkins Pipeline的基础，Jenkins提供了Declarative Pipeline和Scripted Pipeline两种语法结构，这两者在底层都是基于相同的Pipeline子系统，依照"Pipeline as code"的理念进行实现。

相对来说，Declarative Pipeline语法更简洁也更容易理解，而且可以与BlueOcean编辑器进行图形化操作结合，也是Jenkins社区鼓励使用的一种语法结构。

基于以上原因，后面我们的交付流水线样例也都会采用此种语法结构进行编写。

