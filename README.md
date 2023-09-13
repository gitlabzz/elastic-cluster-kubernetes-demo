# elastic-cluster-kubernetes-demo

### Reference
[Quick Start Guide](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html)

[Samples](https://github.com/elastic/cloud-on-k8s/tree/2.9/config/samples)

[Volume claim templates](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-volume-claim-templates.html)

[Setup Your Own Certificated for ELK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-tls-certificates.html#k8s-setting-up-your-own-certificate)

### Steps:

#### 1:Install [custom resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) definitions:
kubectl create -f https://download.elastic.co/downloads/eck/2.9.0/crds.yaml

#### 2: The ECK operator runs by default in the elastic-system namespace.
kubectl apply -f https://download.elastic.co/downloads/eck/2.9.0/operator.yaml

#### 3: Monitor the operator logs:
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator


#### 4: Deploy Elasticsearch and Kibana
kubectl apply -f elasticsearch-and-kibana-deployment.yaml

Retrieve elastic user password using:
echo $(kubectl get secret quickstart-es-elastic-user -n eck -o jsonpath='{.data.elastic}' | base64 --decode; echo)

-- OR --

PASSWORD=$(kubectl -n eck get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

will be used to login to elastic and kibana:

From inside cluster:
curl -u "elastic:$PASSWORD" -k "https://quickstart-es-http:9200"

-- OR --

Using Ingress:
https://elastic.example.com
https://kibana.example.com
https://kibana.172.16.240.240.nip.io


# Kubernetes Logging Operator

## References
[Logging Operator](https://kube-logging.dev/docs/)

## Install
### Installation Guide
[Logging Operator Installation](https://kube-logging.dev/docs/install/)

### Deploy Logging operator with Helm:
helm upgrade --install --wait --create-namespace --namespace logging logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator

### Validate the deployment:
kubectl -n logging get pods

### Check the CRDs. You should see the following five new CRDs:
kubectl get crd

NAME
1. clusterflows.logging.banzaicloud.io
2. clusteroutputs.logging.banzaicloud.io
3. eventtailers.logging-extensions.banzaicloud.io
4. flows.logging.banzaicloud.io
5. fluentbitagents.logging.banzaicloud.io
6. hosttailers.logging-extensions.banzaicloud.io
7. loggings.logging.banzaicloud.io
8. nodeagents.logging.banzaicloud.io
9. outputs.logging.banzaicloud.io
10. syslogngclusterflows.logging.banzaicloud.io
11. syslogngclusteroutputs.logging.banzaicloud.io
12. syslogngflows.logging.banzaicloud.io
13. syslogngoutputs.logging.banzaicloud.io


### Deploy Sample:
kubectl apply -f logging-operator-test.yaml

#### manually update:
secret -> quickstart-fluentbit -> namespace: logging -> from quickstart-fluentd.logging.svc.cluster.local -> to quickstart-fluentd.logging
because fluentbit pods DNS resolution fails for quickstart-fluentd.logging.svc.cluster.local service name.

#### Update Logging Operator Deployment to include test receiver:
helm upgrade --install --wait --create-namespace --namespace logging --set testReceiver.enabled=true logging-operator oci://ghcr.io/kube-logging/helm-charts/logging-operator
kubectl -n logging get deployments
kubectl -n logging get services

#### Install log generator:
helm upgrade --install --wait --create-namespace --namespace logging log-generator oci://ghcr.io/kube-logging/helm-charts/log-generator

#### Check if log receiver is receiving any logs from log generator:
kubectl logs --namespace logging -f svc/logging-operator-test-receiver


#### pre-requisites - create ILM Policy with the name "test-rollover-logs" 

curl -k -u "elastic:84UKWxvCPLE2e859t0a7TK72" -X PUT "https://elastic.example.com/_ilm/policy/test-rollover-logs" -H "kbn-xsrf: reporting" -H "Content-Type: application/json" --data-binary "@ilm-request.json"

#### Now we can deploy logging operator components to shop logs to elasticsearch


### Cleanup:
kubectl get logging-all --namespace logging

kubectl delete logging quickstart
kubectl delete --namespace logging flow log-generator
kubectl delete --namespace logging output http

kubectl delete logging default-logging-simple
kubectl delete --namespace logging flow es-flow
kubectl delete --namespace logging output es-output
























