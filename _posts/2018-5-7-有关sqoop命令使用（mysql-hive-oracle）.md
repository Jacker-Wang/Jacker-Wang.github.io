
以下所有步骤针对的Sqoop版本 #Sqoop Version 1.4.6.2.5.3.0-37
# Oracle》》》Hive

1 从Oracle全量导入到Hive

从Oracle全量导入到Hive，Hive里面自动建表，建表的默认数据类型映射如下（不同版本的sqoop的数据类型映射不同）：
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

2 从Oracle全量导入到Hive（指定Oracle到Hive的列映射）

```
sqoop import  --connect jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=132.246.24.38)(PORT=8888))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=wlwbf94)))
--username app --password IOT_bigbang   --hcatalog-database wss -m 1
--hcatalog-table  CFG_GHX_IMEI  --query
"select
订单编号 as  order_no ,
订单备注 as remark ,
下单时间 as  order_date ,
供货商名称 as  supplier_name ,
零售商名称 as  tradesman_name ,
渠道视图主体编码 as  channel_code ,
门店编码 as   shoping_code  ,
门店名称 as   shoping_name  ,
发货时间 as    send_date ,
出库仓库 as    storehouse ,
省分 as    province ,
地市 as    city_name ,
区县 as    district ,
编码25位 as    code ,
品牌 as  brand   ,
产品类型 as    prod_type ,
型号 as   version  ,
颜色 as    colour ,
串码 as    imei ,
经营地市 as    oper_city ,
经营区县 as    oper_district ,
现返标识 as    return_flag
 from CFG_GHX_IMEI where \$CONDITIONS"
--hcatalog-storage-stanza 'stored as orc tblproperties ("orc.compress"="SNAPPY")'
-- --default-character-set=utf-8

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
