MYELIN_VERSION=v0.3.2
REGISTRY=gcr.io/myelin-public
APP_NAME=myelin

.PHONY: submit-crd
submit-crd:
	kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/alertmanager.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/prometheus.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/prometheusrule.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/servicemonitor.crd.yaml"

.PHONY: create-roles
create-roles:
	kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=tamas.jambor@myelin.io
	kubectl apply -f role/axon-controller-role.yaml
	kubectl apply -f role/prometheus-operator-role.yaml
	kubectl apply -f role/nfs-provisioner-role.yaml
	kubectl apply -f role/nfs-provisioner-storageclass.yaml

.PHONY: build
build:
	helm dependency build chart/myelin-mp
	docker build --tag ${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} .
	docker push ${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION}

.PHONY: test-deploy
test-deploy:
	kubectl create namespace test-ns
	kubectl apply -n test-ns -f role/myelin-minimal-role.yaml
	~/bin/mpdev /scripts/install \
      --deployer=${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} \
      --parameters='{"name": "myelin", "namespace": "test-ns"}'