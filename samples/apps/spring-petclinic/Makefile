PETCLINIC_REPO ?= quay.io/service-binding/spring-petclinic
PETCLINIC_APP_IMAGE ?= $(PETCLINIC_REPO):latest

NAMESPACE ?= my-postgresql

MULTIARCH_PLATFORMS ?= linux/amd64,linux/arm64,linux/s390x,linux/ppc64le

# one of "openshift", "minikube"
KUBERNETES_RUNTIME ?= minikube

.PHONY: namespace
namespace:
ifeq ($(KUBERNETES_RUNTIME), openshift)
	-oc new-project $(NAMESPACE)
else ifeq ($(KUBERNETES_RUNTIME), minikube)
	-kubectl create namespace $(NAMESPACE)
	kubectl config set-context --current --namespace=$(NAMESPACE)
endif

.PHONY: deploy-database
deploy-database:
	kubectl apply -f db-deployment.yaml -n $(NAMESPACE)

.PHONY: build-app
build-app:
	docker build -f Dockerfile.app -t $(PETCLINIC_APP_IMAGE) .

.PHONY: push-app
push-app:
	docker push $(PETCLINIC_APP_IMAGE)

.PHONY: build-app-multiarch
build-app-multiarch:
	docker buildx build --platform "$(MULTIARCH_PLATFORMS)" -f Dockerfile.app -t $(PETCLINIC_APP_IMAGE) .

.PHONY: push-app-multiarch
push-app-multiarch:
	docker buildx build --push --platform "$(MULTIARCH_PLATFORMS)" -f Dockerfile.app -t $(PETCLINIC_APP_IMAGE) .

.PHONY: deploy-app
deploy-app:
	PETCLINIC_APP_IMAGE=$(PETCLINIC_APP_IMAGE) \
	NAMESPACE=$(NAMESPACE) \
	$(KUBERNETES_RUNTIME)/deploy-app.sh

.PHONY: bind
bind:
	kubectl apply -f service-binding.yaml -n $(NAMESPACE)

.PHONY: unbind
unbind:
	kubectl delete -f service-binding.yaml -n $(NAMESPACE)
