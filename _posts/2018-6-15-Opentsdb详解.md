# Opentsdb安装

因为Opentsdb的数据是存储在Hbase中，所以安装Opentsdb之前需要安装Hbase。

1:
直接从 github 上下载 OpenTSDB 的 release 版本的 RPM 包。安装 yum localinstall opentsdb-2.4.0.noarch.rpm。

2：
配置完成后，我们通过下面命令在 HBase 中建立 opentsdb 所需的表。默认情况下 opentsdb 建立的 HBase 表启用了 lzo 压缩。需要开启 Hadoop 中的 lzo 压缩支持， 这里我们直接在下面脚本中把 COMPRESSION 的支持关闭。修改 /usr/share/opentsdb/tools/create_table.sh，设置 COMPRESSION=NONE，并且在文件开始处设置 HBase 所在目录， HBASE_HOME=/home/xxx/hbase-1.1.3。之后执行该脚本，在 HBase 中创建相应的表

3：
修改 OpenTSDB 的配置文件，/etc/opentsdb/opentsdb.conf，例如绑定的端口号等。这里需要注意的是 tsd.core.auto_create_metrics 从 false 改为 true。这样上传数据时会自动创建 metric，否则会提示 Unknown metric 的错误。也可以设置为 false，但是使用 tsdb mkmetric proc.loadavg.1m 来手动添加 metric

opentsdb.conf配置文件如下
```

#以下两行自己添加
HBASE_HOME=/usr/hdp/2.5.3.0-37/hbase
COMPRESSION=NONE

# --------- NETWORK ----------
# The TCP port TSD should use for communications
# *** REQUIRED ***
tsd.network.port = 8383

# The IPv4 network address to bind to, defaults to all addresses
# tsd.network.bind = 0.0.0.0

# Disable Nagel's algorithm, default is True
#tsd.network.tcp_no_delay = true

# Determines whether or not to send keepalive packets to peers, default
# is True
#tsd.network.keep_alive = true

# Determines if the same socket should be used for new connections, default
# is True
#tsd.network.reuse_address = true

# Number of worker threads dedicated to Netty, defaults to # of CPUs * 2
#tsd.network.worker_threads = 8

# Whether or not to use NIO or tradditional blocking IO, defaults to True
#tsd.network.async_io = true

# ----------- HTTP -----------
# The location of static files for the HTTP GUI interface.
# *** REQUIRED ***
tsd.http.staticroot = /usr/share/opentsdb/static/

# Where TSD should write it's cache files to
# *** REQUIRED ***
tsd.http.cachedir = /tmp/opentsdb

# --------- CORE ----------
# Whether or not to automatically create UIDs for new metric types, default
# is False
tsd.core.auto_create_metrics = true

# Full path to a directory containing plugins for OpenTSDB
tsd.core.plugin_path = /usr/share/opentsdb/plugins

# --------- STORAGE ----------
# Whether or not to enable data compaction in HBase, default is True
#tsd.storage.enable_compaction = true

# How often, in milliseconds, to flush the data point queue to storage,
# default is 1,000
# tsd.storage.flush_interval = 1000

# Name of the HBase table where data points are stored, default is "tsdb"
#tsd.storage.hbase.data_table = tsdb

# Name of the HBase table where UID information is stored, default is "tsdb-uid"
#tsd.storage.hbase.uid_table = tsdb-uid

# Path under which the znode for the -ROOT- region is located, default is "/hbase"
#这个值必须填写Hbase的属性  zookeeper.znode.parent  的值
tsd.storage.hbase.zk_basedir = /hbase-unsecure

# A comma separated list of Zookeeper hosts to connect to, with or without
# port specifiers, default is "localhost"
tsd.storage.hbase.zk_quorum = m2.wss.com,m3.wss.com,m1.wss.com

```
4：
启动 OpenTSDB，service opentsdb start

5：
通过浏览器访问 http://x.x.x.x:8383 查看是否安装成功

# Opentsdb插入数据和查询

1：利用python API插入连续的数据

python代码如下：
```

import time
import math
import requests

def get_value(num):
    return math.sin(num)+1


def send_json(json, s):
    r=s.post("http://localhost:8383/api/put?details", json=json)
    return r.text


def main():
    s = requests.Session()
    #hbase的Timestamp需要是13位数字
    a = 1348967654765
    ls = []
    for i in range(1, 100):
        json = {
            "metric": "metricD",
            "timestamp": a,
            "value": get_value(i),
            "tags": {
                "host": "1",
            }
        }
        i += 0.01
        a += 1
        ls.append(json)
        if len(ls) == 10:
            send_json(ls, s)
            ls = []
    send_json(ls, s)
    ls = []


if __name__ == "__main__":
    start = time.time()
    main()
    print time.time()-start
```

2:进入Web界面查询相应的metric，看数据是否插入成功
