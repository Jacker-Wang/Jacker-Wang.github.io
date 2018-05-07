
以下所有步骤针对的Sqoop版本 #Sqoop Version 1.4.6.2.5.3.0-37
# Oracle》》》Hive

1 从Oracle全量导入导Hive

从Oracle全量导入到Hive，Hive里面ui自动建表，建表的默认数据类型映射如下（不同版本的sqoop的数据类型映射不同）：
```
Oracle         Hive

INTEGER       DECIMAL(38,0)
NUMERICAL     DECIMAL(10,0)

CHAR(N)       CHAR(N)
VARCHAR2(N)   VARCHAR(N)

TIMESTAMP(6)  STRING

```
因为字段类型映射导致的值错误需要自己指定字段的类型。
```
#Sqoop Version 1.4.6.2.5.3.0-37

#Oracle
DefaultOraStr="jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=132.246.24.38)(PORT=8888))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=wlwbf94)))"
DefaultUser=app
DefaultPassword=IOT_bigbang


HiveDatabase=wss
table=WLW_PD_INST_MONTHLY

#自定义映射的数据类型，因为sqoop的bug导致(据说此Bug是在1.4.7才修复)，xxx=decimal(m,n)需要修改为xxx=decimal(m%2Cn)

mapColumnhive=FINISH_TIME=DATE,TJDT=DATERCVR_AMNT=DECIMAL(20),CALL_DRTN=DECIMAL(30%2C2),MSGCNT=DECIMAL(30%2C2)

sqoop import  --connect ${DefaultOraStr}   --username ${DefaultUser} --password ${DefaultPassword} --table ${table} --hcatalog-database ${HiveDatabase} -m 1 --create-hcatalog-table  --hcatalog-table ${table}  ${mapColumnhive} --hcatalog-storage-stanza 'stored as orc tblproperties ("orc.compress"="SNAPPY")' -- --default-character-set=utf-8

```

# Hive》》》Mysql
以下2中export方式均需要提前在Mysql中建表，建表的字段类型一定要兼容（尽可能让Mysql中的字段类型范围更大）

1 Hive全表导出到Mysql


```
#mysql info
mysql_db=wss_report
mysql_user=wss
mysql_password=wss


sqoop export --connect  "jdbc:mysql://10.251.44.64:3307/${mysql_db}?useUnicode=true&characterEncoding=utf-8"  --username ${mysql_user} --password ${mysql_password} --table ${table}   --hcatalog-database wss --hcatalog-table ${table} -- --default-character-set=utf-8



```

2 Hive分区表导出指定分区到Mysql

```
#mysql info
mysql_db=wss_report
mysql_user=wss
mysql_password=wss

sqoop export --connect  "jdbc:mysql://10.251.44.64:3307/${mysql_db}?useUnicode=true&characterEncoding=utf-8"  --username ${mysql_user} --password ${mysql_password} --table ${table}  --hive-partition-key yyyymmdd --hive-partition-value '20180414'   --hcatalog-database wss --hcatalog-table ${table} -- --default-character-set=utf-8

```
