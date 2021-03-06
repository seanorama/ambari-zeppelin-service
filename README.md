#### An Ambari Service for Zeppelin
Ambari service for easily installing and managing [Apache Zeppelin](http://zeppelin.incubator.apache.org/) on [HDP](http://hortonworks.com/hdp/) cluster

See [blog](http://hortonworks.com/blog/introduction-to-data-science-with-apache-spark/) for steps on manual Zeppelin setup

Author: [Ali Bajwa](https://www.linkedin.com/in/aliabajwa)

##### Contents:
  - [Setup Pre-requisites](https://github.com/hortonworks-gallery/ambari-zeppelin-service#setup-pre-requisites)
  - [Setup Ambari service](https://github.com/hortonworks-gallery/ambari-zeppelin-service#setup-the-ambari-service)
  - [Install Ambari view](https://github.com/hortonworks-gallery/ambari-zeppelin-service#install-zeppelin-view)
  - [Run demo zeppelin notebooks](https://github.com/hortonworks-gallery/ambari-zeppelin-service#use-zeppelin-notebook)
  - [Remove zeppelin service](https://github.com/hortonworks-gallery/ambari-zeppelin-service#remove-zeppelin-service)
  - [Deploy on clusters without internet access](https://github.com/hortonworks-gallery/ambari-zeppelin-service#deploy-on-clusters-without-internet-access)

##### Pre-requisites:
  - HDP 2.3 with at least HDFS, YARN, Zookeper, Spark and Hive installed. Instructions for older releases available [here](https://github.com/hortonworks-gallery/ambari-zeppelin-service/blob/master/README-22.md)
  - Have 2 ports available and open for zeppelin and its websocket. These will be defaulted to 9995/9996 (but can be configured in Ambari). If using sandbox on VirtualBox, you need to manually forward these.

##### Features:
  - Automates deployment, configuration, management of zeppelin on HDP cluster
  - Automates deployment of Ambari view to bring up Zeppelin webapp (requires manual ambari-server restart)
  - Runs zeppelin in yarn-client mode (instead of standalone) 
  - Runs zeppelin as configurable user, by default zeppelin (instead of root)
  - Uploads zeppelin jar to /apps/zeppelin location in HDFS to be accessible from all nodes in cluster
  - Exposes the [zeppelin-site.xml](https://github.com/apache/incubator-zeppelin/blob/master/conf/zeppelin-site.xml.template) and [zeppelin-env.sh](https://github.com/apache/incubator-zeppelin/blob/master/conf/zeppelin-env.sh.template) files in Ambari for easy configuration
  - Deploys sample notebooks (that demo hive, spark and sparksql, shell intepreters)
  - Configures Zeppelin to point to Hive metastore so Spark commands can access Hive tables out of the box
  - Spark, pyspark, sparksql, hive, shell all tested to be working
  - Offline mode: can manually copy tar to /tmp/zeppelin.tar.gz to allow service to be installed on clusters without internet access
  - Deploy using steps below or via [Ambari Store view](https://github.com/jpplayer/amstore-view)

##### Limitations:
  - Only tested on CentOS/RHEL 6 so far
  - Does not yet support install on secured (kerborized) clusters
  - On cloud envs, Zeppelin view will be setup using internal hostname, so you would need to have a corresponding hosts file entry on local machine
  - After install, Ambari thinks HDFS, YARN, Hive, HBase need restarting (seems like Ambari bug)
  - Current version of service does not support being installed via Blueprint
    
##### Testing:
  - These steps were tested on:
    - HDP 2.3 cluster installed via Ambari 2.1 with both Spark 1.4.1 and 1.3.1 on Centos 6
    - Latest HDP 2.3 sandbox using with both Spark 1.4.1 and 1.3.1 on Centos 6
  
##### Videos (from HDP 2.2.4.2):
  - [How to setup zeppelin service](https://www.dropbox.com/s/9s122qbjilw5d2u/zeppelin-1-setup.mp4?dl=0)
  - [How to setup zeppelin view and run sample notebooks](https://www.dropbox.com/s/skhudcy89s7qho1/zeppelin-2-view-demo.mp4?dl=0)

  
-------------------
  
#### Setup Pre-requisites:

- Download HDP 2.3 sandbox VM image (Sandbox_HDP_2.3_VMWare.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.3_VMWare.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```
- If you deployed in a VirtualBox Sandbox environment, enable port forwarding on ports 9995 and 9996. If you don't enable port 9996, the Zeppelin UI/Ambari View shows disconnected on the upper right and none of the default tutorials are visible. 

- Ensure Spark and Hive are installed. If not, use Add service wizard to install them

- (Optional) If you want to use Spark 1.4 instead of 1.3 (which comes with HDP 2.3), you can use below commands to download and set it up
```
sudo useradd zeppelin
sudo su zeppelin
cd /home/zeppelin
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.4.1-bin-hadoop2.6.tgz -O spark-1.4.1.tgz
tar -xzvf spark-1.4.1.tgz
export HDP_VER=`hdp-select status hadoop-client | sed 's/hadoop-client - \(.*\)/\1/'`
echo "spark.driver.extraJavaOptions -Dhdp.version=$HDP_VER" >> spark-1.4.1-bin-hadoop2.6/conf/spark-defaults.conf
echo "spark.yarn.am.extraJavaOptions -Dhdp.version=$HDP_VER" >> spark-1.4.1-bin-hadoop2.6/conf/spark-defaults.conf
exit
```


#### Setup the Ambari service

- To deploy the Zeppelin service, run below on ambari server
```
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
sudo git clone https://github.com/hortonworks-gallery/ambari-zeppelin-service.git   /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/ZEPPELIN   
```

- Restart Ambari
```
#on sandbox
service ambari restart

#on non-sandbox
sudo service ambari-server restart
```
- Once Ambari comes back up and the services turn green, you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check Zeppelin service -> Next -> Next -> Next -> Deploy. 
![Image](../master/screenshots/install-1.png?raw=true)
![Image](../master/screenshots/install-2.png?raw=true)
![Image](../master/screenshots/install-3.png?raw=true)

- This will bring up the Customize Services page where you can configure the Zeppelin service:
![Image](../master/screenshots/install-4.png?raw=true)

- There are three sections:
  - Advanced zeppelin-ambari-config: Parameters specific to Ambari service only (will not be written to zeppelin-site.xml or zeppelin-env.sh)
    - install dir: Local dir under which to install component
    - setup prebuilt: If true, will download previously built package (instead of building from source). To compile from source instead, set to false. If cluster does not have internet access, manually copy the tar.gz to /tmp/zeppelin.tar.gz on Ambari server and set this property to true. 
    - setup view: Whether the Zeppelin view should be compiled. Set to false if cluster does not have internet access
    - spark jar dir: Shared location where zeppelin spark jar will be copied to. Should be accesible by all cluster nodes
    - spark version: Version of Spark installed in location specified in SPARK_HOME. Default with HDP 2.3 is 1.3, but can also be set to 1.4 or 1.2 (if you manually installed Spark 1.2 or 1.4)
    - executor memory: Executor memory to use (e.g. 512m or 1g)
    
  - Advanced zeppelin-config: Used to populate [zeppelin-site.xml](https://github.com/apache/incubator-zeppelin/blob/master/conf/zeppelin-site.xml.template)
  - Advanced zeppelin-env: Used to populate [zeppelin-env.sh](https://github.com/apache/incubator-zeppelin/blob/master/conf/zeppelin-env.sh.template). See [Zeppelin docs](https://zeppelin.incubator.apache.org/docs/install/install.html) for more info
- (Optional) If you installed Spark 1.4, on the Customize services page:
  - Under 'Advanced zeppelin-config'
    - set `zeppelin.spark.version=1.4`
![Image](../master/screenshots/spark-1.4-config.png?raw=true)    
  - Under 'Advanced zeppelin-env'
    - set `export SPARK_HOME=/home/zeppelin/spark-1.4.1-bin-hadoop2.6/` 
![Image](../master/screenshots/spark-1.4-config2.png?raw=true)    

- Otherwise, to use Spark 1.3, you should not need to change any default configs. 
  - ...but here are sample of configurations that you could modify if needed (e.g. executor memory, port etc)
![Image](../master/screenshots/install-4.5.png?raw=true)
![Image](../master/screenshots/install-5.png?raw=true)
![Image](../master/screenshots/install-6.png?raw=true)
- Click Next to accept defaults...
![Image](../master/screenshots/install-7.png?raw=true)
- Click Deploy to start the installation

Note that:

- The default mode of the service sets up Zeppelin in yarn-client mode by downloading a tarball of precompiled bits (ETA: < 5min)

- (Optional) To instead pull/compile the latest Zeppelin code from the [git page](https://github.com/apache/incubator-zeppelin) (ETA: < 40min depending on internet connection):  
  - While adding zeppelin service, in the configuration step of the wizard:
    - set zeppelin.setup.prebuilt to false

- To track the progress of the install you can run the below:
```
tail -f  /var/log/zeppelin/zeppelin-setup.log
```

- On successful deployment you will see the Zeppelin service as part of Ambari stack and will be able to start/stop the service from here:
![Image](../master/screenshots/1.png?raw=true)

- You can see the parameters you configured under 'Configs' tab
![Image](../master/screenshots/2.png?raw=true)


#### Install Zeppelin view

- If Zeppelin was installed on the Ambari server host, simply restart Ambari server

- Otherwise copy the zeppelin view jar from `/home/zeppelin/zeppelin-view/target/zeppelin-view-1.0-SNAPSHOT.jar` on zeppelin node, to `/var/lib/ambari-server/resources/views/` dir on Ambari server node. Then restart Ambari server

- Now the Zeppelin view should appear under views: http://sandbox.hortonworks.com:8080/#/main/views

#### Use zeppelin notebook

- Lauch the notebook either via navigating to http://sandbox.hortonworks.com:9995 or via the view by opening http://sandbox.hortonworks.com:8080/#/main/views/ZEPPELIN/1.0.0/INSTANCE_1 should show Zeppelin as Ambari view
![Image](../master/screenshots/install-8.png?raw=true)

- There should be a few sample notebooks created. Select the Hive one (make sure Hive service is up first)
- On first launch of a notebook, you will the "Interpreter Binding" settings will be displayed. You will need to click "Save" under the interpreter order.
![Image](../master/screenshots/interpreter-binding.png?raw=true)    

- Now you will see the list of executable cells laid out in a sequence 
![Image](../master/screenshots/install-9.png?raw=true)

- Execute the cells one by one, by clicking the 'Play' (triangular) button on top right of each cell or just highlight a cell then press Shift-Enter

- Next try the same demo using the Spark/SparkSQL notebook (highlight a cell then press Shift-Enter):
![Image](../master/screenshots/install-10.png?raw=true)

  - The first invocation takes some time as the Spark context is launched. You can tail the interpreter log file to see the details.
```
 tail -f /var/log/zeppelin/zeppelin-interpreter-spark--*.log
```

- Other things to try
  - Test by creating a new note and enter some arithmetic in the first cell and press Shift-Enter to execute. 
```
2+2
```
  - Test pyspark by entering some python commands in the second cell and press Shift-Enter to execute. 
```
%pyspark
a=(1,2,3,4,5,6)
print a
```
  - Test settings by checking the spark version and spark home, python path env vars. 
```
sc.version
sc.getConf.get("spark.home")
System.getenv().get("PYTHONPATH")
System.getenv().get("SPARK_HOME")
``` 
 
  - If you are using Spark 1.4, `sc.version` should return `String = 1.4.0` and `SPARK_HOME` should be `/home/zeppelin/spark-1.4.1-bin-hadoop2.6/` (or whatever you set)
  - If you are using Spark 1.3, `sc.version` should return `String = 1.3.0` and `SPARK_HOME` should be `/usr/hdp/current/spark-client/` 
    
  - Test scala by pasting the below in the next cell to read/parse a log file from sandbox local disk
```
val words = sc.textFile("file:///var/log/ambari-agent/ambari-agent.log").flatMap(line => line.toLowerCase().split(" ")).map(word => (word, 1))
words.take(5)
```

  - You can also add a cell as below to read a file from HDFS instead. Prior to running the below cell, you should copy the log file to HDFS by running ```hadoop fs -put /var/log/ambari-agent/ambari-agent.log /tmp``` from your SSH terminal window
```
val words = sc.textFile("hdfs:///tmp/ambari-agent.log").flatMap(line => line.toLowerCase().split(" ")).map(word => (word, 1))
words.take(5)
```
![Image](../master/screenshots/3.png?raw=true)

- Now try a hive query and notice how you can view the results as a table and as different charts
```
%hive
select description, salary from default.sample_07
```

![Image](../master/screenshots/hive-queries.png?raw=true)

- Dependency loading/Form creation: see [Zeppelin docs](https://zeppelin.incubator.apache.org/docs/interpreter/spark.html#dependencyloading)
- Open the ResourceManager UI and notice Spark is running on YARN: http://sandbox.hortonworks.com:8088/cluster
![Image](../master/screenshots/RM-UI.png?raw=true)

You can also use this UI for troubleshooting hanging jobs

- Click on the ApplicationMaster link to access the Spark UI:

![Image](../master/screenshots/spark-UI.png?raw=true)


- One benefit to wrapping the component in Ambari service is that you can now monitor/manage this service remotely via REST API
```
export SERVICE=ZEPPELIN
export PASSWORD=admin
export AMBARI_HOST=sandbox.hortonworks.com
export CLUSTER=Sandbox

#get service status
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X GET http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#start service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#stop service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Stop $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
```
------------

#### Remove zeppelin service

- In case you need to remove the Zeppelin service: 

  - Stop the service and delete it. Then restart Ambari
  
```
export SERVICE=ZEPPELIN
export PASSWORD=admin
export AMBARI_HOST=sandbox.hortonworks.com
export CLUSTER=Sandbox    
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X DELETE http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#if above errors out, run below first
#curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Stop $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

service ambari-server restart
```
  - Remove artifacts 
  
```
rm -rf /opt/incubator-zeppelin
rm -rf /var/log/zeppelin*
rm -rf /var/run/zeppelin*
sudo -u hdfs hadoop fs -rmr /apps/zeppelin
rm -rf /var/lib/ambari-server/resources/views/zeppelin-view-1.0-SNAPSHOT.jar

VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
rm -rf /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/ZEPPELIN
service ambari-server restart
```

----------------

#### Deploy on clusters without internet access 

- Get appropriate zeppelin package copied to /tmp/zeppelin.tar.gz on Ambari server node
```
#package built for spark 1.2.1
PACKAGE=https://www.dropbox.com/s/nhv5j42qsybldh4/zeppelin-0.5.0-SNAPSHOT.tar.gz

#or package built for spark 1.3.1
#PACKAGE=https://www.dropbox.com/s/g9ua0no3gmb16uy/zeppelin-0.6.0-incubating-SNAPSHOT.tar.gz

#or package built for spark 1.4.1
#PACKAGE=https://www.dropbox.com/s/0qyvze6t3xhlthn/zeppelin-0.6.0-incubating-SNAPSHOT.tar.gz

wget $PACKAGE -O /tmp/zeppelin.tar.gz
```

- Get Zeppelin service folder copied to Ambari server dir on Ambari server node
```
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
wget https://github.com/hortonworks-gallery/ambari-zeppelin-service/archive/master.zip -O /tmp/ZEPPELIN.zip
unzip /tmp/ZEPPELIN.zip -d /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services
```

- Restart ambari: `service ambari-server restart`

- Go through 'Add service' wizard, same as above, making the below config changes:
  - Advanced zeppelin-ambari-config
    - zeppelin.setup.view = false (this ensures it does not try to build the view or download sample notebooks)
    - zeppelin.spark.version = 1.2 (or whatever is the version of package you downloaded)

  - Advanced zeppelin-env 
    - export SPARK_HOME=/your/spark/home (only needs to be changed if you installed your own spark version)
    
- Proceed with remaining screens and click Deploy

