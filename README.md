# STEPS TO SETUP 

1. AWS cloud
https://aws.amazon.com/

2. New User / Account : IAM
user -> power : policy : admin access

3. password (key)
access / secrey key

------------------------------------

4. download : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

5. login in CLI # aws configure

6. eksctl tool : install kubernetes

https://github.com/eksctl-io/eksctl/releases/tag/v0.161.0


7. cd C:\Program Files\Kubernetes

8. install kubernetes over AWS cloud
# eksctl create cluster --name priyansh --region=ap-southeast-1

9. install kubectl to connect to kubernetes
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

10. connect check : kubectl get nodes

# devops_eks_jenkins_loki_operator

```shell
$ eksctl create cluster --name mycluster1 --region=us-east-1
```
```shell
$ eksctl get  cluster --name mycluster1 --region=us-east-1
```
```shell
$ kubectl get nodes
```
```shell
kubectl create -f \
https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/crds.yaml
```
```shell
kubectl create -f \
https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/olm.yaml
```
```shell
kubectl get catalogsources -n olm
```
```shell
kubectl get packagemanifests -l catalog=operatorhubio-catalog

https://github.com/helm/helm/releases
```
```shell
$ kubectl create namespace myns
$ ./helm.exe  repo add jenkins https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/chart
$ ./helm.exe install my-jenkins-operator jenkins/jenkins-operator -n myns --set jenkins.enabled=false
```
```shell
$ kubectl --namespace myns get pods -w
```
# YAML CODE FOR JENKINS SETUP USING KUBERNETES OPERATOR
```YAML
$ vim jenkins_instance.yaml
apiVersion: jenkins.io/v1alpha2
kind: Jenkins
metadata:
  name: example
  namespace: myns
spec:
  configurationAsCode:
    configurations: []
    secret:
      name: ""
  groovyScripts:
    configurations: []
    secret:
      name: ""
  jenkinsAPISettings:
    authorizationStrategy: createUser
  master:
    disableCSRFProtection: false
    containers:
      - name: jenkins-master
        image: jenkins/jenkins:lts
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 12
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 100
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 1500m
            memory: 3Gi
          requests:
            cpu: "1"
            memory: 500Mi
  seedJobs:
    - id: jenkins-operator
      targets: "cicd/jobs/*.jenkins"
      description: "MY Jenkins Operator repository"
      repositoryBranch: master
      repositoryUrl: https://github.com/jenkinsci/kubernetes-operator.git
```
```shell
$ kubectl create -f jenkins_instance.yaml
$ kubectl --namespace myns get pods -w
```
```shell
$ kubectl --namespace myns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d

```
```shell
$ kubectl --namespace myns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d
```
```shell
$ kubectl --namespace myns port-forward jenkins-example 8080:8080
```
```shell
$ ./helm.exe  repo add grafana https://grafana.github.io/helm-charts
$ ./helm.exe  repo update
```

```shell
$ ./helm.exe upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false
```
```shell
$ kubectl patch svc loki-grafana -p '{"spec": {"type": "LoadBalancer"}}'
$ kubectl get svc loki-grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

```shell
$ kubectl get secret loki-grafana -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```

Loki log explorer and give this LogQL query in Grafana:
{app="loki-medium-logs",namespace="default"}
{namespace="myns"}

Clean up:
```shell
$ helm delete loki 
$ kubectl delete deploy loki-medium-logs 
```


https://grafana.com/grafana/dashboards/15141-kubernetes-service-logs/
https://grafana.com/grafana/dashboards/315-kubernetes-cluster-monitoring-via-prometheus/






      
 