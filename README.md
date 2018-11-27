# ClouderaAdmin


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
