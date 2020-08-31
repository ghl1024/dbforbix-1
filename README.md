# dbforbix
monitor  db2(9/10) mysql oracle(10/11) and  zabbix3.x zabbix4.x zabbix5.x can use   and centos6/7 can run 

本版本基于 smartmarmot/DBforBIX Version 2.2-beta 


参考资料 http://www.smartmarmot.com/wiki/index.php?title=DBforBIX


准备步骤
Download DBforBIX to your Zabbix Server （被监控服务器上）
On your Zabbix server, unzip DBforBIX to: /opt/dbforbix
Install the JAVA JVM >= 1.7 (both Oracle and OpenJDK works properly)   #rpm安装
Install JSVC (Java daemon launcher)   #编译安装 
Copy config.properties.sample to config.properties and change it adding your databases.

# cp  config.properties.sample to config.properties 

####需要安装gcc 
checking for gcc... no
checking for cc... no
checking for cl.exe... no

yum -y install gcc-c++ gcc
#编译安装jsvc  上传commons-daemon-1.0.15-src.tar.gz  使用root用户执行
  cd	/root/commons-daemon-1.0.15-src/src/native/unix
 sh support/buildconf.sh  
 如果报错 autoconf: command not found ,则 yum -y install autoconf 

./configure --with-java=/usr/java  --with-os-type=jdk1.7.0_80 
make
-----172.16.131.142
./configure --with-java=/usr/local/java/jdk1.8.0_131/ --with-os-type=bin 
make
-----
##验证是否安装成功
/home/zabbix/commons-daemon-1.0.15-src/src/native/unix/jsvc -help
vi  /opt/dbforbix/dbforbix.sh 替换路径  (find / -name jsvc )

验证jsvc是否安装好：
EXEC=`whereis -b -B /bin /sbin /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin /root/commons-daemon/commons-daemon-1.0.15-src/src/native/unix  -f jsvc | awk '{ print $2;}'`

然后：  sh  dbforbix.sh start 

Initd  #此步有误，不存在init.d的目录，不要配置
Now for all the distribution that are still using initd use the following procedure:
Copy file /opt/dbforbix/init.d/dbforbix to /etc/init.d/dbforbix

# cp /opt/dbforbix/init.d/dbforbix   /etc/init.d/dbforbix       

Grant execute permissions to the following files:
/etc/init.d/dbforbix
/opt/dbforbix/run.sh
For this example on RedHat, run:
#  chkconfig -add dbforbix

Systemd  #redhat 没安装此命令 不用配置
If your distribution uses systemd, like most of the nowadays here are the steps:
Copy the systemd included files
# mkdir -p /etc/systemd/system/
# cp systemd/dbforbix.service /etc/systemd/system/dbforbix.service

Notify systemd that a new dbforbix.service file exists by executing the following command as root:
# systemctl daemon-reload
# systemctl start dbforbix.service

To configure the service to start at each boot run (from root console):
# systemctl enable name.service



启动服务进行测试  数据库是否可以连接
If you would like to test dbforbix from command line you can simply type:
cd /opt/dbforbix   
 java -jar dbforbix.jar -a start -C /opt/dbforbix


如果报错： java.lang.ClassNotFoundException: com.mysql.jdbc.Driver
将jdbc 连接包上传到 里
/usr/java/jdk1.7.0_80/jre/lib/ext

vi dbforbix.sh  里的 user=dbforbix 更改为 user=zabbix 
chmod -R 777 ./logs/*    #否者日志写入报错
#启动服务
 sh dbforbix.sh start 

===============================================
oracle 数据库的设置
Install steps for Oracle
Create a User (ZABBIX) for DBforBIX to access your Oracle Database. You can use the following script
CREATE USER ZABBIX
  IDENTIFIED BY zabbix 
  DEFAULT TABLESPACE SYSTEM
  TEMPORARY TABLESPACE TEMP
  PROFILE DEFAULT
  ACCOUNT UNLOCK;
  -- 2 Roles for ZABBIX
  GRANT CONNECT TO ZABBIX;
  GRANT RESOURCE TO ZABBIX;
  ALTER USER ZABBIX DEFAULT ROLE ALL;
  -- 5 System Privileges for ZABBIX
  GRANT SELECT ANY TABLE TO ZABBIX;
  GRANT CREATE SESSION TO ZABBIX;	
  GRANT SELECT ANY DICTIONARY TO ZABBIX;
  GRANT UNLIMITED TABLESPACE TO ZABBIX;
 GRANT SELECT ANY DICTIONARY TO ZABBIX;


NOTE : If you are using Oracle 11g, you will need to add the following:
  exec dbms_network_acl_admin.create_acl(acl => 'resolve.xml',description => 'resolve acl', principal =>'ZABBIX',is_grant => true, privilege => 'resolve');
  exec dbms_network_acl_admin.assign_acl(acl => 'resolve.xml', host =>'*');

You can verify the above is correct by running:
  select utl_inaddr.get_host_name(’127.0.0.1′) from dual;

NOTE: To create a User (ZABBIX) for DBforBIX with MINIMAL grants you can use the following script:
 CREATE USER ZABBIX
  IDENTIFIED BY <REPLACE WITH PASSWORD>
  DEFAULT TABLESPACE USERS
  TEMPORARY TABLESPACE TEMP
  PROFILE DEFAULT
  ACCOUNT UNLOCK;
  GRANT ALTER SESSION TO ZABBIX;
  GRANT CREATE SESSION TO ZABBIX;
  GRANT CONNECT TO ZABBIX;
  ALTER USER ZABBIX DEFAULT ROLE ALL;
  GRANT SELECT ON V_$INSTANCE TO ZABBIX;
  GRANT SELECT ON DBA_USERS TO ZABBIX;
  GRANT SELECT ON V_$LOG_HISTORY TO ZABBIX;
  GRANT SELECT ON V_$PARAMETER TO ZABBIX;
  GRANT SELECT ON SYS.DBA_AUDIT_SESSION TO ZABBIX;
  GRANT SELECT ON V_$LOCK TO ZABBIX;
  GRANT SELECT ON DBA_REGISTRY TO ZABBIX;
  GRANT SELECT ON V_$LIBRARYCACHE TO ZABBIX;
  GRANT SELECT ON V_$SYSSTAT TO ZABBIX;
  GRANT SELECT ON V_$PARAMETER TO ZABBIX;
  GRANT SELECT ON V_$LATCH TO ZABBIX;
  GRANT SELECT ON V_$PGASTAT TO ZABBIX;
  GRANT SELECT ON V_$SGASTAT TO ZABBIX;
  GRANT SELECT ON V_$LIBRARYCACHE TO ZABBIX;
  GRANT SELECT ON V_$PROCESS TO ZABBIX;
  GRANT SELECT ON DBA_DATA_FILES TO ZABBIX;
  GRANT SELECT ON DBA_TEMP_FILES TO ZABBIX;
  GRANT SELECT ON DBA_FREE_SPACE TO ZABBIX;
  GRANT SELECT ON V_$SYSTEM_EVENT TO ZABBIX;



vi  config.properties

ZabbixServer.1.Address=192.168.70.138
ZabbixServer.1.Port=10051

DB.DB.Type=oracle
DB.DB.Name=node1   ###这个是被监控服务器的名称，必须和服务端监控的机器名称一样
DB.DB.Url=jdbc:oracle:thin:@localhost:1521:sales
DB.DB.User=zabbix
DB.DB.Password=zabbix
DB.DB.MaxWait=10
DB.DB.MaxSize=10
DB.DB.MaxIdle=1
DB.DB.ItemFile=oracle
#DB.DB4.Persistence=TRUE

需要注意oracle RAC的配置需要在
conf.property 里修改
DB.DB.Url=jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS = (PROTOCOL = TCP)(HOST = 1.2.3.4)(PORT = 1521)))(CONNECT_DATA=(SERVICE_NAME=YOUR_SERVICE)))


-----------------------------------------------------------
-------批量脚本-------------------

yum -y install gcc-c++ gcc
cd /home/zabbix/
tar -xvf jdbc_jdk1.8.0_131.tar 
tar -xvf commons-daemon-1.0.15-src.tar.gz
cd ./commons-daemon-1.0.15-src/src/native/unix
sh support/buildconf.sh  
./configure --with-java=/home/zabbix/jdk1.8.0_131/ --with-os-type=bin 
make

mkdir /home/zabbix/log
chmod 777  /home/zabbix/log
mv  /tmp/dbforbix_ora.tar  .
tar -xvf dbforbix_ora.tar
cd /home/zabbix/dbforbix
IP=`ifconfig | grep -A1 eth | tail -1 | awk '{print $2}'|awk -F ':' '{print $2}'`
sed -i.bak 's/x.x.x.x/'"$IP"' ./conf/config.properties
sh  dbforbix.sh start 


#===测试数据库连接
#cd /home/zabbix/dbforbix
#export JAVA_HOME=/home/zabbix/jdk1.8.0_131/
#/home/zabbix/jdk1.8.0_131/bin/java -jar dbforbix.jar -a start -C /home/zabbix/dbforbix

===============================
---mysql

mysql -uroot -h172.16.131.142 -p123456
 CREATE USER 'zabbix'@'%' IDENTIFIED BY 'zabbix';
 GRANT SELECT, SHOW VIEW ON *.* TO 'zabbix'@'%';
grant select on *.* to zabbix@localhost; 
grant all privileges on *.* to 'root'@'localhost' identified by '123456' with grant option;
flush privileges;
开启远程访问   
mysql -uzabbix -h localhost -p
rpm 包安装mysql5.6.note



注意点： 
oracle数据库表空间如果不设置成自动扩展的则，表空间监控的触发器会失效。因为 dba_data_files的maxbytes 不会有设置的值。 

#===注意自己添加的Item , 需要写清列名。并且取值只取最后一个列值。 
/home/zabbix/dbforbix/items



常见问题：
启动日志里  dbfor.err 或者  dbforbix.log  中出现 java.lang.classnotfoundException  sun/io/UnkonwCharacterException
原因： jdbc的jar包需要更新。  举例 db2的jar包放到
 jdk1.8._131的  jre/lib/ext/目录下  db2jcc4.jar  db2jcc.jar  db2jcc_license_cu.jar 





