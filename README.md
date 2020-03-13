# hadoop安裝流程(centOS 7)
agenda: 作業系統更新與相關套件安裝 >>> 安裝JDK >>> 安裝Hadoop >>> 安裝Zookeeper
* linux軟體安裝準則：下載 > 解壓縮 > 搬移 > 軟連結 > 設定環境變數(軟連結跟環境變數可互換順序)

* 安裝此類大型系統請使用root帳號操作

## 作業系統更新與相關套件安裝
```bash
# 拿到機器後第一件事：更新
yum update

# 安裝相關套件 wget、git、ntp
yum install wget git ntp
```
## 安裝JDK

Orcale官網要登入才能下載，改用以下網址下載最新版
```bash
# 下載OrcaleJDK 8 到/tmp資料夾
– cd /tmp;wget https://mail-tp.fareoffice.com/java/jdk-8u211-linux-x64.rpm

# 解壓縮OrcaleJDK(解壓縮時會自動搬移到指定位置)
– rpm -ivh /tmp/jdk-8u211-linux-x64.rpm

# 建立軟連結(注意資料夾名稱)
– ln -s /usr/java/jdk1.8.0_211-amd64/ /usr/java/java

# 設定環境變數
- vi /etc/profile
  #	文件最後面加入以下參數
  export JAVA_HOME=/usr/java/java
  export JRE_HOME=$JAVA_HOME/jre
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/rt.jar
  export PATH=$PATH:$JAVA_HOME/bin
```
CentOS 7內建OpenJDK，為求穩定，還是使用Orcale的
```bash
# 如何選擇?
- update-alternatives --config java
# 輸入選用版本之編號

# 檢查使用中之java版本
- java -version
```
## 安裝Hadoop
1. [Hadoop安裝與設定-1: 安全性與防火牆設定]\
  為使後續操作順利，每台電腦間互通沒有障礙，須關閉防火牆(每一台都要做)

SELinux（Security-Enhanced Linux）是為了避免使用者資源的誤用，
在進行程序、檔案等細部權限設定依據的一個核心模組！
遵從最小權限理念，預設情況下所有程序都是被拒絕的。

設定安全性(關閉selinux服務)
- setenforce 0

永久停止selinux安全性
(設定/etc/selinux/config檔案中SELINUX參數，由enforcing改為disabled)
- sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config
[或]
- vi /etc/selinux/config
#	找到SELINUX後把enforcing改成disable
SELINUX=disable
	
關閉防火牆(才能停止)
- systemctl disable firewalld

停止防火牆服務
- systemctl stop firewalld

設定SSH登入免詢問
- echo 'StrictHostKeyChecking no' >> /etc/ssh/ssh_config
[或]
- vi /etc/ssh/ssh_config
#	文件最後面加入以下參數
StrictHostKeyChecking no

重啟sshd服務
- systemctl restart sshd

啟動標準時間服務
- systemctl enable ntpd
- systemctl start ntpd

[Hadoop安裝與設定-2: 叢集主機間的網路打通]
(後面開始每台都有不同操作)
為電腦命名(方便之後操作與Hadoop識別機器)
[[master]]
– hostname master
– vi /etc/hostname
#	全部刪掉改成master
master

[[slaver1]]
– hostname slaver1
– vi /etc/hostname
#	全部刪掉改成slaver1
slaver1
	
[[slaver2]]
– hostname slaver2
– vi /etc/hostname
#	全部刪掉改成slaver2
slaver2
	
[[master]]
設定ip的別名(可用別名連線)
- vi /etc/hosts
#	在文件最後面加入以下設定
xxx.xxx.xx.xxx master
xxx.xxx.xx.xxx slaver1
xxx.xxx.xx.xxx slaver2
	
複製主機名稱與ip設定檔給其他主機
– scp -rp /etc/hosts root@slaver1:/etc/hosts
– scp -rp /etc/hosts root@slaver2:/etc/hosts
	
產生 keygen
- ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ""

把id_rsa.pub的資料寫到authorized_keys最後面
- cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

複製master的key到其他叢集主機(避免每次連線都要密碼)
- ssh-copy-id -i ~/.ssh/id_rsa.pub root@slaver1
- ssh-copy-id -i ~/.ssh/id_rsa.pub root@slaver2

試著從master登入所有主機
- ssh master
- ssh slaver1
- ssh slaver2

[Hadoop安裝與設定-3: Hadoop下載與設定]
[[master]]
下載 Hadoop 2.7.3
- cd /tmp; wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/hadoop-2.7.3.tar.gz

解壓縮
- tar -zxvf /tmp/hadoop-2.7.3.tar.gz

移動檔案
– mv hadoop-2.7.3 /opt

設定環境變數(共三份)
- vi /etc/profile
#	於文件最末新增以下參數
export HADOOP_HOME=/opt/hadoop/
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
- vi /opt/hadoop-2.7.3/libexec/hadoop-config.sh
#	於文件最末新增以下參數
export JAVA_HOME=/usr/java/java
- vi /opt/hadoop-2.7.3/etc/hadoop/hadoop-env.sh
#	於文件最末新增以下參數
export JAVA_HOME=/usr/java/java
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_JOURNALNODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
export HDFS_ZKFC_USER=root
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
	
下載Git clone 設定檔(徐老師提供)
- cd /tmp; git clone https://github.com/orozcohsu/hadoop-2.7.3-ha.git

把設定檔覆蓋至指定路徑，內容包含：
	1.yarn-site.xml
	2.mapred-site.xml
	3.dfs-site.xml
	4.core-site.xml
- cd /tmp/hadoop-2.7.3-ha; cp * /opt/hadoop-2.7.3/etc/hadoop/

建立slaver資訊(列在文件中的主機會成為存資料的空間(datanode))
- vi /opt/hadoop-2.7.3/etc/hadoop/slaves
#	刪掉內容新增以下文字
master
slaver1
slaver2

複製hadoop資料夾給其他主機
– scp -rp /opt/hadoop-2.7.3/ root@slaver1:/opt/hadoop-2.7.3
– scp -rp /opt/hadoop-2.7.3/ root@slaver2:/opt/hadoop-2.7.3

複製profile設定檔(環境變數)給其他主機
– scp -rp /etc/profile root@slaver1:/etc/profile
– scp -rp /etc/profile root@slaver2:/etc/profile

設定軟連結(每台電腦都要做)
- ln -s /opt/hadoop-2.7.3 /opt/hadoop

[Zookeeper HA服務安裝]
選擇slaver1作為namenode的備援(流程同master)
[[slaver1]]
- ssh-keygen -t rsa -f ~/.ssh/id_rsa -P ""
- cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
- ssh-copy-id -i ~/.ssh/id_rsa.pub root@master
- ssh-copy-id -i ~/.ssh/id_rsa.pub root@slaver2
測試連線
- ssh master
- ssh slaver1
- ssh slaver2

[[master]]
下載Zookeeper
– cd /tmp;wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
解壓縮
– tar -zxvf zookeeper-3.4.9.tar.gz
移動檔案
– mv zookeeper-3.4.9 /opt
軟連結設定
– ln -s /opt/zookeeper-3.4.9 /opt/zookeeper
複製設定檔
– cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
設定環境變數
– vi /opt/zookeeper/conf/zoo.cfg
#	修改dataDir的路徑(tmp >> opt)
dataDir=/opt/zookeeper
#	在clientPort=2181這行下面新增以下文字
server.1=master:2888:3888
server.2=slaver1:2888:3888
server.3=slaver2:2888:3888
複製zookeeper資料夾到其他主機
– scp -rp /opt/zookeeper root@slaver1:/opt/zookeeper
– scp -rp /opt/zookeeper root@slaver2:/opt/zookeeper

設定電腦id(每一台編不同數字，1開始)
[[master:1、svlaer1:2、slaver2:3]]
-  vi /opt/zookeeper/myid

[首次啟動]
[[master、svlaer1、slaver2]]
啟動zookeeper服務
- /opt/zookeeper/bin/zkServer.sh start

每台主機都啟動後檢查zookeeper狀態(系統會自動選出leader)
- /opt/zookeeper/bin/zkServer.sh status

啟動journalnode服務
https://blog.csdn.net/Androidlushangderen/article/details/48415073
- hadoop-daemon.sh start journalnode

建立tmp資料夾(for Hadoop環境)
[[master]]
– mkdir -p $HADOOP_HOME/tmp
– mkdir -p $HADOOP_HOME/tmp/dfs/name
– mkdir -p $HADOOP_HOME/tmp/dfs/data
– mkdir -p $HADOOP_HOME/tmp/journal

設定tmp內的操作權限為最大777
– chmod 777 $HADOOP_HOME/tmp

傳送tmp資料夾到其他主機
– scp -rp $HADOOP_HOME/tmp slaver1:/opt/hadoop
– scp -rp $HADOOP_HOME/tmp slaver2:/opt/hadoop

hdfs 格式化
– hdfs namenode -format

ZK 格式化
– hdfs zkfc -formatZK

啟動Hadoop平台
- start-all.sh

將namenode的資料同步至slaver1
[[slaver1]]
– hdfs namenode -bootstrapStandby

啟動slaver1的namenode功能(待命)
[[slaver1]]
– hadoop-daemon.sh start namenode

jps檢查服務是否啟動
- jps
確認是否有以下服務：
[[master]]
JournalNode
NameNode
DataNode
ResourceManager
DFSZKFailoverController
NodeManager
QuorumPeerMain

[[slaver1]]
```
QuorumPeerMain
JournalNode
DataNode
NameNode
DFSZKFailoverController
NodeManager
```

[[slaver2]]
JournalNode
NodeManager
QuorumPeerMain
DataNode

[啟動服務]
[[master、svlaer1、slaver2]]
- /opt/zookeeper/bin/zkServer.sh start
- hadoop-daemon.sh start journalnode
[[master]]
- start-all.sh

[關閉服務]
[[master]]
- stop-all.sh

[[master、svlaer1、slaver2]]
- /opt/zookeeper/bin/zkServer.sh stop


[namenode切換]
[[master or slaver1]]
- hdfs haadmin -failover nn2 nn1  # nn2換成nn1

[namenode啟動]
[[欲重新啟動的設備]]
- hadoop-daemon.sh start namenode


[新增節點]
準備好一個跟slave2一樣的環境(從slaver2複製環境最快)
設定新主機名稱
[[slaver3]]
– hostname slaver3
– vi /etc/hostname
#	slaver2改成slaver3
slaver3

新增連線主機
[[master、svlaer1、slaver2、slaver3]]
- vi /etc/hosts
#	最後面新增新主機的ip跟名稱
xxx.xxx.xxx.xxx lsaver3

修改master的etc/hadoop/slaves文件，添加新增的節點slaver3
[[master]]
- vi /opt/hadoop-2.7.3/etc/hadoop/slaves
#	後面新增slaver3
slaver3

新增zookeeper環境變數
[[master、slaver1、slaver2、slaver3]]
– vi /opt/zookeeper/conf/zoo.cfg
#	在server.3=slaver2:2888:3888這行下面新增以下文字
server.4=slaver3:2888:3888

設定電腦id(每一台編不同數字，1開始)
[[slaver3:4]]
-  vi /opt/zookeeper/myid

重建tmp資料夾
– rm /opt/hadoop/tmp -r -f
– mkdir /opt/hadoop/tmp

啟動 ntp 服務
– systemctl enable ntpd
– systemctl start ntpd

啟動slaver3的服務
[[slaver3]]
- /opt/zookeeper/bin/zkServer.sh start
- hadoop-daemon.sh start journalnode
- /opt/hadoop/sbin/hadoop-daemon.sh start datanode
- /opt/hadoop/sbin/yarn-daemon.sh start nodemanager

檢查啟動狀態
- jps
