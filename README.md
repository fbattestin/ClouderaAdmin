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

