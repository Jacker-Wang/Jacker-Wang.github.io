# 常用的Kerberos命令

```
#进入KDC的管理命令
kadmin.local （用输入密码，但是需要在KDC本地运行）
kadmin  (需要输入密码)

#进入管理命令行之后列出所有的principals
kadmin.local: listprincs

```

# KDC Server 搭建

1 关闭防火墙，关闭SELINUX

2 安装组件
```
yum install krb5-server krb5-libs krb5-workstation
```

# KDC Serber 配置


vim /etc/krb5.conf （服务端和客户端都会用到的配置文件）
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 720h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = ENABLE.COM
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
ENABLE.COM = {
  kdc = KDC server主机名或者IP
  admin_server = KDC server主机名或者IP
}

[domain_realm]
 .enable.com = ENABLE.COM
 enable.com = ENABLE.COM
```

vim /var/kerberos/krb5kdc/kdc.conf （只是KDC服务端使用的配置）
```
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 ENABLE.COM ={
  master_key_type = des3-hmac-sha1
  database_name = /var/kerberos/krb5kdc/principal
  key_stash_file = /var/kerberos/krb5kdc/.ENABLE.COM
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  kdc_ports = 750,88
  max_life = 2d 0h 2m 0s
  max_renewable_life = 500d 0h 0m 0s
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

# 使用kdb5_util创建数据库
```
kdb5_util create -r DCS.COM –s
```
过程中输入的密码请记住！！！

# 启动服务以及设置开机自启动

```
 service krb5kdc start
 service kadmin start
 chkconfig krb5kdc on
 chkconfig kadmin on
```

# 创建KDC管理
```
kadmin.local
kadmin.local: addprinc admin/admin
kadmin.local: 输入密码
kadmin.local: ktadd -k /var/kerberos/krb5kdc/kadm5.keytab kadmin/admin

```

# 配置节点JCE（每个节点都要配置）
下载地址：

jdk1.8
```
http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html
```
jdk1.7
```
http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
```
```
unzip -o -j -q jce_policy-8.zip -d $JAVA_HOME/jre/lib/security/
```

# 通过Ambari启用kerberos
Ambari UI界面操作


1 选择使用已存在的KDC

2 填写KDC的host

最后进行检测。

等检测通过之后，会停止所有服务，自动修改相关配置文件，然后等待服务都启动起来就可以了。

# 创建princ并且添加keytab

以下是批量添加principal的脚本
```
#需要添加的principal的用户名称
user=alogic
#kdc的root/admin的密码
passwd=1234
hostname=$(hostname -f)

ssh 192.168.1.137 -p 2200 -t $*
"

sudo /usr/bin/kadmin -p root/admin -w ${passwd} -q 'ank -randkey  ${user}/${hostname}';

sudo /usr/bin/kadmin -p root/admin -w ${passwd} -q 'xst -k /etc/security/keytabs/${user}.app.keytab   ${user}/${hostname}';

sudo su - alogic -c 'kinit -kt /etc/security/keytabs/${user}.app.keytab ${user}/${hostname};

klist'

"

```
