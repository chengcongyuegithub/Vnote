# 在Ubuntu下搭建solr环境
今天一天的时间基本都在搭建solr的环境,因为太久没有使用过,差不多就是重新学一遍.现在把今天一天的成果总结一下
## 准备
solr是全文检索用的,它是自带jetty这个服务器的,也就是可以不用配置,直接就可以启动.
我们要做的就是将服务器改为tomcat.
* jdk1.8的环境
* solr-4.9.1.tgz
* tomcat7
* 中文的分词器
* mysql的jar包
## 搭建初始环境

首先明确我们要搭建服务器,通过tomcat跑的,然后其中的core可以类比于数据库中的一张张表.
本身的solr就有服务器,我们要讲solr-4.9.1中有用的部分放入到tomcat的webapps下,最后我们总的文件环境如下:
```
chengcongyue@chengcongyue:/usr/local/solrresource$ ls
solr-4.9.1  solrhome  tomcat
```
其中可以认为solr-4.9.1是没有用的,因为在运行时,并没有用到它.

然后我们就正式开始了,我在搭建的期间又更加的熟悉了linux的命令...
* **首先我们把solr-4.9.1中的dist下的war放入到tomcat的webapps下**.
```
chengcongyue@chengcongyue:/usr/local/solrresource$ cd solr-4.9.1/
chengcongyue@chengcongyue:/usr/local/solrresource/solr-4.9.1$ ls
CHANGES.txt  dist  example   LICENSE.txt  README.txt
contrib      docs  licenses  NOTICE.txt   SYSTEM_REQUIREMENTS.txt
chengcongyue@chengcongyue:/usr/local/solrresource/solr-4.9.1$ cd dist
chengcongyue@chengcongyue:/usr/local/solrresource/solr-4.9.1/dist$ ls
solr-4.9.1.war                           solr-map-reduce-4.9.1.jar
solr-analysis-extras-4.9.1.jar           solr-morphlines-cell-4.9.1.jar
solr-cell-4.9.1.jar                      solr-morphlines-core-4.9.1.jar
solr-clustering-4.9.1.jar                solr-solrj-4.9.1.jar
solr-core-4.9.1.jar                      solr-test-framework-4.9.1.jar
solr-dataimporthandler-4.9.1.jar         solr-uima-4.9.1.jar
solr-dataimporthandler-extras-4.9.1.jar  solr-velocity-4.9.1.jar
solrj-lib                                test-framework
solr-langid-4.9.1.jar
```
将solr-4.9.1.war复制到tomcat下,复制完成如下图:
```
chengcongyue@chengcongyue:/usr/local/solrresource/tomcat/webapps$ ls
docs  examples  host-manager  manager  ROOT  solr  solr.war
chengcongyue@chengcongyue:/usr/local/solrresource/tomcat/webapps$ 
```
并且更加名称为:solr.war,其中的solr是怎么来的呢,我们这个时候运行tomcat,solr.war就自动解压称为了solr.这个就相当于是solr的项目.

* **然后我们要知道Core所在的位置,也就是类比于数据库(存放索引的地方)(solr)**
我们知道在web.xml中配置,也就是solr这个"网站"中的WEB-INF中的web.xml
我们在其中添加这么一段内容:
```
 <env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>/usr/local/solrresource/solrhome</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
```
有了这么一段内容,我们这个tomcat服务器就知道了solrhome的位置.其中有好多的collection.
* **然后就是拷贝jar包,网上有好多的拷贝方法,就重要的就是如下:**
```
chengcongyue@chengcongyue:/usr/local/solrresource/solr-4.9.1/example/lib/ext$ lsjcl-over-slf4j-1.7.6.jar  log4j-1.2.17.jar     slf4j-log4j12-1.7.6.jar
jul-to-slf4j-1.7.6.jar    slf4j-api-1.7.6.jar
```
将这些jar包拷贝到我们的网站也就是solr的中的lib下:
```
chengcongyue@chengcongyue:/usr/local/solrresource/tomcat/webapps/solr/WEB-INF/lib$ ls
antlr-runtime-3.5.jar                lucene-highlighter-4.9.1.jar
asm-4.1.jar                          lucene-join-4.9.1.jar
asm-commons-4.1.jar                  lucene-memory-4.9.1.jar
commons-cli-1.2.jar                  lucene-misc-4.9.1.jar
commons-codec-1.9.jar                lucene-queries-4.9.1.jar
commons-configuration-1.6.jar        lucene-queryparser-4.9.1.jar
commons-fileupload-1.2.1.jar         lucene-spatial-4.9.1.jar
commons-io-2.3.jar                   lucene-suggest-4.9.1.jar
commons-lang-2.6.jar                 mysql-connector-java-5.1.39.jar
concurrentlinkedhashmap-lru-1.2.jar  noggit-0.5.jar
dom4j-1.6.1.jar                      org.restlet-2.1.1.jar
guava-14.0.1.jar                     org.restlet.ext.servlet-2.1.1.jar
hadoop-annotations-2.2.0.jar         protobuf-java-2.5.0.jar
hadoop-auth-2.2.0.jar                slf4j-api-1.7.6.jar
hadoop-common-2.2.0.jar              slf4j-log4j12-1.7.6.jar
hadoop-hdfs-2.2.0.jar                solr-analysis-extras-4.9.1.jar
hppc-0.5.2.jar                       solr-cell-4.9.1.jar
httpclient-4.3.1.jar                 solr-clustering-4.9.1.jar
httpcore-4.3.jar                     solr-core-4.9.1.jar
httpmime-4.3.1.jar                   solr-dataimporthandler-4.9.1.jar
IKAnalyzer2012FF_u1.jar              solr-dataimporthandler-extras-4.9.1.jar
jcl-over-slf4j-1.7.6.jar             solr-langid-4.9.1.jar
joda-time-2.2.jar                    solr-map-reduce-4.9.1.jar
jul-to-slf4j-1.7.6.jar               solr-morphlines-cell-4.9.1.jar
log4j-1.2.17.jar                     solr-morphlines-core-4.9.1.jar
lucene-analyzers-common-4.9.1.jar    solr-solrj-4.9.1.jar
lucene-analyzers-kuromoji-4.9.1.jar  solr-test-framework-4.9.1.jar
lucene-analyzers-phonetic-4.9.1.jar  solr-uima-4.9.1.jar
lucene-codecs-4.9.1.jar              solr-velocity-4.9.1.jar
lucene-core-4.9.1.jar                spatial4j-0.4.1.jar
lucene-expressions-4.9.1.jar         wstx-asl-3.2.7.jar
lucene-grouping-4.9.1.jar            zookeeper-3.4.6.jar
```
拷贝到这里面,这样服务器就搭建好了.
* **然后就是创建一个原本的collection(core)(类比数据)**
创建一个solrhome,然后将/usr/local/solrresource/solr-4.9.1/example/solr/中的内容考入到solrhome中.
这样就完成了.

## 中文分词器的使用
中文分词器,就是一个xml和一个jar,分别导入到tomcat这个项目就可以了,其中xml需要到如classes中,我们需要自己创建这个classes文件夹
分词器在schema中的配置
```
<fieldType name="text_ik" class="solr.TextField">
              <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>
```
**这个时候我们就可以进入到solr的网页了,我们如果要创建core会报错,我们可以先在solrhome创建想要的core的名字的文件夹,将collection1的conf全部复制到这个新的文件夹中,然后在在网页上创建.否则会报错,估计就是因为没有使用内置的jetty服务器**
## 数据库导入
这个比较复杂...
这个主要操作的是solrhome中的core,我们以自己创建的easyzhihu为例子,
```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
 <lst name="defaults">
 <str name="config">data-config.xml</str>
 </lst>
</requestHandler>
```
其中有个data-config.xml的指向,这个就是数据库的配置文件,我们需要配置一个
```
<?xml version="1.0" encoding="UTF-8"?>
<dataConfig>
   <dataSource name="source1" type="JdbcDataSource"
           driver="com.mysql.jdbc.Driver" url="jdbc:mysql://127.0.0.1:3306/easyzhihu" user="root" password="mysql"
           ></dataSource>
   <document>
      <entity name="question" dataSource="source1" query="select * from question">
              <field name="zh_title" column="title"></field>
              <field name="zh_content" column="content"></field>
      </entity>
   </document>
</dataConfig>
```
一个document代表这一条sql语句.
然后我们需要在配置schema.xml
```
 <field name="zh_title" type="text_ik" indexed="true" stored="true"/>
 <field name="zh_content" type="text_ik" indexed="true" stored="true"/>
```
最后讲mysql的驱动导入到tomcat下的solr下
复制solr-4.9.1/example/dist/*.jar 至tomcat/webapps/solr/WEB-INF/lib中.
然后就完成了.