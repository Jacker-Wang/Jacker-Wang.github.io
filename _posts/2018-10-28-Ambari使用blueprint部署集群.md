

# Blueprint方式部署说明

  Blueprint方式部署适用于集群的自动化部署，特别是上云的场景。官方资料请参考 [Blueprint参考]

  [Blueprint参考]: https://cwiki.apache.org/confluence/display/AMBARI/Blueprints+-+2.4.0 "Blueprint参考"

# 部署步骤

## 环境说明

3台 centos7 ，主机名如下（对应的ip映射都在/etc/hosts中配置好）：

t3m1.ecloud.com

t3m2.ecloud.com

t3m3.ecloud.com

## 创建蓝图

Blueprint Post请求发送的请求体全部采用JSON的格式。

**Blueprint模板如下：**

blueprint.json

```
{
  "configurations": [
    {}
  ],
  "Blueprints": {
    "stack_version": "3.0",
    "stack_name": "HDP",
    "blueprint_name": "cc1"
  },
  "host_groups" : [
    {
      "name" : "master",
      "configurations": [
        {}
      ],
      "components": [{
                        "name": "SECONDARY_NAMENODE"
                }, {
                        "name": "NAMENODE"
                }, {
                        "name": "METRICS_MONITOR"
                }, {
                        "name": "METRICS_COLLECTOR"
                        "name": "METRICS_GRAFANA"
                }, {
                        "name": "HBASE_MASTER"
                }, {
                        "name": "ZOOKEEPER_SERVER"
                }, {
                        "name": "DATANODE"
                }, {
                        "name": "HBASE_REGIONSERVER"
                }, {
                        "name": "PHOENIX_QUERY_SERVER"
                }, {
                        "name": "HDFS_CLIENT"
                }, {
                        "name": "HBASE_CLIENT"
                }, {
                        "name": "ZOOKEEPER_CLIENT"
                }],
      "cardinality" : "1"
    },
    {
      "name" : "slaves",
      "configurations": [
        {}
      ],
      "components": [{
                        "name": "ZOOKEEPER_SERVER"
                }, {
                        "name": "METRICS_MONITOR"
                }, {
                        "name": "DATANODE"
                }, {
                        "name": "HBASE_REGIONSERVER"
                }, {
                        "name": "HDFS_CLIENT"
                }, {
                        "name": "HBASE_CLIENT"
                }, {
                        "name": "ZOOKEEPER_CLIENT"
                }],
      "cardinality" : "1+"
    }
  ],
  "settings": [
    {
      "recovery_settings": [
                {
                    "recovery_enabled": "true"
                }
            ]
        },
        {
            "service_settings": [
                {
                    "recovery_enabled": "true",
                    "name": "AMBARI_METRICS"
                }
            ]
        },
        {
            "component_settings": [
                {
                    "recovery_enabled": "true",
                    "name": "METRICS_COLLECTOR"
                }
            ]
        }
  ]
}
```
**发送post请求**
```
curl -i -H "X-Requested-By: ambari" -X POST -u admin:admin http://localhost:8080/api/v1/blueprints/cc1 -d @blueprint.json
```

## 设置本地yum源

**请求体如下**

vdf.json

```
{
 "VersionDefinition": {
   "version_url": "http://192.168.0.47/ambari/blueprint/HDP-3.0.0.0-1634.xml"
 }
}
```

以上 version_url 的值必须能够访问到,HDP-3.0.0.0-1634.xml 内容如下

```
<?xml version="1.0"?>
<repository-version xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="version_definition.xsd">
  <release>
    <type>STANDARD</type>
    <stack-id>HDP-3.0</stack-id>
    <version>3.0.0.0</version>
    <build>1634</build>
    <compatible-with>3\.\d+\.\d+\.\d+</compatible-with>
    <release-notes>http://example.com</release-notes>
    <display>HDP-3.0.0.0</display>
  </release>
  <manifest>
    <service id="SPARK2-231" name="SPARK2" version="2.3.1"/>
    <service id="HBASE-200" name="HBASE" version="2.0.0"/>
    <service id="PIG-0160" name="PIG" version="0.16.0"/>
    <service id="DRUID-0121" name="DRUID" version="0.12.1"/>
    <service id="ATLAS-100" name="ATLAS" version="1.0.0"/>
    <service id="OOZIE-431" name="OOZIE" version="4.3.1"/>
    <service id="HIVE-310" name="HIVE" version="3.1.0"/>
    <service id="ZEPPELIN-080" name="ZEPPELIN" version="0.8.0"/>
    <service id="RANGER-110" name="RANGER" version="1.1.0"/>
    <service id="RANGER_KMS-110" name="RANGER_KMS" version="1.1.0"/>
    <service id="ZOOKEEPER-346" name="ZOOKEEPER" version="3.4.6"/>
    <service id="ACCUMULO-170" name="ACCUMULO" version="1.7.0"/>
    <service id="KAFKA-101" name="KAFKA" version="1.0.1"/>
    <service id="HDFS-310" name="HDFS" version="3.1.0"/>
    <service id="YARN-310" name="YARN" version="3.1.0"/>
    <service id="MAPREDUCE2-310" name="MAPREDUCE2" version="3.1.0"/>
    <service id="TEZ-091" name="TEZ" version="0.9.1"/>
    <service id="STORM-121" name="STORM" version="1.2.1"/>
    <service id="SQOOP-147" name="SQOOP" version="1.4.7"/>
    <service id="KNOX-100" name="KNOX" version="1.0.0"/>
  </manifest>
  <available-services/>
  <repository-info>
    <os family="redhat7">
      <package-version>3_0_0_0_*</package-version>
      <repo>
        <baseurl>http://192.168.0.47/ambari/HDP/centos7/3.0.0.0-1634</baseurl>
        <repoid>HDP-3.0</repoid>
        <reponame>HDP</reponame>
        <unique>true</unique>
      </repo>
      <repo>
        <baseurl>http://192.168.0.47/ambari/HDP-GPL/centos7/3.0.0.0-1634</baseurl>
        <repoid>HDP-3.0-GPL</repoid>
        <reponame>HDP-GPL</reponame>
        <unique>true</unique>
        <tags>
          <tag>GPL</tag>
        </tags>
      </repo>
      <repo>
        <baseurl>http://192.168.0.47/ambari/HDP-UTILS/centos7/1.1.0.22</baseurl>
        <repoid>HDP-UTILS-1.1.0.22</repoid>
        <reponame>HDP-UTILS</reponame>
        <unique>false</unique>
      </repo>
    </os>
  </repository-info>
</repository-version>
```


**发送Post请求**
```
curl -i -H "X-Requested-By: ambari" -X POST -u admin:admin http://localhost:8080/api/v1/version_definitions -d @vdf.json
```


## 创建集群

**集群模板如下**

clustertemplate.json

```
{
    "blueprint": "cc1",
    "repository_version_id": 1,
    "repository_version": "3.0.0.0-1634",
    "default_password": "admin",
    "config_recommendation_strategy": "NEVER_APPLY",
    "provision_action": "INSTALL_AND_START",
    "configurations": [ ],
    "host_groups": [
        {
            "name": "master",
            "hosts": [
                {
                    "fqdn": "t3m1.ecloud.com"
                }
            ]
        },
        {
            "name": "slaves",
            "hosts": [
                {
                    "fqdn": "t3m2.ecloud.com"
                },
                {
                    "fqdn": "t3s3.ecloud.com"
                }
            ]
        }
    ],
    "Clusters": {
        "cluster_name": "cc1"
    }
}
```
**发送post请求**
```
curl -i -H "X-Requested-By: ambari" -X POST -u admin:admin http://localhost:8080/api/v1/clusters/cc1 -d @clustertemplate.json
```

# 登录Ambari Web管理界面查看安装和启动进度

 发送集群模版的请求后就可以在web后台查看组件安装和启动的进度。

 ------

 _JackerWang 于2018年秋（10月28日）上午广州_
