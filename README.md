## Griffin [![Travic-CI](https://api.travis-ci.org/eBay/griffin.svg)](https://travis-ci.org/eBay/griffin)

Griffin is a Data Quality solution for distributed data systems at any scale in both streaming or batch data context. It provides a framework process for defining data quality model, executing data quality measurement, automating data profiling and validation, as well as a unified data quality visualization across multiple data systems. You can access our home page [here](https://ebay.github.io/griffin/).


### Contact us
[Google Groups](mailto://ebay-griffin-devs@googlegroups.com)


### CI
https://travis-ci.org/eBay/griffin

### Repository
Snapshot: https://oss.sonatype.org/content/repositories/snapshots

Release: https://oss.sonatype.org/service/local/staging/deploy/maven2

### How to build
1. git clone the repository of https://github.com/eBay/griffin
2. run "mvn install"

### How to run in docker
1. Install [docker](https://www.docker.com/).
2. Download our [docker](https://github.com/eBay/griffin/tree/master/docker) folder to your work path.
3. Enter docker directory and build images.  
    The first step is to build griffin-base-env, which prepares the environment for griffin.
    ```
    cd <your work path>/docker/griffin-base
    docker build -t griffin-base-env .
    ```
    The second step is to build griffin-env, which contains examples for griffin demo.
    ```
    cd <your work path>/docker/griffin
    docker build --no-cache -t griffin-env .
    ```

4. Run docker image griffin-env, then the backend is ready.
    ```
    docker run -it -h sandbox --name griffin -m 8G --memory-swap -1 \
    -p 2122:2122 -p 47077:7077 -p 46066:6066 -p 48088:8088 -p 48040:8040 \
    -p 48042:8042 -p 48080:8080 -p 47017:27017 griffin-env bash
    ```
    You can also drop the tail "bash" of the command above, then you will get tomcat service log printing in docker only.

5. Now you can visit UI through your browser, and follow the next steps on web UI [here](https://github.com/eBay/griffin/tree/master/griffin-doc/dockerUIguide.md#webui-test-case-guide).
    ```
    http://<your local IP address>:48080/
    ```  
    And you can also ssh to the docker container using account "griffin" with password "griffin".
    ```
    ssh griffin@<your local IP address> -p 2122
    ```

### How to deploy and run at local
1. Install jdk (1.7 or later versions)
2. Install Tomcat (7.0 or later versions)
3. Install MongoDB and import the collections
	```
	mongorestore /db:unitdb0 /dir:<dir of griffin-doc>/db/unitdb0
	```

4. Install [Hadoop](http://mirror.stjschools.org/public/apache/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz) (2.7 or later versions), you can get some help [here](https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/SingleCluster.html).  
Make sure you have the permission to use command "hadoop".   
Create an empty directory in hdfs as your hdfs path, and then create running and history directory in it
    ```
    hadoop fs -mkdir <your hdfs path>
    hadoop fs -mkdir <your hdfs path>/running
    hadoop fs -mkdir <your hdfs path>/history
    ```
5. Install [Spark](http://www.webhostingjams.com/mirror/apache/spark/spark-2.0.0/spark-2.0.0-bin-hadoop2.7.tgz) (version 2.0.0), if you want to install Pseudo Distributed/Single Node Cluster, you can get some help [here](http://why-not-learn-something.blogspot.com/2015/06/spark-installation-pseudo.html).  
Make sure you have the permission to use command "spark-shell".
6. Install [Hive](http://mirrors.koehn.com/apache/hive/hive-2.1.0/apache-hive-2.1.0-bin.tar.gz) (version 2.1.0), you can get some help [here](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHive).  
Make sure you have the permission to use command "hive".
7. Create a working directory, and it will be **your local path** now.
8. In your local path, put your data into Hive.  
First, you need to create some directories in hdfs
    ```
    hadoop fs -mkdir /tmp
    hadoop fs -mkdir /user/hive/warehouse
    hadoop fs -chmod g+w /tmp
    hadoop fs -chmod g+w /user/hive/warehouse
    ```
Then, run the following command in **your local path**
    ```
    schematool -dbType derby -initSchema
    ```
Now you can put your data into Hive by running "hive" here. You can get sample data [here](https://github.com/eBay/griffin/tree/master/griffin-doc/hive), then put into hive as following commands

    ```
    CREATE TABLE movie_source (
      movieid STRING,
      title STRING,
      genres STRING)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\;'
    STORED AS TEXTFILE;

    LOAD DATA LOCAL INPATH '<your data path>/MovieLensSample_Source.dat' OVERWRITE INTO TABLE movie_source;

    CREATE TABLE movie_target (
      movieid STRING,
      title STRING,
      genres STRING)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\;'
    STORED AS TEXTFILE;

    LOAD DATA LOCAL INPATH '<your data path>/MovieLensSample_Target.dat' OVERWRITE INTO TABLE movie_target;
    ```

    If you use hive command mode to input data, remember to create _SUCCESS file in hdfs table path as following

    ```
    hadoop fs -touchz /user/hive/warehouse/movie_source/_SUCCESS
    hadoop fs -touchz /user/hive/warehouse/movie_target/_SUCCESS
    ```
9. You can create your own model, build your jar file, and put it in **your local path**.  
(If you want to use our default models, please skip this step)
10. Currently we need to run the jobs automatically by script files, you need to set your own parameters in the script files and run it. You can edit the [demo script files](https://github.com/eBay/griffin/tree/master/griffin-doc/hive/script/) as following

    [env.sh](https://github.com/eBay/griffin/tree/master/griffin-doc/hive/script/env.sh)
    ```
    HDFS_WORKDIR=<your hdfs path>/running
    ```

    [griffin_jobs.sh](https://github.com/eBay/griffin/tree/master/griffin-doc/hive/script/griffin_jobs.sh)
    ```
    spark-submit --class com.ebay.griffin.Accu33 --master yarn --queue default --executor-memory 512m --num-executors 10 griffin-models-0.0.1-SNAPSHOT.jar  $lv1dir/cmd.txt $lv1dir/
    spark-submit --class com.ebay.griffin.Vali3 --master yarn --queue default --executor-memory 512m --num-executors 10 griffin-models-0.0.1-SNAPSHOT.jar  $lv1dir/cmd.txt $lv1dir/
    ```

    These commands submit the jobs to spark, if you want to try your own model or modify some parameters, please edit it.
    If you want to use your own model, change "griffin-models-0.0.1-SNAPSHOT.jar" to "your path/your model.jar", and change the class name.  

    Put these script files in **your local path**, run griffin_regular_run.sh as following
    ```
    nohup ./griffin_regular_run.sh &
    ```

11. Open [application.properties](https://github.com/eBay/griffin/tree/master/griffin-core/src/main/resources/application.properties) file, read the comments and specify the properties correctly. Or you can edit it as following
    ```
    env=prod
    job.local.folder=<your local path>/tmp
    job.hdfs.folder=<your hdfs path>
    job.hdfs.runningfoldername=running
    job.hdfs.historyfoldername=history
    ```
    If you set the properties as above, you need to make sure the directory "tmp" exists in your local path
12. Build the whole project and deploy griffin-core/target/ROOT.war to tomcat
    ```
    mvn install -DskipTests
    ```
13. Then you can review the RESTful APIs through http://localhost:8080/api/v1/application.wadl

### How to develop
In dev environment, you can run backend REST service and frontend UI seperately. The majority of the backend code logics are in the [griffin-core](https://github.com/eBay/griffin/tree/master/griffin-core) project. So, to start backend, please import maven project Griffin into eclipse, right click ***griffin-core->Run As->Run On Server***

To start frontend, please follow up the below steps.

1. Open **griffin-ui/js/services/services.js** file

2. Specify **BACKEND_SERVER** to your real backend server address, below is an example

    ```
    var BACKEND_SERVER = 'http://localhost:8080'; //dev env
    //var BACKEND_SERVER = 'http://localhost:8080/ROOT'; //dev env
    ```

3. Open a command line, run the below commands in root directory of **griffin-ui**

   - npm install
   - bower install
   - npm start

4. Then the UI will be opened in browser automatically, please follow the [User Guide](https://github.com/eBay/griffin/tree/master/griffin-doc/userguide.md), enjoy your journey!

**Note**: The front-end UI is still under development, you can only access some basic features currently.


### Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to contribute code, documentation, etc.
