# eap-datagrid
download eap from 
https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=appplatform&version=7.4

create data grid user
redhat-datagrid-8.4.1-server/bin
./cli.sh user create jbosseap -p jbosseap -g admin

start data grid
redhat-datagrid-8.4.1-server/bin
./server.sh -c infinispan.xml --cluster-stack=tcp --node-name=jdg1 --bind-address=127.0.0.1

configure EAP
jboss-eap-7.4/bin
cat <<EOT > wf.cli
embed-server --std-out=echo --server-config=standalone-ha.xml
/subsystem=jgroups/channel=ee:write-attribute(name=stack,value=tcp)
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=infinispan-server:add(port=11222,host=127.0.0.1)
batch
/subsystem=infinispan/remote-cache-container=jdg_rc:add(default-remote-cluster=infinispan-server-cluster, properties={infinispan.client.hotrod.auth_username=jbosseap, infinispan.client.hotrod.auth_password=jbosseap}, protocol-version=3.0, statistics-enabled=true)
/subsystem=infinispan/remote-cache-container=jdg_rc/remote-cluster=infinispan-server-cluster:add(socket-bindings=[infinispan-server])
run-batch
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic:add()
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic/store=hotrod:add(remote-cache-container=jdg_rc,fetch-state=false,preload=false,passivation=false,purge=false,shared=true)
/subsystem=infinispan/cache-container=web:write-attribute(name=default-cache, value=jdg_ic)
EOT

./jboss-cli.sh --file=wf.cli

start EAP
jboss-eap-7.4/bin
./standalone.sh --server-config=standalone-ha.xml -Djboss.default.jgroups.stack=tcp -Dprogram.name=wfl1 -Djboss.node.name=wfl1

cp ../clusterbench-ee8.ear ./jboss-eap-7.4/standalone/deployments/

http://localhost:8080/clusterbench/session