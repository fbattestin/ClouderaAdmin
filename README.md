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


