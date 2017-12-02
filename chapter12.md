# 一、前言

随着用户数量不断增加、日常发布越来越频繁，现有的人工性能测试显然无法满足需求。而且一旦出现性能bug，通常影响非常严重。所以需要开发一套性能自动化测试平台，对核心功能进行自动化性能覆盖，如果发现性能下降就向相关人员告警，从而降低性能风险。

#### 流程图：

![](/assets/import.png)

# 二、依赖环境 {#id-新建性能自动化任务-1.依赖环境}

#### 压测机:

jmeter 插件SSH Protocol Support,具体见：[https://jmeter-plugins.org/](https://jmeter-plugins.org/)

#### 应用服务器：

应用服务；nmon数据采集

# 三、jmeter脚本 {#id-新建性能自动化任务-2.新建jmeter脚本}

#### 脚本结构

![](/assets/import1201.png)

#### 脚本分析

###### 用户自定义变量

主要包括应用服务器IP、用户名、密码等公用信息。

###### setUp Thread Group

ssh命令，远程启动nmon命令。具体格式如下：

nohup /root/nmon/nmon\_x86\_64\_rhel4 -s3 -c9 -F ${\_\_P\(testID,9999\)}.nmon -m /root/nmon/result/ &

###### 测试脚本

* 每个页面或功能独立一个线程组。
* 线程并发数量和循环次数，由外部参数传入并设置默认值。
* 测试计划，必须勾选“独立运行每个线程组”，确保每个功能的测试是独立的。

###### tearDown Thread Group

ssh命令，远程停止nmon命令。具体格式如下：

ps -ef \| grep nmon \| grep -v grep \| awk '{print $2}' \| xargs kill -9

###### 脚本存放

git地址

# 四、jenkins任务 {#id-新建性能自动化任务-3.新建jenkins任务}

#### 定义参数

* 并发数量
* 循环次数
* testID，本次测试记录ID，取当前时间，毫秒级（
  获得自1970年1月1日00:00:00 GMT开始到现在所表示的毫秒数
  ）。

#### git获取jmeter脚本

#### 执行测试

```shell

rm -f ${WORKSPACE}/150*
rm -rf ${WORKSPACE}/aggregate_report.jtl
rm -rf ${WORKSPACE}/report/
JMETER_ROOT_PATH=/root/.bzt/jmeter-taurus/3.2/bin/
JMETER_FILE_NAME=search.jmx
$JMETER_ROOT_PATH/jmeter -n -t ${WORKSPACE}/$JMETER_FILE_NAME -l aggregate_report.jtl -e -o ${WORKSPACE}/report/ --jmeterproperty threadCount=${threadCount} --jmeterproperty loopCount=${loopCount} --jmeterproperty testID=${testID}
 
sshpass -p lx123321$ scp -r root@192.168.3.99:/root/nmon/result/${testID}.nmon ./
mv ${WORKSPACE}/charts/*.* ${WORKSPACE}/
chmod +x ${WORKSPACE}/nmon2html.sh 
java -jar ./rem.jar -r ${testID} $JMETER_FILE_NAME ${threadCount} ${WORKSPACE}/report ${WORKSPACE}/${testID}.nmon
tar -zcvf ${testID}.tar.gz ${WORKSPACE}/report/ ${WORKSPACE}/${testID}.nmon ${WORKSPACE}/${testID}.html
```





