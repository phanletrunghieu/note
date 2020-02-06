db:
	helm install pg --namespace postgres stable/postgresql --set \
	postgresqlPassword=secretpassword,\
	postgresqlDatabase=my-database,\
	replication.enabled=true,\
	replication.slaveReplicas=2,\
	replication.synchronousCommit="on",\
	replication.numSynchronousReplicas=1,\
	metrics.enabled=true
	# service.type=LoadBalancer,
down:
	helm delete pg --namespace postgres