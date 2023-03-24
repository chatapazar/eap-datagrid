
#jboss cli
./jboss-cli.sh
# add core module jdbc driver
module add --name=com.oracle --resources=/path/to/ojdbc7.jar --dependencies=javaee.api,sun.jdk,ibm.jdk,javax.api,javax.transaction.api
# example
module add --name=com.oracle --resources=/Users/ckongman/work/workspace/eap-datagrid/lib/ojdbc7.jar --dependencies=javaee.api,sun.jdk,ibm.jdk,javax.api,javax.transaction.api

exit

# verify module in jboss path
$JBOSS_HOME/modules/com/oracle/main


./jboss-cli.sh --file=datasource.cli

./standalone.sh --server-config=standalone-ha.xml

#run add-user.sh for enable admin user/password

# Testing in console web JBoss/EAP 7:
Standalone mode: Configuration => Subsystems => Datasources => Non-XA or XA => DataSourceName => View => Connection => Test Connect.

Domain mode: Runtime => Hosts => HostName => ServerName=> Subsystem => Datasources =>Non-XA or XA => DataSourceName => Dropdown option beside View => Test Connection => Test Connect.

# Test with CLI
./jboss-cli.sh
connect
/subsystem=datasources/xa-data-source=OracleXADS:test-connection-in-pool