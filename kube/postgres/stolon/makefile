db:
	helm install pg --namespace postgres -f values.yaml stable/stolon

down:
	helm delete pg --namespace postgres
	# kubectl delete --all statefulsets.apps --namespace=postgres
	# kubectl delete --all replicasets.apps --namespace=postgres
	# kubectl delete --all jobs.batch --namespace=postgres
	# kubectl delete --all deployments.apps --namespace=postgres
	# kubectl delete --all service --namespace=postgres
	# kubectl delete --all pods --namespace=postgres
	# kubectl delete --all Role -n postgres
	# kubectl delete --all RoleBinding -n postgres
	# kubectl delete --all PodDisruptionBudget -n postgres
	# kubectl delete --all ServiceAccount -n postgres
	# kubectl delete --all ConfigMap -n postgres