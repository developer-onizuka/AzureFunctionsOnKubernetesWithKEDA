# Azure Functions On Kubernetes With KEDA

# 1. Run registry
In my case, the private registry run at 192.168.1.5, which is outside of the kubernetes cluster.
```
$ sudo docker run --rm --name registry -d -p 5000:5000 -v /mnt/registry:/var/lib/registry registry:2.7.1
```

# 2. Install KEDA with Helm in Kubernetes Master node
```
# helm repo add kedacore https://kedacore.github.io/charts
"kedacore" has been added to your repositories
root@master:/home/vagrant# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kedacore" chart repository
...Successfully got an update from the "csi-driver-nfs" chart repository
...Successfully got an update from the "nvidia" chart repository
...Successfully got an update from the "mellanox" chart repository
Update Complete. ⎈Happy Helming!⎈

# kubectl create namespace keda
namespace/keda created

# helm install keda kedacore/keda --namespace keda
NAME: keda
LAST DEPLOYED: Mon Apr  4 08:58:51 2022
NAMESPACE: keda
STATUS: deployed
REVISION: 1
TEST SUITE: None

# kubectl get pods -o wide -n keda
NAME                                              READY   STATUS    RESTARTS   AGE   IP              NODE      NOMINATED NODE   READINESS GATES
keda-operator-8644dcdb79-j9tb9                    1/1     Running   0          18m   10.10.235.135   worker1   <none>           <none>
keda-operator-metrics-apiserver-66d6c4454-k9z8d   1/1     Running   0          18m   10.10.235.134   worker1   <none>           <none>
```

# 3. Install Azure-functions-core-tools

```
# wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
# sudo dpkg -i packages-microsoft-prod.deb
# sudo apt-get update
# sudo apt-get install azure-functions-core-tools-4
# func --version
4.0.3971
```

# 4. Build App as Docker container
```
# mkdir keda
# cd keda

# func new --docker
Select a number for worker runtime:
1. dotnet
2. dotnet (isolated process)
3. node
4. python
5. powershell
6. custom
Choose option: 4

python
Found Python version 3.8.10 (python3).
Writing requirements.txt
Writing getting_started.md
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing /home/vagrant/keda/.vscode/extensions.json
Writing Dockerfile
Writing .dockerignore

Select a number for template:
1. Azure Blob Storage trigger
2. Azure Cosmos DB trigger
3. Durable Functions activity
4. Durable Functions entity
5. Durable Functions HTTP starter
6. Durable Functions orchestrator
7. Azure Event Grid trigger
8. Azure Event Hub trigger
9. HTTP trigger
10. Kafka output
11. Kafka trigger
12. Azure Queue Storage trigger
13. RabbitMQ trigger
14. Azure Service Bus Queue trigger
15. Azure Service Bus Topic trigger
16. Timer trigger
Choose option: 9

HTTP trigger
Function name: [HttpTrigger] myfuncapp
Writing /home/vagrant/keda/myfuncapp/__init__.py
Writing /home/vagrant/keda/myfuncapp/function.json
The function "myfuncapp" was created successfully from the "HTTP trigger" template.

# ls
Dockerfile  getting_started.md  host.json  local.settings.json  myfuncapp  requirements.txt

# docker build -t myfuncapp:v1 .
```

# 5. Deploy it to the Kubernetes Cluster
```
# func kubernetes deploy --name myfunctionapp --registry 192.168.1.5:5000
Running 'docker build -t 192.168.1.5:5000/myfunctionapp:latest /home/vagrant/keda'..done
secret/myfunctionapp created
secret/func-keys-kube-secret-myfunctionapp created
serviceaccount/myfunctionapp-function-keys-identity-svc-act created
role.rbac.authorization.k8s.io/functions-keys-manager-role created
rolebinding.rbac.authorization.k8s.io/myfunctionapp-function-keys-identity-svc-act-functions-keys-manager-rolebinding created
service/myfunctionapp-http created
deployment.apps/myfunctionapp-http created
Waiting for deployment "myfunctionapp-http" rollout to finish: 0 of 1 updated replicas are available...
deployment "myfunctionapp-http" successfully rolled out
	myfuncapp - [httpTrigger]
	Invoke url: http://192.168.33.223/api/myfuncapp?code=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

	Master key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

You can find the container at your private registry.
```
$ curl 192.168.33.1:5000/v2/_catalog
{"repositories":["face_recognizer","myfunctionapp","ubuntu"]}

$ curl 192.168.33.1:5000/v2/myfunctionapp/tags/list
{"name":"myfunctionapp","tags":["latest"]}
```

# 6. Access to it
<img src="https://github.com/developer-onizuka/AzureFunctionsOnKubernetesWithKEDA/blob/main/azureFunc1.png" width="640">
