# ClouderaAdmin

# Erro comum ao instalar o CM
    Não ter o Oracle JDK instalado nas máquinas (Java 7) para o CDH 5, para o CDH 6 Java 8 por Default.
    Instalar o pacote Python, importante para o funcionamento do HUE
    
# CM User Config Conventional mode
    Cloudera Manager Agent runs as the root OS user
    Single user mode
      ─ Available beginning with Cloudera Manager 5.3 and CDH 5.2
      ─ Cloudera Manager Agent, and all processes run by services managed by
    Cloudera Manager, are started as a single configured user and group
      ─ Default user: cloudera-scm
      ─ Prioritizes isolation between Hadoop and the rest of the system
      ─ De-prioritizes isolation between Hadoop process running on the system
      ─ Limitations and manual configurations apply—see http://tiny.cloudera.com/sum for details
    The choice made applies to all clusters managed by the Cloudera Manager deployment

# View logs CM:
    On the machine where Cloudera Manager Server is installed
    ─ /var/log/cloudera-scm-server/cloudera-scm-server.log
# View logs agetnt CM (datanode):
    On all machines in the cluster
    ─ /var/log/cloudera-scm-agent/cloudera-scm-agent.log

# Cloudera Manager CDH installation wizard will suggest specific Hadoop daemons be installed to specific cluster nodes
    ─ Based on available resources
    ─ It is easy to override suggestions
# Not all daemons run on each machine
    ─ NameNode, ResourceManager, JobHistoryServer (“master” daemons)
    ─ One per cluster, unless running in an HA configuration
    ─ Secondary NameNode
    ─ One per cluster in a non-HA environment
    ─ DataNodes, NodeManagers
    ─ On each data node in the cluster
# Exception: for small clusters (less than 10 – 20 nodes), it is acceptable for more than one of the master daemons to run on the same physical node

# Install Cloudera Manager Server
    sudo yum install -y cloudera-manager-daemons \cloudera-manager-server
    Note: The -y option provides an answer of yes in response to an expected confirmation prompt.
    
    Set the Cloudera Manager Server service to not start on boot.
    sudo chkconfig cloudera-scm-server off
    
    Run the script to prepare the Cloudera Manager database.
    sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql cmserver cmserveruser password
    After running the command above you should see the message, “All done, your SCM database is configured correctly!”
    
    Verify the local software repositories.
    curl lion:8000
  
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
        <title>Directory listing for /</title>
        <body>
        <h2>Directory listing for /</h2>
        <hr>
        <ul>
        <li><a href="CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel">CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel</a>
        <li><a href="CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha">CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha</a>
        <li><a href="CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha1">CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha1</a>
        <li><a href="manifest.json">manifest.json</a>
        </ul>
        <hr>
        </body>
        </html>

    $ curl lion:8050/RPMS/x86_64/
    
        training@lion:~$ curl lion:8050/RPMS/x86_64/
        <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
        <title>Directory listing for /RPMS/x86_64/</title>
        <body>
        <h2>Directory listing for /RPMS/x86_64/</h2>
        <hr>
        <ul>
        <li><a href="cloudera-manager-agent-5.9.0-1.cm590.p0.249.el6.x86_64.rpm">cloudera-manager-agent-5.9.0-1.cm590.p0.249.el6.x86_64.rpm</a>
        <li><a href="cloudera-manager-daemons-5.9.0-1.cm590.p0.249.el6.x86_64.rpm">cloudera-manager-daemons-5.9.0-1.cm590.p0.249.el6.x86_64.rpm</a>
        <li><a href="cloudera-manager-server-5.9.0-1.cm590.p0.249.el6.x86_64.rpm">cloudera-manager-server-5.9.0-1.cm590.p0.249.el6.x86_64.rpm</a>
        <li><a href="cloudera-manager-server-db-2-5.9.0-1.cm590.p0.249.el6.x86_64.rpm">cloudera-manager-server-db-2-5.9.0-1.cm590.p0.249.el6.x86_64.rpm</a>
        <li><a href="enterprise-debuginfo-5.9.0-1.cm590.p0.249.el6.x86_64.rpm">enterprise-debuginfo-5.9.0-1.cm590.p0.249.el6.x86_64.rpm</a>
        </ul>
        <hr>
        </body>
        </html>
        training@lion:~$ 

    Start the Cloudera Manager Server.
    sudo service cloudera-scm-server start
    
    training@lion:~$ sudo service cloudera-scm-server start
    Starting cloudera-scm-server:                              [  OK  ]


    training@lion:~$ ps wax | grep cloudera-scm-server
    11191 ?        Ssl    1:59 /usr/java/jdk1.7.0_67/bin/java -cp .:lib/*:/usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar -server -Dlog4j.configuration=file:/etc/cloudera-scm-server/log4j.properties -Dfile.encoding=UTF-8 -Dcmf.root.logger=INFO,LOGFILE -Dcmf.log.dir=/var/log/cloudera-scm-server -Dcmf.log.file=cloudera-scm-server.log -Dcmf.jetty.threshhold=WARN -Dcmf.schema.dir=/usr/share/cmf/schema -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dpython.home=/usr/share/cmf/python -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+HeapDumpOnOutOfMemoryError -Xmx2G -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp -XX:OnOutOfMemoryError=kill -9 %p com.cloudera.server.cmf.Main
    11631 pts/0    S+     0:00 grep cloudera-scm-server
    
        The results of the ps command above show that Cloudera Manager Server is using the JDBC MySQL connector to connect to MySQL. It also shows logging configuration and other details.
    
    Review the Cloudera Manager Server log file.
    sudo less /var/log/cloudera-scm-server/cloudera-scm-server.log
    
    018-11-27 10:38:14,469 INFO main:org.mortbay.log: Logging to org.slf4j.impl.Log4jLoggerAdapter(org.mortbay.log) via org.mortbay.log.Slf4jLog
    2018-11-27 10:38:14,470 INFO main:org.mortbay.log: Registered SubjectType HOST
    2018-11-27 10:38:14,470 INFO main:com.cloudera.cmon.TimeSeriesAttribute: Registered TimeSeriesAttribute hostId as HOSTID
    2018-11-27 10:38:14,470 INFO main:com.cloudera.cmon.TimeSeriesAttribute: Registered TimeSeriesAttribute hostname as HOSTNAME
    2018-11-27 10:38:14,470 INFO main:com.cloudera.cmon.TimeSeriesEntityType: Registered TimeSeriesEntityType HOST
    2018-11-27 10:38:14,470 INFO main:com.cloudera.cmon.TimeSeriesEntityType: Registered TimeSeriesEntityType ACTIVITY

# Configuring HDFS Read Caching
    ▪ Cloudera Manager enables HDFS Read Caching by default
    ▪ Amount of RAM per DataNode to use for caching is controlled by dfs.datanode.max.locked.memory
        ─ Can be configured per role group
        ─ Default is 4GB
        ─ Set to 0 to disable caching
        
# Anatomy of a File Write (2)
    1. Client connects to the NameNode
    2. NameNode places an entry for the file in its metadata, returns the block name
    and list of DataNodes to the client
    3. Client connects to the first DataNode and starts sending data
    4. As data is received by the first DataNode, it connects to the second and starts
    sending data
    5. Second DataNode similarly connects to the third
    6. Acknowledgement (ACK) packets from the pipeline are sent back to the client
    7. Client reports to the NameNode when the block is written        

# Anatomy of a File Write (3)
    ▪ If a DataNode in the pipeline fails
        ─ The pipeline is closed
        ─ A new pipeline is opened with the two good nodes
        ─ The data continues to be written to the two good nodes in the pipeline
        ─ The NameNode will realize that the block is under-replicated, and will re-replicate it to another DataNode

    ▪ As blocks of data are written, the client calculates a checksum for each block
        ─ Sent to the DataNode along with the data
        ─ Written together with each data block
        ─ Used to ensure the integrity of the data when it is later read
        
# Hadoop is “Rack-aware”
    ▪ Hadoop understands the concept of “rack awareness”
        ─ The idea of where nodes are located, relative to one another
        ─ Helps the ResourceManager allocate processing resources on nodes closest to the data
        ─ Helps the NameNode determine the “closest” block to a client during reads
        ─ In reality, this should perhaps be described as being “switch-aware”
    ▪ HDFS replicates data blocks on nodes on different racks
        ─ Provides extra data security in case of catastrophic hardware failure
    ▪ Rack-awareness is determined by a user-defined script
        ─ Rack Topology configuration details through Cloudera Manager discussed later in this course

# Anatomy of a File Read (2)
    1. Client connects to the NameNode
    2. NameNode returns the name and locations of the first few blocks of the file
        ─ Block locations are returned closest first
    3. Client connects to the first of the DataNodes, and reads the block
        ─ If the DataNode fails during the read, the client will seamlessly connect to the next one in the list to read the block
        
# HDFS File Permissions
    ▪ Files in HDFS have an owner, a group, and permissions
        ─ Very similar to Unix file permissions
    ▪ File permissions are read (r), write (w), and execute (x) for each of owner, group, and other
        ─ x is ignored for files
        ─ For directories, x means that its children can be accessed
    ▪ HDFS permissions are designed to stop good people doing foolish things
        ─ Not to stop bad people doing bad things!
        ─ HDFS believes you are who you tell it you are
        
# Hadoop Security Overview
    ▪ Authentication
        ─ Proving that a user or system is who he or she claims to be
        ─ Hadoop can provide strong authentication control via Kerberos
        ─ Cloudera Manager simplifies Kerberos deployment
        ─ Authentication via LDAP is available with Cloudera Enterprise
    ▪ Authorization (access control)
        ─ Allowing people or systems to do some things but not other things
        ─ CDH has traditional POSIX-style permissions for files and directories
        ─ Access Control Lists (ACLs) for HDFS
        ─ Role-based access control provided with Apache Sentry
    ▪ Data encryption
        ─ OS filesystem-level, HDFS-level, and Network-level options

# More hdfs dfs Command Examples
    ▪ Copy file input.txt from local disk to the user’s directory in HDFS
        $ hdfs dfs -put input.txt input.txt
        ─ This will copy the file to /user/username/input.txt
    
    ▪ Get a directory listing of the HDFS root directory
        $ hdfs dfs -ls /
    ▪ Delete the file /reports/sales.txt
        $ hdfs dfs –rm /reports/sales.txt
        
# Apache HBase
    ▪ HBase: a highly-scalable, distributed column-oriented database
        ─ An open-source Apache project, based on Google’s BigTable
        ─ Google’s example use case is “search the entire Internet”
        ─ Massive amounts of data in a table
        ─ Very high throughput to handle a massive user base
    ▪ A complex client of HDFS
        ─ All HBase files are stored in HDFS
        ─ HBase adds a NoSQL key-value retrieval system on top of HDFS
    ▪ HBase from the Hadoop Administrator’s perspective
        ─ An automated client of HDFS trying to present the appearance of a random store by continuously organizing and reorganizing its blocks
        
# Characteristics of HBase
    ▪ Scalable
        ─ Easily handles massive tables
        ─ Handles high throughput
        ─ Sparse rows where number of columns varies
    ▪ Fault tolerant
        ─ Derives some of it’s fault tolerance from HDFS copying of blocks
    ▪ Common use cases for HBase
        ─ Random write, random read, or both (but not neither)
        ─ No need for HBase if neither random read nor write is needed
        ─ Thousands of operations per second on multiple terabytes of data
        ─ Access patterns are well-known and relatively simple
        
# HBase Data Locality Considerations
    ▪ Data locality means moving the computation close to the data
        ─ Some interactions between HBase and HDFS can risk data locality
        ─ If the data locality is lost, data is copied over the network leading to poor performance
    ▪ HBase and HDFS interaction
        ─ The HBase RegionServer writes data to HDFS on its local disk
        ─ HDFS replicates the data to other nodes in the cluster
        ─ Replication ensures the data remains available even if a node fails
    ▪ Risks to data locality
        ─ HBase balancer moves a region to balance data sizes across RegionServers
        ─ A RegionServer dies and all it’s regions are relocated
        ─ The cluster is stopped and restarted
        ─ A table is disabled and re-enabled
        
# General Advice on HBase and HDFS
    ▪ Start the HDFS service before starting the HBase service
    ▪ Consider how to balance blocks so data locality isn’t lost
    ▪ Advanced configuration parameters available
        ─ For example, for write-heavy workloads
    ▪ Cloudera offers a full course on HBase
        
# Apache Kudu
    ▪ Kudu is a storage engine for structured data
        ─ Designed for specific use cases that require fast analytics on rapidly changing
    data
    ▪ Provides a data service with benefits of both HDFS and HBase
        ─ Simplify applications that use hybrid HDFS/HBase
        ─ Accesses data through the local file system—not an HDFS client
    ▪ Characteristics of Kudu
        ─ Performs well for both scan and random access
        ─ High CPU efficiency
        ─ High IO efficiency
        ─ Can update data in place
# Voltar        
# Add Directories and Files to HDFS

    As the hdfs user, create a home directory for the training user on HDFS and give the training user ownership of it.
        training@elephant:~$ sudo -u hdfs hdfs dfs -mkdir -p /user/training/
        training@elephant:~$ sudo -u hdfs hdfs dfs -chown training -p /user/training/
        training@elephant:~$ sudo -u hdfs hdfs dfs -chown training /user/training/
        training@elephant:~$ hdfs dfs -ls /user/
        Found 2 items
        drwxrwxrwx   - mapred   hadoop              0 2018-11-28 05:27 /user/history
        drwxr-xr-x   - training supergroup          0 2018-11-28 08:46 /user/training
        training@elephant:~$ 

    Create an HDFS directory called /user/training/weblog, in which you will store the access log.
    hdfs dfs -mkdir weblog
    
    Extract the access_log.gz file and upload it to HDFS.
    cd ~/training_materials/admin/data
    $ gunzip access_log.gz
    $ hdfs dfs -put access_log weblog
    
    Run the hdfs dfs -ls command to review the file’s permissions in HDFS.
    
    
# Hadoop Computational Frameworks
    ▪ HDFS provides scalable storage in your Hadoop cluster
    ▪ Computational frameworks provide the distributed computing
        ─ Batch processing
        ─ SQL queries
        ─ Search
        ─ Machine learning
        ─ Stream processing
    ▪ Computational frameworks compete for resources
    ▪ YARN provides resource management for computational frameworks that support it
        ─ Examples discussed in this chapter:
        ─ MapReduce
        ─ Apache Spark

# Notable Computational Frameworks on YARN
    ▪ MapReduce
        ─ The original framework for writing Hadoop applications
        ─ Proven, widely used
        ─ Sqoop, Pig, and other tools use MapReduce to interact with HDFS
    ▪ Spark
        ─ A programming framework for writing Hadoop applications
        ─ A fast, general, big data processing framework
        ─ Built-in modules for streaming, SQL, machine learning, and graph processing
        ─ Supports processing of streaming data
        ─ Faster than MapReduce
    ▪ Hive can run on Spark or MapReduce
    
# What Is MapReduce?
    ▪ MapReduce is a programming model
        ─ Neither platform- nor language-specific
        ─ Record-oriented data processing (key and value)
        ─ Facilitates task distribution across multiple nodes
    ▪ Where possible, each node processes data stored on that node
    ▪ Consists of two developer-created phases
        ─ Map
        ─ Reduce
    ▪ In between Map and Reduce is the shuffle and sort
        ─ Sends data from the Mappers to the Reducers
        
# What is Apache Spark?
    ▪ A fast, general engine for large-scale data processing
    on a cluster
        ─ One of the fastest-growing Apache projects
        ─ Includes map and reduce as well as non-batch processing models
    ▪ High-level programming framework
        ─ Programmers can focus on logic not plumbing
        ─ Works directly with HDFS
        ─ Near real-time processing
        ─ Configurable in-memory data caching for efficient iteration
    ▪ Application processing is distributed across worker nodes
        ─ Distributed storage, horizontally scalable, fault tolerance

# Spark RDDs
    ▪ RDD: the fundamental unit of data in Spark
        ─ Resilient: if data in memory is lost, it can be recomputed
        ─ Distributed: stored in memory across the cluster
        ─ Dataset: initial data comes from a file or is created programmatically
    ▪ RDDs are immmutable
    ▪ RDDs are created from file(s), data in memory or another RDD
    ▪ Most Spark programming consists of performing operations on RDDs
        ─ Two types of operations
        ─ Transformations: create a new RDD based on an existing one
        ─ Actions: return a value from an RDD

# Spark Executors
    ▪ A Spark Application consists of one or more Spark jobs
        ─ Jobs run tasks in executors and executors run in YARN containers (one executor per container)
        ─ Executors exist for the life of the application
        ─ If spark.dynamicAllocation.enabled is set to true (default),the number of executors for an application can scale based on workload after it starts
        ─ YARN does not yet support already allocated container resizing
        ─ An executor can run tasks concurrently on in-memory data

# YARN Daemons
    ▪ ResourceManager: one per cluster
        ─ Initiates application startup
        ─ Schedules resource usage on worker nodes
    ▪ JobHistoryServer: one per cluster
        ─ Archives MapReduce jobs’ metrics and metadata
    ▪ NodeManager: one per worker node
        ─ Starts application processes
        ─ Manages resources on worker nodes

# YARN ResourceManager—Key Points
    ▪ What the ResourceManager does:
        ─ Manages nodes
        ─ Tracks heartbeats from NodeManagers
        ─ Runs a scheduler
        ─ Determines how resources are allocated
        ─ Manages containers
        ─ Handles ApplicationMasters requests for resources
        ─ Releases containers when they expire or when the application completes
        ─ Manages ApplicationMasters
        ─ Creates a container for ApplicationMasters and tracks heartbeats
        ─ Manages cluster-level security

# Running an Application in YARN
    ▪ Containers
        ─ Allocated by the ResourceManager
        ─ Require a certain amount of resources (memory, CPU) on a worker node
        ─ YARN Applications run in one or more containers
    ▪ ApplicationMaster
        ─ One per YARN application execution
        ─ Runs in a container
        ─ Framework/application specific
        ─ Communicates with the ResourceManager scheduler to request containers to run application tasks
        ─ Ensures NodeManager(s) complete tasks
        
# Summary: Cluster Resource Allocation
    ▪ ResourceManager (master)
        ─ Grants containers
        ─ Performs cluster scheduling
    ▪ ApplicationMaster (runs within a container)
        ─ Negotiates with the ResourceManager to obtain containers on behalf of the application
        ─ Presents containers to Node Managers
    ▪ Node Managers (workers)
        ─ Manage life-cycle of containers
        ─ Launch Map and Reduce tasks or Spark executors in containers
        ─ Monitor resource consumption
        
# Summary: Requesting Resources
    ▪ A resource request is a fine-grained request for memory and CPU sent to the ResourceManager to be fulfilled
    ▪ If the request is successful, a container is granted
    ▪ A resource request is composed of several fields that specify
        ─ The amount of a given resource required
        ─ Data locality information, that is, the preferred node or rack on which to run
        Field Name Sample Value
        priority        integer
        capability      <2 gb, 1 vcore>
        resourceName    host22, Rack5, *
        numContainers   integer
        
# Discovering YARN Application Details in the Shell
    ▪ Displaying YARN application details in the shell
        ─ To view all applications currently running on the cluster
        ─ yarn application -list
        ─ Returns all running applications, including the application ID for each
        ─ To view all applications on the cluster, including completed applications
        ─ yarn application -list –appStates all
        ─ To display the status of an individual application
        ─ yarn application -status <application_ID>

# YARN Application States and Types
    ▪ Some logs and command results list YARN Application States
    ▪ Operating states: SUBMITTED, ACCEPTED, RUNNING
    ▪ Initiating states: NEW, NEW_SAVING
    ▪ Completion states: FINISHED, FAILED, KILLED
    ▪ Some logs and command results list YARN Application Types: MAPREDUCE, YARN, OTHER
    
# Killing YARN Applications
    ▪ To kill a running YARN application from Cloudera Manager
        ─ Use the drop-down menu for the application in the YARN Applications page for the cluster
    ▪ To kill a running YARN application from the command line
        ─ You can not kill it just by hitting Ctrl+C in the terminal
        ─ This only kills the client that launched the application
        ─ The application is still running on the cluster!
        ─ Utilize the YARN command line utility instead
        ─ yarn application –kill <application_ID>

# YARN High Availability
    ▪ ResourceManager High Availablity removes a single point of failure
        ─ Active-standby ResourceManager pair
        ─ Automatic failover option
    ▪ Protects against significant performance effects on running applications
        ─ Machine crashes
        ─ Planned maintenance events on the ResourceManager host machine
    ▪ If failover to the standby occurs
        ─ In-flight YARN applications resume from the last saved state store
    ▪ To enable
        ─ From the Cloudera Manager YARN page, select Actions > Enable High Availability, and select the host for the standby ResourceManager
        
# YARN Application Log Aggregation

https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YarnCommands.html
    
    yarn application -list -appStates ALL
    yarn application -list -appStates FINISHED
    yarn application -list -appStates RUNNING
    yarn application -list -appStates FAILED
    
    yarn logs -applicationId <appId> | less
    
    ▪ Debugging distributed processes is intrinsically difficult
        ─ Cloudera Manager’s log aggregation improves this situation
    ▪ YARN provides application log aggregation services
        ─ Recommended (and enabled by default by Cloudera Manager)
        ─ Logs are aggregated by application
        ─ Access the logs from Cloudera Manager, any HDFS client, or the shell
    ▪ When YARN log aggregation is enabled:
        ─ Container log files are moved from NodeManager hosts’ directories to HDFS when the application completes
        ─ NodeManager directory: /var/log/hadoop-yarn/container
        ─ Default HDFS directory: /var/log/hadoop-hdfs
    ▪ Aggregated YARN application log retention
        ─ Apache default: log retention disabled (and infinite when enabled)
        ─ Cloudera Manager default: enabled and 7 day retention

# Accessing YARN Application Logs in Cloudera Manager
    ▪ Access all application logs in Cloudera Manager
        ─ From the YARN Applications page, choose Collect Diagnostics Data
        ─ Option provided to send Diagnostic Data to Cloudera Support
        ─ Option to Download Result Data, and view recent or full logs
        
# MapReduce Log Details
    ▪ MapReduce jobs produce the following logs:
        ─ ApplicationMaster log
        ─ stdout, stderr, and syslog output for each Map and Reduce task
        ─ Job configuration settings specified by the developer
        ─ Counters
    ▪ MapReduce Job History Server log directory
        ─ Default: /var/log/hadoop-mapreduce
        
# Spark Log Details
    ▪ Spark can write application history logs to HDFS
        ─ Default: enabled
    ▪ Spark application history location
        ─ Default location in HDFS: /user/spark/applicationHistory
        ─ Find and view the logs from the command line:
            $ sudo –u hdfs hdfs dfs –ls /user/spark/applicationHistory
            $ sudo –u hdfs hdfs dfs –cat /user/spark/applicationHistory/\application_<application_id>
    ▪ Spark History Server log directory
        ─ Default: /var/log/spark

# Accessing YARN Application Logs From the Command Line
    ▪ Accessing application logs from the command line allows you to use utilities like grep for quick log analysis
    ▪ First find the application ID with yarn application –list
    ▪ Then, once you have the application ID, you can list its logs
    
# Cloudera Manager—Configuration Terminology 
    ▪ Services: A Hadoop cluster has Services, such as HDFS or YARN 
    ▪ Roles: The individual components of a service. For example, the HDFS service has the NameNode and DataNode roles 
    ▪ Role Group: A way of creating logical groups of daemons across multiple machines 
    ▪ Role Instance: A member of a Role Group. For example, a DataNode installed on a specific machine is a role instance

# Configuration Settings—Inheritance
    ▪ Configuration settings: order of inheritance
        ─ Service > Role Group > Role Instance
    ▪ A setting at the Role Group level will override the Service level setting
        ─ Example: Assume some DataNode hosts in the cluster have more HDFS storage capacity than other DataNode hosts in the cluster
        ─ Configure a DataNode Role Group for the larger hosts with storage settings that override the lower default service-level storage capacity settings
        ─ Add the larger hosts’ DataNode roles to the Role Group
    ▪ A setting at the Role Instance level will override the Role Group level and Service level settings
        ─ Example: Temporarily enable debug logging for a role instance while troubleshooting a potential hardware problem
        
# Service Configuration Settings—Deployment
    ▪ Key point: each daemon configuration is given its own execution and configuration environment
        ─ This is very different than in non-Cloudera Manager managed clusters
    ▪ Configuration files for daemons that Cloudera Manager manages are deployed on any host that has a role for the service
    ▪ The Cloudera Manager Agent pulls the daemon configuration and places it in the appropriate configuration directory
        ─ Includes arguments to exec(), config files, directories to create, and other information (such as cgroup settings)
    ▪ Default location for Hadoop daemon configurations
        ─ A subdirectory of: /var/run/cloudera-scm-agent/process/
        
# Client Configuration Files—Deployment
    ▪ Cloudera Manager stores client configurations separately from service configurations
        ─ Default location: /etc/hadoop/conf
    ▪ Cloudera Manager creates a “client configuration file” zip archive of the configuration files containing service properties
        ─ Each archive has the configuration files needed to access the service
        ─ Example: a MapReduce client configuration file contains copies of core-site.xml, hadoop-env.sh, hdfs-site.xml, log4j.properties and mapred-site.xml
    ▪ Client configuration files are generated by Cloudera Manager when a cluster is created, or a service or a gateway role is added on a host
        ─ A “gateway” role has client configurations, but no daemons
        ─ Cloudera Manager indicates when Client Configuration Redeployment is needed
        ─ There is also an option to download and deploy manually
        
 # Cloudera Manager decouples server and client configurations
    ─ Server settings (NameNode,DataNode) are stored in /var/run/clouderascm-agent/process subdirectories
    ─ Client settings are stored in /etc/hadoop subdirectories
    
    
# (IMPORTANTE PARA OS MASTERS ) HDFS NameNode Metadata Location (1)
    * PARA OS MASTERS É IMPORTANTE TEM RAID;
    ▪ The single most important configuration value on your entire cluster, used by the NameNode: dfs.namenode.name.dir Set in HDFS / NameNode Group
        ─ Location on the local filesystem where the NameNode stores its name table (fsimage) metadata
        ─ Multiple entries in Cloudera Manager
        ─ Default is file://${hadoop.tmp.dir}/dfs/name
    ▪ Loss of a NameNode’s metadata will result in the effective loss of all the data in its namespace
        ─ Although the blocks will remain, there is no way of reconstructing the original files without the metadata  
        
# HDFS NameNode Metadata Location (2)
    ▪ For all HDFS deployments, dfs.namenode.name.dir must specify at least two disks (or a RAID volume) on the NameNode
        ─ Failure to set correctly will result in eventual loss of your cluster’s data
    ▪ By default, a NameNode will write to the edit log files in all directories in dfs.namenode.name.dir synchronously
        ─ For non-HA HDFS deployments, can explicitly specify the path to the edits log directory by setting dfs.namenode.edits.dir
        ─ High-Availability HDFS settings are discussed in a later chapter
    ▪ If a directory in the list disappears, the NameNode will continue to function
        ─ It will ignore that directory until it is restarted

# HDFS NameNode Metadata Location (3)
    ▪ For non-HA HDFS deployments, dfs.namenode.name.dir must additionally specify an NFS mount elsewhere on the network
        ─ If you do not do this, catastrophic failure of the NameNode will result in the loss of the metadata
    ▪ Recommendation for the NFS mount point tcp,soft,intr,timeo=10,retrans=10
        ─ Soft mount so the NameNode will not hang if the mount point disappears
        ─ Will retry transactions 10 times, at 1-10 second intervals, before being deemed to have failed

# dfs.datanode.du.reserved
    Set in HDFS / DataNode Group
        ─ The amount of space on each disk in dfs.datanode.data.dir which cannot be used for
    HDFS block storage
        ─ Reserve space typically used for mapper and reducer intermediate data
        ─ Default: 10GB
        ─ Recommended: 25% of the space on each volume, with a minimum of 10GB
# dfs.datanode.data.dir
    Set in HDFS / DataNode Group
        ─ Location on the local filesystem where a DataNode stores its blocks
        ─ Can be a list of directories; round-robin writes to the directories (no redundancy)
        ─ Can be different on each DataNode
# dfs.blocksize
    Set in HDFS / Service-Wide
        ─ The block size for new files, in bytes
        ─ Default and recommended value is 134217728 (128MB)
        ─ Note: this is a client-side setting
# dfs.replication
    Set in HDFS / Service-Wide / Replication
        ─ The number of times each block should be replicated when a file is written
        ─ Default and recommended value is 3
        ─ Note: this is a client-side setting

# Use Trash
    Set in HDFS / Gateway Group
        ─ When a file is deleted using the Hadoop shell, it is moved to the .Trash directory in the user’s home instead of being immediately deleted
        ─ Supports data recovery
        ─ Default in Cloudera Manager: true
        ─ Recommendation: true
        ─ If fs.trash.interval is enabled, then this setting is ignored
        ─ To recover a file, move it to a location outside the .Trash subdirectory
# fs.trash.interval
    Set in HDFS / NameNode Group
        ─ Specifies the minimum number of minutes files will be kept in the trash
        ─ CM Default : 1 day (1440 minutes)
        ─ Setting this to 0 disables the trash feature

# HDFS Memory Settings
    ▪ Ideal heap size will vary by use case
    ▪ Recommended to reserve 3GB of available memory for the OS
    ▪ NameNode: 1GB per million blocks (recommended minimum 4GB)
    ▪ DataNode: recommend 1GB min, 4GB max
    
# All logs have these threshold options:
    Recomendado  para produção deixar nível baixo de LOG - ERROR. O Dev consegue setar o nível de log em tempo de execução (parâmetro de script).
    Níves:
      - TRACE, DEBUG, INFO, WARN, ERROR, FATAL
    ─ INFO is the default
   
# Every daemon writes to two files
    ─ (STD OUT) .out file: combination of stdout and stderr during daemon startup
    ─ (ERROR) .log file: first port of call when diagnosing problems

#  Persistent log level for each daemon configured in Cloudera Manager
    ─ In the Configuration area for each service, search “logging threshold”

#  Logs are consolidated and searchable using Cloudera Manager
    ─ Choose Diagnostics > Logs
    ─ Set filters described below, enter search term(s), and click Search    
# Search results include all log entries that match the filters applied
        ─ Every log entry presented includes a View Log File link which opens the Log
    Details page
        ▪ Log Details page shows a portion of the full log
        ─ Download Full Log option available
# yarn.log-aggregation-enable
    Set in YARN / Service-Wide
        ─ Enable log aggregation. Default: true. Recommendation: true.
        ─ Used by the NodeManagers
# yarn.nodemanager.remote-app-log-dir
    Set in YARN / NodeManager Group
        ─ HDFS directory to which application log files are aggregated
        ─ Default: /tmp/logs. Example: /var/log/hadoop-yarn/apps
        ─ Used by NodeManagers and the JobHistoryServer
# yarn.nodemanager.log-dirs
    Set in YARN / NodeManager Group
        ─ Local directories to which application log files are written
        ─ If log is aggregation enabled, these files are deleted after an application completes
        ─ Default: /var/log/hadoop-yarn/container
        ─ Used by NodeManagers

# yarn.nodemanager.local-dirs
    Set in YARN / NodeManager Group
        ─ Directories in which local resource files are stored
        ─ In MapReduce, intermediate data files generated by the Mappers and used by the Reducers are local resource files
        ─ Spark applications also make use of local resource files
        ─ If these directories are not big enough, then jobs (applications) will fail
        ─ Example: /disk1/yarn/nm, /disk2/yarn/nm
        ─ Used by NodeManagers

# Exploring Hadoop Configuration Settings
     cd /var/run/cloudera-scm-agent/process
     $ sudo tree
       
     . Compare the first 20 lines of the NameNode’s copy of hdfs-site.xml to the NodeManager’s copy of the same file
     $ sudo ls | grep NODE
     $ sudo head -20 nn-hdfs-NAMENODE/core-site.xml
     $ sudo head -20 nn-yarn-NODEMANAGER/core-site.xml
     
     Return to the elephant terminal window and list the contents of the /var/run/cloudera-scm-agent/process directory.
     $ sudo ls -l /var/run/cloudera-scm-agent/process
     
     Find the difference between the old NameNode core-site.xml file and the new one
      sudo diff -C 2 nn-hdfs-NAMENODE/core-site.xml nn-hdfs-NAMENODE/core-site.xml

# Examining Hadoop Daemon Log Files
    7. View the NameNode log file using the NameNode Web UI.
    From the Cloudera Manager HDFS page, click the NameNode Web UI link.
    From the NameNode Web UI, select Utilities > Logs.
    The list of directories and files in the /var/log/hadoop-hdfs directory on elephant appears.
    Open the NameNode log file and review the file.
    8. Access the daemon logs directly from Cloudera Manager.
    In Cloudera Manager, choose Hosts > All Hosts from the top navigation bar.
    Select elephant and then choose the Processes tab.
    Locate the row for the NameNode and click Full log file.
    The Log Details page opens at the tail end of the NameNode log file. Scroll up to view earlier log messages.
    Note: If the log file is very large, and you want to see messages near the top, scrolling in the Cloudera Manager UI will be slow.       Other tools provide quick access to the entire log file.
    Click Download Full Log to download the entire log.
    9. Review the NameNode daemons standard error and standard output logs using Cloudera Manager.
    Return to the Processes page for elephant.
    Click the Stdout link for the NameNode instance. The standard output log appears.
    Review the file, then return to the Processes page.
    Click the Stderr link for the NameNode instance. The standard error log appears.
    Review the file.
    Note: if you want to locate these log files on disk, they can be found on elephant
    in the /var/log/hadoop-hdfs and /var/run/cloudera-scm-agent/
    process/nn-hdfs-NAMENODE/logs directories.
    10. Using Cloudera Manager, review recent entries in the SecondaryNameNode logs.
    To find the log, go to the HDFS Instances page for your cluster, then locate the
    SecondaryNameNode role type and click the tiger link in the Host column.
    In the Status page for the tiger host, scroll down to the Roles area and click Role
    Log File in the SecondaryNameNode column.
    © Copyright 2010–2018 Cloudera. All Rights Reserved.
    Not to be reproduced or shared without prior written consent from Cloudera. 44
    45
    11. Access the ResourceManager log file using the ResourceManager Web UI.
    Navigate to the ResourceManager Web UI (from the Cloudera Manager YARN
    page’s Web UI menu or by specifying the URL http://horse:8088 in your
    browser).
    Choose Nodes from the Cluster menu on the left side of the page.
    Click the horse:8042 link to be taken to the NodeManager Web UI.
    Expand the Tools menu on the left side of the page.
    Click Local logs.
    Finally, click the entry for the ResourceManager log file and review the file.
    
    
# Sqoop Usage Examples
    ▪ List all databases
        $ sqoop list-databases --username fred -P \
        --connect jdbc:mysql://dbserver.example.com/
    ▪ List all tables in the world database
        $ sqoop list-tables --username fred -P \
        --connect jdbc:mysql://dbserver.example.com/world
    ▪ Import all tables in the world database
        $ sqoop import-all-tables --username fred --password derf \
        --connect jdbc:mysql://dbserver.example.com/world
        
# The WebHDFS REST API
    REST: REpresentational State Transfer
    ▪ Provides an HTTP/HTTPS REST interface to HDFS
        ─ Supports both reads and writes from/to HDFS
        ─ Can be accessed from within a program or script
        ─ Can be used via command-line tools such as curl or wget
    ▪ Installs with the HDFS service in Cloudera Manager
        ─ Enabled by default (dfs.webhdfs.enabled)
    ▪ Requires client access to every DataNode in the cluster
    ▪ Does not support HDFS HA deployments
    
# The HttpFS REST API
    ▪ Provides an HTTP/HTTPS REST interface to HDFS
        ─ The interface is identical to the WebHDFS REST interface
    ▪ Installable part of the HDFS service in Cloudera Manager
        ─ Choose to install it when install HDFS or Add Role Instances from existing
    deployed HDFS service
        ─ Installs and configure an HttpFS server
        ─ Enables proxy access to HDFS for an httpfs user
    ▪ Requires client access to the HttpFS server only
        ─ The HttpFS server then accesses HDFS
    ▪ Supports HDFS HA deployments
    
# The WebHDFS/HttpFS REST Interface Examples
    ▪ These examples will work with either WebHDFS or HttpFS
        ─ For WebHDFS, specify the NameNode host and port (default: 50070)
        ─ For HttpFS, specify the HttpFS server and port (default: 14000)
    ▪ Open and get the shakespeare.txt file
        $ curl -i -L "http://host:port/webhdfs/v1/tmp/\shakespeare.txt?op=OPEN&ampuser.name=training"
    ▪ Make the mydir directory
    $ curl -i -X PUT "http://host:port/webhdfs/v1/user/\training/mydir?op=MKDIRS&user.name=training"

# Importing Data: Best Practices
    ▪ Best practice is to import data into a temporary directory
    ▪ After the file is completely written, move data to the target directory
        ─ This is an atomic operation
        ─ Happens very quickly since it merely requires an update of the NameNode’s metadata
    ▪ Many organizations standardize on a directory structure such as
        ─ /incoming/<import_job_name>/<files>
        ─ /for_processing/<import_job_name>/<files>
        ─ /completed/<import_job_name>/<files>
    ▪ It is the job’s responsibility to move the files from for_processing to completed after the job has finished successfully
    ▪ Discussion point: your best practices?

# Cluster Growth Based on Storage Capacity (PROCESSO DO FRAMEWORK DE DECISAO)
    ▪ Basing your cluster growth on storage capacity is often a good method
    ▪ Example:
        ─ Data grows by approximately 3TB per week
        ─ HDFS set up to replicate each block three times
        ─ Therefore, 9TB of extra storage space required per week
        ─ Plus some overhead—say, 25% of all disk space
        ─ Assuming machines with 16 x 3TB hard drives, this equates to a new machine required every four weeks
        ─ Alternatively: Two years of data—1.2 petabytes—will require approximately 35 machines
        
# Configuring The System (2)
    ▪ Hadoop has no specific disk partitioning requirements
        ─ Use whatever partitioning system makes sense to you
    ▪ Mount disks with the noatime option
    ▪ Common directory structure for data mount points:
        /data/<n>/dfs/nn
        /data/<n>/dfs/dn
        /data/<n>/dfs/snn
        /data/<n>/mapred/local
    ▪ Reduce the swappiness of the system in /etc/sysctl.conf
        ─ Set vm.swappiness to 1
    ▪ Configure IPTables if required by your security policies—Hadoop requires many ports for communication
        ─ From Cloudera Manager’s Cluster page, choose Configuration > All Port
    Configurations to see all ports used
    
# Configuring The System (3)
    ▪ Disable Transparent Huge Page compaction
        ─ Can degrade the performance of Hadoop workloads
        ─ Open the defrag file of your OS to see if it is enabled
        ─ Red Hat/CentOS: /sys/kernel/mm/\redhat_transparent_hugepage/defrag
        ─ Ubuntu/Debian, OEL, SLES: /sys/kernel/mm/\transparent_hugepage/defrag
        ─ A line reading [always] never means it is enabled
        ─ A line reading [never] always means it is disabled
        ─ To temporarily disable it
    $ sudo sh -c "echo 'never' > defrag_file_pathname"
        ─ Add the following to /etc/rc.local to persist the change
      echo never > defrag_file_pathname
      
# Filesystem Considerations
    ▪ Cloudera recommends that you use one of the following filesystems tested on the supported operating systems:
        ─ ext3: The most tested underlying filesystem for HDFS
        ─ ext4: Scalable extension of ext3, supported in more recent Linux releases
        ─ XFS: The default filesystem in RHEL 7

# Operating System Parameters
    ▪ Increase the nofile ulimit for the cloudera-scm user to at least 32K
        ─ Cloudera Manager sets this to 32K in /usr/sbin/cmf-agent by default
    ▪ Disable IPv6
    ▪ Disable SELinux if possible
        ─ Incurs a performance penalty on a Hadoop cluster
        ─ Configuration is non-trivial
        ─ When doing this, disable it on each host before deploying CDH on the cluster
    ▪ Install and configure the ntp daemon
        ─ Ensures the time on all nodes is synchronized
        ─ Important for HBase, ZooKeeper, Kerberos
        ─ Useful when using logs to debug problems

# Java Virtual Machine (JVM) Requirements
    ▪ Always use the official Oracle JDK (http://java.com/)
        ─ Only 64-bit JDKs from Oracle are supported
        ─ Oracle JDK 7 is supported across all versions of Cloudera Manager 5 and CDH 5
        ─ Oracle JDK 8 is supported in C5.3.x and higher
    ▪ Running CDH nodes within the same cluster on different JDK releases is not supported
        ─ JDK release across a cluster needs to match the patch level
        ─ All nodes in your cluster must run the same Oracle JDK version
        ─ All services must be deployed on the same Oracle JDK version
    ▪ For version specific information see:
        ─ http://tiny.cloudera.com/jdk
        
# Essential Points of Configuration
    ▪ Master nodes run the NameNode, Standby NameNode (or Secondary NameNode), and ResourceManager
        ─ Provision with carrier-class hardware and lots of RAM
    ▪ Worker nodes run DataNodes and NodeManagers
        ─ Provision with industry-standard hardware
        ─ Consider your data storage growth rate when planning current and future cluster size
    ▪ RAID (on worker nodes) and virtualization can incur a performance penalty
    ▪ Make sure that forward and reverse domain lookups work when configuring a cluster
    ▪ Consider Cloudera Director and Cloud Options for cost savings / other benefits

# Hive Tables
    ▪ A Hive Table represents an HDFS directory
        ─ Hive interprets all files in the directory as the contents of the table
        ─ Hive stores information about how the rows and columns are delimited within the files in the Hive Metastore
    ▪ Tables are created as either managed or external
        ─ Managed
        ─ If the table is dropped, the schema and the HDFS data is deleted
        ─ External
        ─ If the table is dropped, only the table schema is deleted
    ▪ The default location for any table is /user/hive/warehouse
        ─ This path can be overridden by using the LOCATION keyword

# Hive Is Not an RDBMS
    ▪ Note that Hive is not an RDBMS!
        ─ Results take several seconds, minutes, or even hours to be produced
        ─ Not possible to modify the data using HiveQL
        ─ UPDATE and DELETE are not supported
        
# Hive Service Roles Installed by Cloudera Manager
    ▪ Hive Metastore Server
        ─ Manages the remote Metastore process
    ▪ HiveServer2
        ─ Supports the Thrift API used by clients
        ─ Has a Hive CLI named Beeline
    ▪ Hive Gateway
    ─ To deploy Hive client configurations to specific nodes in the cluster, add the Hive Gateway role instance to those nodes
    ─ Nodes with the Gateway role instance will receive the latest client configurations when you choose Deploy Client Configuration from  Cloudera Manager
    ─ Client Configuration files can also be manually downloaded and distributed
    
# Hive Metastore Server (Remote mode)
    ─ Recommended deployment mode
    ─ Datastore resides in a standalone RDBMS such as MySQL
    ─ HiveServer2 and other processes, such as Impala, communicate with the metastore service via JDBC
    ─ Advantage over Local mode:
    ─ Do not need to share JDBC login information for metastore database with each Hive user
    ▪ Local mode
    ─ Functionality of Metastore Server embedded in the HiveServer process
    ─ Database runs separately, accessed via JDBC
    ▪ Embedded mode
    ─ Supports only one active user—for experimental purposes only
    ─ Uses Apache Derby, a Java-based RDBMS

# HiveServer2
    ▪ HiveServer2 runs Hive as a server
        ─ A container for the Hive execution engine
        ─ Enables remote clients to execute queries against Hive and retrieve the results
        ─ Accessible via JDBC, ODBC, or Thrift
        ─ Example clients: Beeline (the Hive CLI), Hue (Web UI)
    ▪ Supports concurrent queries from more than one Hive client
        ─ Why is concurrency support needed?
        ─ Example: a Hive client issues a DROP TABLE command, while at the same time another Hive client running on a different machine runs 
            SELECT query against the same table
        ─ Hive concurrency requires ZooKeeper to guard against data corruption

# HiveServer2 Uses ZooKeeper
    ▪ ZooKeeper is a distributed coordination service for Hadoop
        ─ Allows processes to share information with each other in a reliable and redundant way
    ▪ Used for many purposes
        ─ Examples: Leader election, distributed synchronization, failure recovery
    ▪ ZooKeeper always runs on an odd number of machines
        ─ Called a “ZooKeeper ensemble”
        ─ Makes the service highly available
    ▪ HiveServer2 uses ZooKeeper to support concurrent clients
        ─ Client processes could be running on different machines
    ▪ Other Hadoop components can share the same ZooKeeper ensemble
    
# Configuring Hive on Spark or MapReduce
    ▪ Hive in CDH supports two execution engines: MapReduce and Spark
    ▪ Execution engine used by Beeline—can be set per query
        ─ Run the set hive.execution.engine=engine command
        ─ Where engine is either mr or spark. The default is mr
    ▪ Cloudera Manager (Affects all queries, not recommended)
        ─ Configuration tab of Hive service
        ─ Set the Default Execution Engine to MapReduce or Spark. The default is MapReduce.

# Cloudera Search (Solr)
    ▪ Cloudera Search provides an interactive full-text search capability for data in
    your Hadoop cluster
    ─ Makes the data accessible to non-technical audiences
    ─ Compare: Write code for Spark or MapReduce vs SQL queries vs use a search
    engine
    ─ Foundation of Cloudera Search is Apache Solr
    ─ High-performance, scalable, reliable
    ─ Indexing and query can be distributed
    ─ Open source, standard Solr APIs and Web UI
    ─ Supports 30 languages
    ─ Integration with HDFS, MapReduce, HBase, and Flume
    ─ Support for common Hadoop file formats
    ─ Hue web-based dashboard and search interface
    ─ Apache Sentry access control


# How Cloudera Search Works
    ▪ Client connects to a Solr host in the cluster
        ─ Hardcoded in client to connect to a single host
        ─ Zookeeper ensemble: round robin Solr hosts
        ─ Supported for Java SolrJ API only
        ─ Load balancer: for more sophisticated distribution and resiliency
    ▪ Solr host
        ─ Distributes search work to other Solr nodes
        ─ Aggregates results (including its own)
        ─ Return results to client

# Indexing
    ▪ Data must be indexed prior to search
        ─ Indexing involves data extraction, transformation, and is similar to database
    design
        ─ Once indexed, basic queries and advanced queries are possible
    ▪ Data indexing methods
        ─ Use Flume to index data on entry to cluster
        ─ Use MapReduce for batch index of data in HDFS
        ─ Indexing of data in HBase is also available
        
# HDFS—NameNode Tuning
    Para Calcular Logaritmo Natural: https://www.wolframalpha.com/
    ln(67)*20 = 84

    dfs.namenode.handler.count
    Set in HDFS / NameNode Group / Performance
        ▪ The number of server threads for the NameNode that listen to requests from clients
        ▪ Threads used for RPC calls from clients and DataNodes (heartbeats and metadata operations)
        ▪ Default in Cloudera Manager: 30 (non-CM default is 10)
        ▪ Recommended: Natural logarithm of the number HDFS nodes x 20
        ▪ Symptoms of this being set too low: “connection refused” messages in DataNode logs as they try to transmit block reports to the NameNode
        ▪ Used by the NameNode
        
# HDFS—DataNode Tuning
    dfs.datanode.failed.volumes.tolerated
    
    Set in HDFS / DataNode Group
        ▪ The number of volumes allowed to fail before the DataNode takes itself offline, ultimately resulting in all of its blocks being re-replicated
        ▪ CM Default: 0
        ▪ For each DataNode, set to (number of mountpoints on DataNode host) / 2
        ▪ Used by DataNodes
        
    dfs.datanode.max.locked.memory
    Set in HDFS / DataNode Group / Resource Management
        ▪ The maximum amount of memory (in bytes) a DataNode can use for caching
        ▪ CM Default: 4GB
        ▪ Must be less than the value of the OS configuration property ulimit -l for the DataNode user
        ▪ Used by DataNodes
                
# File Compression
    io.compression.codecs
    Set in HDFS / Service-Wide
        ▪ List of compression codecs that Hadoop can use for file compression
        ▪ If you are using another codec, add it here
        ▪ CM default value includes the following org.apache.hadoop.io.compress
            codecs: DefaultCodec, GzipCodec, BZip2Codec, DeflateCodec, SnappyCodec, Lz4Codec
        ▪ Used by clients and all nodes running Hadoop daemons

# MapReduce—Reducer Scheduling and Input Fetch
    mapreduce.job.reduce.slowstart.completedmaps
    Set in YARN / Gateway Group
        ▪ The percentage of Map tasks which must be completed before the ResourceManager
    will schedule Reducers on the cluster
        ▪ CM Default: 0.8 (80 percent)
        ▪ Recommendation: 0.8 (80 percent)
        ▪ Used by the ResourceManager
    mapreduce.reduce.shuffle.parallelcopies
    Set in YARN / Gateway Group
        ▪ Number of threads a Reducer can use in parallel to fetch Mapper output
        ▪ CM Default: 10
        ▪ Recommendation: ln(number of cluster nodes) * 4 with a floor of 10
        ▪ Used by ShuffleHandler
        

# MapReduce—Speculative Execution
    ▪ If a MapReduce task is running significantly more slowly than the average speed of tasks for that job, speculative execution may occur
        ─ Another attempt to run the same task is instantiated on a different node
        ─ Results from the first completed task are used, the slower task is killed
    mapreduce.map.speculative
    Set in YARN / Gateway Group
        ▪ Whether to allow speculative execution for Map tasks
        ▪ CM Default: false, Recommendation: false
        ▪ Used by MapReduce ApplicationMasters
    mapreduce.reduce.speculative
    Set in YARN / Gateway Group
        ▪ Whether to allow speculative execution for Reduce tasks
        ▪ CM Default: false, Recommendation: false
        ▪ Used by MapReduce ApplicationMasters
        
# Common Hadoop Ports
    ▪ Hadoop daemons each provide a Web-based user interface
        ─ Useful for both users and system administrators
    ▪ Expose information on a variety of different ports
        ─ Port numbers are configurable, although there are defaults for most
    ▪ Hadoop also uses various ports for components of the system to communicate with each other
    ▪ Full list of ports used by components of CDH 5:
        ─ http://www.cloudera.com/documentation/enterprise/
    latest/topics/cdh_ig_ports_cdh5.html
    ▪ All ports used in a Cluster are listed in one location in Cloudera Manager
        ─ From the Cluster page’s Configuration menu, choose All Port Configurations

Web UI Ports for Users
Daemon Default
Port
Configuration parameter
NameNode 50070 dfs.namenode.http-address
DataNode 50075 dfs.datanode.http.address
HDFS
Secondary NameNode 50090 dfs.namenode.secondary.http-address
ResourceManager 8088 yarn.resourcemanager.webapp.address
NodeManager 8042 yarn.nodemanager.webapp.address
YARN
MR JobHistoryServer 19888 mapreduce.jobhistory.webapp.address
Cloudera Manager 7180 Configure in Cloudera Manager by going to
Administration > Settings > Ports and Addresses
Hue 8888 http_port
Other
Spark History Server 18088 history.port


Daemon Default
Port
Configuration parameter
NameNode 50070 dfs.namenode.http-address
DataNode 50075 dfs.datanode.http.address
HDFS
Secondary NameNode 50090 dfs.namenode.secondary.http-address
ResourceManager 8088 yarn.resourcemanager.webapp.address
NodeManager 8042 yarn.nodemanager.webapp.address
YARN
MR JobHistoryServer 19888 mapreduce.jobhistory.webapp.address
Cloudera Manager 7180 Configure in Cloudera Manager by going to
Administration > Settings > Ports and Addresses
Hue 8888 http_port
Other
Spark History Server 18088 history.port
-----
Daemon Default
Port
Configuration Parameter Used for
NameNode 8020 fs.defaultFS Filesystem metadata
operations
DataNode 50010 dfs.datanode.address DFS data transfer
50020 dfs.datanode.ipc.address Block metadata operations
and recovery
ResourceManager 8032 yarn.resourcemanager.address Application submission; used
by clients
8033 yarn.resourcemanager.admin.
address
Administration RPC server
port; used by the yarn
rmadmin client
8030 yarn.resourcemanager.
scheduler.address
Scheduler RPC port; used by
ApplicationMasters
8031 yarn.resourcemanager.
resource-tracker.address
Resource tracker RPC port;
used by NodeManagers


Daemon Default
Port
Configuration Parameter Used for
NodeManager 8040 yarn.nodemanager.
localizer.address
Resource localization
RPC port
JobHistoryServer 10020 mapreduce.jobhistory.
address
JobHistoryServer RPC
port; used by clients to
query job history.
ShuffleHandler
(MapReduce’s
auxiliary service)
13562 mapreduce.shuffle.port ShuffleHandler’s HTTP
port; used for serving
Mapper outputs



# HDFS—Rack Topology Awareness
    ▪ Recall that HDFS is rack aware
        ─ Distributes blocks based on hosts’ locations
    ▪ To maximize performance, specify network locations of hosts and racks
        ─ Important for clusters that span more than one rack
        ─ In the Hosts page in Cloudera Manager, assign hosts to racks
        ─ Specify Rack ID in the form /datacenter/rack
        ─ Any host without a specified rack location shows /default location
        
# HDFS High Availability Overview
    ▪ A single NameNode is a single point of failure
    ▪ Two ways a NameNode can result in HDFS downtime
        ─ Unexpected NameNode crash (rare)
        ─ Planned maintenance of NameNode (more common)
    ▪ HDFS High Availability (HA) eliminates this SPOF
        ─ Available since CDH4 (or related Apache Hadoop 0.23.x, and 2.x)
    ▪ New Daemons that HDFS HA introduces to the cluster
        ─ NameNode (active)
        ─ NameNode (standby)
        ─ Failover Controllers
        ─ Journal Nodes
    ▪ The Secondary NameNode is not used in an HDFS HA configuration

# HDFS High Availability Architecture (1)
    ▪ HDFS High Availability uses a pair of NameNodes
        ─ One Active and one Standby
        ─ Clients only contact the Active NameNode
        ─ DataNodes heartbeat in to both NameNodes
        ─ Active NameNode writes its metadata to a quorum of JournalNodes
        ─ Standby NameNode reads from the JournalNodes to remain in sync with the Active NameNode
        
# HDFS High Availability Architecture (2)
    ▪ Active NameNode writes edits to the JournalNodes
        ─ Software to do this is the Quorum Journal Manager (QJM)
        ─ Built in to the NameNode
        ─ Waits for a success acknowledgment from the majority of JournalNodes
        ─ Majority commit means a single crashed or lagging JournalNode will not impact NameNode latency
        ─ Uses the Paxos algorithm to ensure reliability even if edits are being writtenas a JournalNode fails
    ▪ Note that there is no Secondary NameNode when implementing HDFS High Availability
        ─ The Standby NameNode periodically performs checkpointing

# Failover
    ▪ Only one NameNode must be active at any given time
        ─ The other is in standby mode
    ▪ The standby maintains a copy of the active NameNode’s state
        ─ So it can take over when the active NameNode goes down
    ▪ Two types of failover
        ─ Manual (detected and initiated by a user)
        ─ Automatic (detected and initiated by Hadoop itself)
        ─ Automatic failover uses ZooKeeper
        ─ When HA is configured, a daemon called the ZooKeeper Failover Controller (ZKFC) runs on each NameNode machine

# HDFS High Availability and Automatic Failover
    ▪ Cloudera Manager configures HA using Quorum-based storage
        ─ Uses a quorum of JournalNodes
        ─ Each JournalNode maintains a local edits directory
        ─ That directory contains files detailing namespace metadata modifications
    ▪ With HA configured, automatic failover is also available
        ─ Cloudera Manager sets dfs.ha.automatic-failover.enabled to true for the NameNode role instances it configures for HA
        
# Fencing
    ▪ It is essential that exactly one NameNode be active
        ─ Possibility of split-brain syndrome otherwise
    ▪ The Quorum Journal Manager ensures that a previously-active NameNode cannot corrupt the NameNode metadata
    ▪ When configuring HDFS HA, one or more fencing methods must be specified
        ─ Methods available:
        ─ shell: the Cloudera Manager default (uses CM Agent). Configure in HDFS > Configuration > Service-Wide > High Availability
        ─ sshfence: connects to the target node via ssh and kills the process listening on service’s TCP port
        ─ In order for failover to occur, one of the methods must run successfully

# HDFS HA Deployment—Without Cloudera Manager
    ▪ Without Cloudera Manager, enabling High Availability for HDFS is a long, complex, error prone process
        ─ Overview of major steps involved
            1. Modify multiple Hadoop configurations across all DataNodes
            2. Install and start the JournalNodes
            3. If not already installed, configure and start a ZooKeeper ensemble
            4. Initialize the shared edits directory if converting from a non-HA deployment
            5. Install, bootstrap, and start the Standby NameNode
            6. Install, format, and start the ZooKeeper failover controllers
            7. Restart DataNodes and the YARN and MapReduce daemons
    ▪ With Cloudera Manager it is a simpler process
    
# Enabling HDFS HA Using Cloudera Manager
    ▪ Three steps to Enable HDFS HA in Cloudera Manager
    1. Configure directories for JournalNodes to store edits directories
        ─ Set permissions on these new directories for the hdfs user
    2. Ensure the ZooKeeper service is installed and enabled for HDFS
    3. From the HDFS Instances page, run the Enable High Availability wizard
        ─ Specify the hosts for the two NameNodes and the JournalNodes
        ─ Specify the JournalNode Edits directory for each host
        ─ The wizard performs the necessary steps to enable HA
        ─ Including the creation and configuration of new Hadoop daemons

# After Enabling HDFS HA
    ▪ After enabling HDFS HA, some manual configurations may be needed
        ─ For Hive
        ─ Upgrade the Hive Metastore
        ─ Consult the Cloudera documentation for details
        ─ For Impala
        ─ Complete the same steps as for Hive (above)
        ─ Next, run INVALIDATE METADATA command from the Impala shell
        ─ For Hue
        ─ Add the HttpFS role (if not already on the cluster)

# Kerberos Exchange Participants (1)
    ▪ Kerberos involves messages exchanged among three parties
        ─ The client
        ─ The server providing a desired network service
        ─ The Kerberos Key Distribution Center (KDC)

# General Kerberos Concepts (2)
    ▪ Authenticated status is cached
        ─ You do not need to submit credentials explicitly with each request
    ▪ Passwords are not sent across network
        ─ Instead, passwords are used to compute encryption keys
        ─ The Kerberos protocol uses encryption extensively
    ▪ Timestamps are an essential part of Kerberos
        ─ Make sure you synchronize system clocks (NTP)
    ▪ It is important that reverse lookups work correctly

# Kerberos Terminology
    ▪ Knowing a few terms will help you with the documentation
    ▪ Realm
        ─ A group of machines participating in a Kerberos network
        ─ Identified by an uppercase domain (EXAMPLE.COM)
    ▪ Principal
        ─ A unique identity which can be authenticated by Kerberos
        ─ Can identify either a host or an individual user
        ─ Every user in a secure cluster will have a Kerberos principal
    ▪ Keytab file
        ─ A file that stores Kerberos principals and associated keys
        ─ Allows non-interactive access to services protected by Kerberos
        
# Hadoop Security Setup Prerequisites
    ▪ Working Hadoop cluster
        ─ Installing CDH from parcels or packages is strongly advised!
        ─ Ensure your cluster actually works before trying to secure it!
    ▪ Working Kerberos KDC server
    ▪ Kerberos client libraries installed on all Hadoop nodes

# Configuring Hadoop Security (1)
    ▪ Hadoop security configuration is a specialized topic
    ▪ Many specifics depend on
        ─ Version of Hadoop and related programs
        ─ Type of Kerberos server used (Active Directory or MIT)
        ─ Operating system and distribution
    ▪ You must follow instructions exactly
        ─ There is little room for misconfiguration
        ─ Mistakes often result in vague “access denied” errors
        ─ May need to work around version-specific bugs

# Configuring Hadoop Security (2)
    ▪ For these reasons, we can’t cover this in depth during class
    ▪ See the Cloudera Security documentation for detailed instructions
        ─ Available at http://tiny.cloudera.com/cdh-security
        ─ Be sure to read the details corresponding to your version of CDH
    ▪ The security recommendations document manual configurations
        ─ Configuring Hadoop with Kerberos involves many tedious steps
        ─ Cloudera Manager (Enterprise) automates many of them

# Securing Related Services
    ▪ There are many “ecosystem” tools that interact with Hadoop
    ▪ Most require minor configuration changes for a secure cluster
        ─ For example, specifying Kerberos principals or keytab file paths
    ▪ Exact configuration details vary with each tool
        ─ See documentation for details
    ▪ Some require no configuration changes at all
        ─ Such as Pig and Sqoop

# Active Directory Integration
    ▪ Microsoft’s Active Directory (AD) is a enterprise directory service
    ─ Used to manage user accounts for a Microsoft Windows network
    ▪ Recall that every Hadoop user must have a Kerberos principal
    ─ It can be tedious to set up all these accounts
    ─ Many organizations would prefer to use AD for Hadoop users
    ▪ Cloudera’s recommended approach
    ─ Integrate with Active Directory KDC or run a local MIT Kerberos KDC
    ─ Create all service principals (like hdfs and mapred) in this realm
    ▪ Instructions can be found in Cloudera’s CDH Security Guide

# Direct Active Directory Integration with Kerberos
    ▪ Integrate with an existing Active Directory KDC
    ▪ Wizard in Cloudera Manager enables Kerberos for the cluster
        ─ Generates and deploys Kerberos client configuration (krb5.conf)
        ─ Configures CDH components
        ─ Generates principals needed by all processes running in the cluster

# Encryption Overview
    ▪ Encryption seeks to ensure that only authorized users can view, use, or contribute to a data set
    ▪ Data protection can be applied at three levels within Hadoop:
        ─ Linux filesystem level
        ─ Example: Cloudera Navigator Encrypt
        ─ HDFS level
        ─ Encryption applied by the HDFS client software
        ─ HDFS Data At Rest Encryption operates at the HDFS folder level, enabling encryption to be applied only to the HDFS folders where it is needed.

# Navigator Key Trustee should be used
    ─ Network level
    ─ Encrypt data just before sending across a network and decrypt it as as it is received. This protection uses industry-standard protocols such as SSL/TLS

# OS Filesystem Level Encryption—Navigator Encrypt
    ▪ Production ready, and included with the Cloudera Navigator license
    ▪ Operations at the Linux volume level
        ─ Capable of encrypting cluster data inside and outside of HDFS
        ─ No change to application code required
    ▪ Provides a transparent layer between the application and file system reducing the performance impact of encryption
    ▪ Navigator Key Trustee
        ─ The default HDFS encryption uses a local Java keystore
        ─ Navigator Key Trustee is a “virtual safe-deposit box” keystore server for managing encryption keys, certificates, and passwords
        ─ Encryption keys stored separately from encrypted data

# HDFS Level “Data at Rest” Encryption
    ▪ Provides transparent end-to-end encryption of data read from and written to HDFS
        ─ No change to application code required
        ─ Data encrypted and decrypted only by the HDFS client
        ─ HDFS does not store or have access to unencrypted data or keys
    ▪ Operates at the HDFS folder level
        ─ Apply to folders where needed
    ▪ Deployment Overview
        ─ Use Cloudera Manager to deploy the Hadoop Key Management Server (KMS) service for storing keys
        ─ Create Encryption Zones, then add files to the Encryption Zones

# Network Level Encryption
    ▪ Transport Layer Security (TLS)
        ─ Provides communication security over the network to prevent snooping
    ▪ Enable TLS between the Cloudera Manager Server and Agents—three levels available
        ─ Level 1 (good): Encrypts communication between browser and CM and between agents and CM Server
        ─ Level 2 (better): Adds strong verification of CM Server certificate
        ─ Level 3 (Best): Authentication of agents to the CM Server using certs
    ▪ Configure SSL encryption for CDH services (HDFS, YARN, and so on)
        ─ Example: Encrypt data transferred between DataNodes and between DataNodes and clients
        ─ Recommendation: enable Kerberos on the cluster first
        ─ Steps to configure encryption differs for each CDH service
        ─ See Cloudera documentation for details

# Apache Sentry 
    ▪ Sentry is a plugin for enforcing role-based authorization for Search, Hive and Impala
        ─ Stores authorization policy metadata
        ─ Provides clients with concurrent and secure access to this metadata
        ─ Database-style authorization for Hive, Impala, and Search
    ▪ Sentry requirements
        ─ CDH 4.3 or later
        ─ HiveServer2 and Hive Metastore running with strong authentication
        ─ Impala 1.2.1 or later running with strong authentication
        ─ Kerberos enabled on the cluster
    ▪ Use Cloudera Manager’s Add Service wizard to install Sentry
        ─ Requires HDFS and ZooKeeper roles
        ─ Deploys a “Sentry Server” daemon

# Sentry Access Control Model
    ▪ What does Sentry control access to?
        ─ Server, Database, Table, Column, View
    ▪ Who can access Sentry-controlled objects?
        ─ Users in a Sentry role
        ─ Sentry roles = one or more groups
    ▪ How can role members access Sentry-controlled objects?
        ─ Read operations (SELECT privilege)
        ─ Write operations (INSERT privilege)
        ─ Metadata definition operations (ALL privilege)
    ▪ Option to synchronize HDFS ACLs and Sentry permissions
        ─ Set user access permissions and securely share the same HDFS data with other components (Pig, Spark, MR, and HDFS clients, and so on)
        ─ Example use case: importing data into Hive using Sqoop
