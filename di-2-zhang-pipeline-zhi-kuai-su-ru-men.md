# 第2章 Pipeline之快速入门

### 什么是pipeline {#什么是pipeline}

先介绍下什么是Jenkins 2.0，Jenkins 2.0的精髓是Pipeline as Code，是帮助Jenkins实现CI到CD转变的重要角色。什么是Pipeline，简单来说，就是一套运行于Jenkins上的工作流框架，将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂发布流程。Pipeline的实现方式是一套Groovy DSL，任何发布流程都可以表述为一段Groovy脚本，并且Jenkins支持从代码库直接读取脚本，从而实现了Pipeline as Code的理念。  
Pipeline的几个基本概念：  
Stage: 阶段，一个Pipeline可以划分为若干个Stage，每个Stage代表一组操作。注意，Stage是一个逻辑分组的概念，可以跨多个Node。  
Node: 节点，一个Node就是一个Jenkins节点，或者是Master，或者是Agent，是执行Step的具体运行期环境。  
Step: 步骤，Step是最基本的操作单元，小到创建一个目录，大到构建一个Docker镜像，由各类Jenkins Plugin提供。  
本节介绍Jenkins Pipeline的一些核心概念，并介绍在运行的Jenkins实例中定义和使用Pipelines的基础知识。

### 使用条件 {#使用条件}

要使用Jenkins Pipeline，需要：  
Jenkins 2.x或更高版本  
Pipeline插件

### Pipeline定义 {#Pipeline定义}

Pipeline脚本是用Groovy写的 。Groovy语法将在后续文档中介绍。  
可以通过以下任一方式创建基本Pipeline：  
pipeline script：直接在Web UI的script输入框里面输入pipeline script语句即可，参考说明可以点击输入框下边的Pipeline Syntax，里面有很多示例操作说明，非常好用。  
pipeline script from SCM：需要配置SCM代码存储Git地址或SVN地址，指定script文件Jenkinsfile所在路径，每次构建job会自动去指定的目录执行script文件  
以上两种方法定义Pipeline的语法都是一样的。

**在Web UI中定义Pipeline**  
要在Jenkins Web UI中创建基本Pipeline Job，请按照下列步骤操作：  
单击Jenkins主页上的New Item。  
[![](https://testerhome.com/uploads/photo/2017/a0b3ec34-82c3-45e8-81a5-c32cb3dd02ed.png!large)](https://testerhome.com/uploads/photo/2017/a0b3ec34-82c3-45e8-81a5-c32cb3dd02ed.png!large)  
输入Pipeline的名称，选择Pipeline，然后单击确定。  
[![](https://testerhome.com/uploads/photo/2017/7d52e087-4cfb-4f53-9ca2-ccc66e73ed83.png!large)](https://testerhome.com/uploads/photo/2017/7d52e087-4cfb-4f53-9ca2-ccc66e73ed83.png!large)  
在脚本文本区域中，输入Pipeline，然后单击保存。  
[![](https://testerhome.com/uploads/photo/2017/18e45ee8-be88-4fec-b428-26b3f16faadd.png!large)](https://testerhome.com/uploads/photo/2017/18e45ee8-be88-4fec-b428-26b3f16faadd.png!large)  
[![](https://testerhome.com/uploads/photo/2017/32890478-cd90-42b1-98a6-48740c6e120b.png!large)](https://testerhome.com/uploads/photo/2017/32890478-cd90-42b1-98a6-48740c6e120b.png!large)  
[![](https://testerhome.com/uploads/photo/2017/2e47139e-063d-4e5b-8121-d97da555a864.png!large)](https://testerhome.com/uploads/photo/2017/2e47139e-063d-4e5b-8121-d97da555a864.png!large)  
单击“构建历史记录”下的＃buildId，然后单击控制台输出以查看Pipeline的完整输出  
[![](https://testerhome.com/uploads/photo/2017/41e722cd-c0f8-4652-a695-9c8b853755ab.png!large)](https://testerhome.com/uploads/photo/2017/41e722cd-c0f8-4652-a695-9c8b853755ab.png!large)  
[![](https://testerhome.com/uploads/photo/2017/c910af92-b975-475f-9188-a84f8fc393e5.png!large)](https://testerhome.com/uploads/photo/2017/c910af92-b975-475f-9188-a84f8fc393e5.png!large)

**在SCM中定义pipeline**  
复杂的Pipeline难以在Pipeline配置页面的文本区域内进行写入和维护。为了解决这一问题，jenkins Pipeline支持在文本编辑器中编写脚本文件jenkinsFile，Jenkins可以通过从SCM选项的控件中加载Pipeline脚本。  
选择SCM选项中的Pipeline脚本后，不要在Jenkins UI中输入任何Groovy代码; 只需指定要检索的Pipeline脚本的路径。更新指定的存储库时，只要Pipeline配置了SCM轮询触发器，就会触发一个新构建。  
---文本编辑器，IDE，GitHub等将使用Groovy代码进行语法高亮显示， 第一行Jenkinsfile应该是\#!/usr/bin/env groovy Jenkinsfile。  
**内置文档**  
Pipeline配有内置的文档功能，可以更轻松地创建不同复杂性的Pipeline。根据Jenkins实例中安装的插件自动生成和更新内置文档。  
内置文档可以在全局范围内找到： localhost:8080/pipeline-syntax/，假设您有一个Jenkins实例在本地端口8080上运行。同样的文档也作为pipeline语法链接到任何配置的Pipeline的侧栏中项目。  
[![](https://testerhome.com/uploads/photo/2017/c26461d8-61e3-4368-90ab-ae160a27233b.png!large)](https://testerhome.com/uploads/photo/2017/c26461d8-61e3-4368-90ab-ae160a27233b.png!large)

**代码段生成器**  
内置的“Snippet Generator”程序有助于为单个步骤生成代码段。  
Snippet Generator动态填充Jenkins实例可用的步骤列表。可用的步骤数量取决于安装的插件，它明确地暴露了在Pipeline中使用的步骤。  
要使用代码段生成器生成步骤代码片段：  
1、从配置的流水线或本地主机：8080 / pipeline-syntax导航到Pipeline语法链接Pipeline Syntax  
[![](https://testerhome.com/uploads/photo/2017/54fdf0d9-d4b1-4e01-be0c-89bc5a0fee32.png!large)](https://testerhome.com/uploads/photo/2017/54fdf0d9-d4b1-4e01-be0c-89bc5a0fee32.png!large)  
2、在“ Sample Step”下拉菜单中选择所需的步骤,使用“Sample Step”下拉列表下方的动态填充区域配置所选步骤,如message为“hello world”，单击生成Pipeline脚本以创建一个可以复制并粘贴到Pipeline中的Pipeline代码段  
[![](https://testerhome.com/uploads/photo/2017/613ab7ea-c732-468e-9844-799adf134e03.png!large)](https://testerhome.com/uploads/photo/2017/613ab7ea-c732-468e-9844-799adf134e03.png!large)  
**全局变量引用**  
除了代码片段生成器之外，Pipeline还提供了一个内置的“ 全局变量引用”。像Snippet Generator一样，它也是由插件动态填充的。与代码段生成器不同的是，全局变量引用仅包含Pipeline提供的变量，这些变量可用于Pipeline。  
[![](https://testerhome.com/uploads/photo/2017/689fbe6a-8212-4c3b-a9d5-9d2a6262cbc7.png!large)](https://testerhome.com/uploads/photo/2017/689fbe6a-8212-4c3b-a9d5-9d2a6262cbc7.png!large)  
在Pipeline中默认提供的变量是：  
ENV  
Pipeline脚本可访问的环境变量，例如： env.PATH或env.BUILD\_ID。请参阅内置的全局变量，以获取管道中可用的完整和最新的环境变量列表。  
PARAMS  
将为Pipeline定义的所有参数公开，例如：params.MY\_PARAM\_NAME。  
currentBuild  
可获取当前正在执行的Pipeline job的信息，例如属性currentBuild.result，currentBuild.displayName等等

引用官方文档：[https://jenkins.io/doc/book/pipeline/getting-started/](https://jenkins.io/doc/book/pipeline/getting-started/)  
Groovy语法及入门：  
[Groovy入门之语法和变量定义](http://www.sunnyang.com/521.html)  
[Groovy进阶之函数、闭包和类](http://www.sunnyang.com/522.html)  
[精通 Groovy](http://www.ibm.com/developerworks/cn/education/java/j-groovy/j-groovy.html#ibm-pcon)

