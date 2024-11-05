MYELIN_VERSION=0.11
REGISTRY=gcr.io/myelin-public
APP_NAME=myelin
NAMESPACE=test-ns

.PHONY: submit-crd
submit-crd:
	kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
	helm install --devel myelin-app-crd myelin.io/myelin-crd

.PHONY: create-roles
create-roles:
	kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=tamas.jambor@myelin.io
	kubectl apply -f role/axon-controller-role.yaml
	kubectl apply -f role/prometheus-operator-role.yaml
	kubectl apply -f role/nfs-provisioner-role.yaml
	kubectl apply -f role/nfs-provisioner-storageclass.yaml

.PHONY: tiller
tiller:
	kubectl -n kube-system create sa tiller
	kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
	helm init --service-account tiller

.PHONY: build
build:
	helm dependency build chart/myelin-mp
	docker buildx build --platform linux/amd64 --push --tag ${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} .

.PHONY: test-deploy
test-deploy:
	kubectl create namespace test-ns
	kubectl label namespace test-ns istio-injection=enabled
	kubectl apply -n ${NAMESPACE} -f role/myelin-minimal-role.yaml
	# set defaults in schema.yaml
	~/bin/mpdev install \
		  --deployer=${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} \
      --parameters='{"name": "myelin", "namespace": "test-ns", "myelin.prometheus.namespace": "prometheus", "myelin.prometheus.serviceName": "prometheus-operated", "myelin.prometheus-pushgateway.namespace": "myelin", "myelin.prometheus-pushgateway.serviceName": "myelin-app-prometheus-pushgateway", "myelin.axonController.prometheusRules.namespace": "prometheus"}'

.PHONY: verify
verify:
	# this creates a new namespace in the active cluster and runs the installation process using the default values from data-test/schema and data/test/chart
	~/bin/mpdev verify \
		  --deployer=${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION}