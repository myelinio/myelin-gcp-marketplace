MYELIN_VERSION=0.4
REGISTRY=gcr.io/myelin-public
APP_NAME=myelin
NAMESPACE=myelin-ns

.PHONY: submit-crd
submit-crd:
	kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/alertmanager.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/prometheus.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/prometheusrule.crd.yaml"
	kubectl apply -f "https://raw.githubusercontent.com/coreos/prometheus-operator/master/example/prometheus-operator-crd/servicemonitor.crd.yaml"
	helm install myelin.io/myelin-crd --devel

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
	docker build --tag ${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} .
	docker push ${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION}

.PHONY: test-deploy
test-deploy:
	kubectl create namespace ${NAMESPACE}
	kubectl label namespace ${NAMESPACE} istio-injection=enabled
	kubectl apply -n ${NAMESPACE} -f role/myelin-minimal-role.yaml
	~/bin/mpdev /scripts/install \
		  --deployer=${REGISTRY}/${APP_NAME}/deployer:${MYELIN_VERSION} \
      --parameters='{"name": "myelin", "namespace": "test-ns"}'