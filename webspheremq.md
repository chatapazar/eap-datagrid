docker run \
  --env LICENSE=accept \
  --env MQ_QMGR_NAME=QM1 \
  --env MQ_APP_PASSWORD=passw0rd \
  --publish 1414:1414 \
  --publish 9443:9443 \
  --detach \
  icr.io/ibm-messaging/mq

https://localhost:9443/ibmmq/console

User: admin/passw0rd
User: app/passw0rd

./standalone.sh --server-config=standalone-ha.xml

#run add-user.sh for enable admin user/password

First, deploy the resource adapter manually by copying the wmq.jmsra.rar file to the EAP_HOME/standalone/deployments/ directory.

Next, use the management CLI to add the resource adapter and configure it.
./jboss-cli.sh

/subsystem=ejb3:write-attribute(name=default-resource-adapter-name, value=${ejb.resource-adapter-name:wmq.jmsra.rar})
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar:add(archive=wmq.jmsra.rar,transaction-support=XATransaction,statistics-enabled=true)

/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF:add(class-name=com.ibm.mq.connector.outbound.ManagedConnectionFactoryImpl,jndi-name=JMS2CF, use-java-context=false)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=hostName:add(value=localhost)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=password:add(value=passw0rd)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=queueManager:add(value=QM1)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=port:add(value=1414)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=channel:add(value=DEV.APP.SVRCONN)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=transportType:add(value=CLIENT)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=JMS2CF/config-properties=username:add(value=app)

/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF:add(class-name=com.ibm.mq.connector.outbound.ManagedConnectionFactoryImpl,jndi-name=jms/ivt/IVTCF, use-java-context=false)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=hostName:add(value=localhost)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=password:add(value=passw0rd)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=queueManager:add(value=QM1)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=port:add(value=1414)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=channel:add(value=DEV.APP.SVRCONN)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=transportType:add(value=CLIENT)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/connection-definitions=IVTCF/config-properties=username:add(value=app)

/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/admin-objects=QueuePool:add(class-name=com.ibm.mq.connector.outbound.MQQueueProxy,jndi-name=jms/ivt/IVTQueue,use-java-context=true,enabled=true)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/admin-objects=QueuePool/config-properties=baseQueueName:add(value=DEV.QUEUE.1)
/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar/admin-objects=QueuePool/config-properties=baseQueueManagerName:add(value=QM1)

/subsystem=resource-adapters/resource-adapter=wmq.jmsra.rar:activate()