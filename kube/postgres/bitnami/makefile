db:
	helm install pg bitnami/postgresql-ha --namespace postgres --set \
	postgresql.username=postgres,\
	postgresql.password=123456,\
	postgresql.database=app

down:
	helm delete pg --namespace postgres