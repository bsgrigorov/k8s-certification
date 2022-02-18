# K8s Commands
Get secret value
```
b64dec $(k get secret posdtna-poc-postgres -o=jsonpath='{.data.postgres\.properties}')
```

Get resources with one of labels
```
k get po -l 'app in (alertmanager,prometheus)'
```

Patch
```
kubectl patch prometheus prometheus -n kube-system --type=json -p '[{"op":"add","path":"/spec/volumeMounts","value":[{"mountPath":"/data","name":"storage-volume"}]}]'
kubectl patch prometheus prometheus -n kube-system --type=json -p '[{"op":"remove","path":"/spec/volumeMounts"}]'
```

Force delete pod set grace period to 0
```
kubectl delete pod --grace-period=0 --force POD_NAME
```
https://containersolutions.github.io/runbooks/posts/kubernetes/pod-stuck-in-terminating-state/

Exec command
```
k exec podname -- command
```
Remove finalizers
```
kubectl patch pod [POD_NAME] -p '{"metadata":{"finalizers":null}}'
```

Install nginx ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/cloud/deploy.yaml
```

Show labels
```
k get po --show-labels
```

Get nodes of pods
```
k get po -o wide
```

Create a pod
```
k run nginx --image=nginx
```
nginx is the name of the pod

Extract definition to a file 
```
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```

Change namespace context
```
kubectl config set-context --current --namespace=
```

View config
```
kubectl config view
```

Get pods with label
```
k get po -l environment=production,tier=frontend
```

Set based requirements
```
kubectl get pods -l 'environment in (production, dev),tier in (frontend)'
```

Resources explain
```
k api-resources # get api resources in k8s
k explain mutatingwebhookconfigurations # explain api resource
```

Convert api version
```
k convert -f file.yaml --output-version=<new-version>
```

Check if a pod has access to a svc or pod
```
nc -v -z HOST PORT
```

Execute a command in a container
```
command:
- sh
- -c
- mycommands
```

Add shiftwidth to vimrc
```
echo 'set sw=2' > .vimrc
```

Send token to file
```
k get secret -n neptune neptune-sa-v2-token-kq65j -o jsonpath='{.data.token}' | base64 -d > file1
```

Rollout deployment revision undo
```
k rollout undo deploy api-new-c32  --to-revision 4 -n neptune
```

Run nginx and execute curl command
```
k run nginx --image nginx
k exec nginx -- sh -c "curl web-moon.moon.svc.cluster.local"
```

Get IP of nodes
```
k get nodes -o wide
```

Label based on other labels
```
k label pods protected=true -n sun -l type=runner
```

Annotate pods
```
k annotate pods protected='do not delete this pod' -n sun -l protected=true
```

Get logs of previous instance of pod before restart
```
kubectl logs nginx -p
```

Run a pod and delete it after
```
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
```

Get IPs of pods that can be used to access them from other pods
```
kubectl get po -o wide
```

Create pods with labels
```
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
# or
for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1 ; done
```

Override labels
```
kubectl label po nginx2 app=v2 --overwrite
```

Get label column for specific label
```
kubectl get po -L app
```

Remove label from pods
```
kubectl label po -l app app-
```

Show labels of a pod
```
k label pod nginx --list
```

Show annotations of a pod
```
kubectl annotate pod nginx1 --list
kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

Remove annotations
```
kubectl annotate po nginx{1..3} description-
```

Check rollout status
```
kubectl rollout status deploy nginx
```

Get secret value 
```
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d
```

Get pods with liveness probes failed
```
kubectl get events | grep -i "Liveness probe failed"
```

Change image in place
```
k set image deployment/depl1 container-name=nginx:1.12
```


# Fast creation of resources

Dry run output
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Deployment
```
kubectl create deployment --image=nginx nginx
kubectl create deployment --image=nginx nginx --dry-run -o yaml
kubectl create deployment nginx --image=nginx --replicas=4
```

Scale deployment
```
kubectl scale deployment nginx --replicas=4
```

Save yaml definition file starter
```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Service
```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
This will automatically use the pod's labels as selectors

```
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

Run pod and expose container port 
```
k run custom-nginx --image nginx --port 8080
```

Ingress
```
kubectl create ingress <ingress-name> --rule="host/path=service:port"
```

Role
```
k create role --verb=list,create,delete --resource=pods developer
```

Change image in place
```
k set image deployment/depl1 container-name=nginx:1.12
```

Config 
```
kubectl create configmap app-config --from-literal=APP_COLOR=blue \
  --from-literal=APP_MOD=prod
kubectl create configmap \
  app-config --from-file=app_config.properties
```

Run
```
kubectl run nginx --image=nginx (deployment)
kubectl run nginx --image=nginx --restart=Never (pod)
kubectl run nginx --image=nginx --restart=OnFailure (job)
kubectl run nginx --image=nginx --restart=OnFailure --schedule="* * * * *" (cronJob)
```

Secret
```
kubectl create secret generic db-user-pass --from-literal=username='devuser'
```
characters such as $, \, *, =, and ! will be interpreted by your shell and require escaping with '

Get secret data jsonpath
```
kubectl get secret db-user-pass -o jsonpath='{.data}'
```

Rollout
```
kubectl rollout history deployment/frontend                      # Check the history of deployments including the revision 
kubectl rollout undo deployment/frontend                         # Rollback to the previous deployment
kubectl rollout undo deployment/frontend --to-revision=2         # Rollback to a specific revision
kubectl rollout status -w deployment/frontend                    # Watch rolling update status of "frontend" deployment until completion
kubectl rollout restart deployment/frontend                      # Rolling restart of the "frontend" deployment
```

Pod force recreate
```
# Force replace, delete and then re-create the resource. Will cause a service outage.
kubectl replace --force -f ./pod.json
```

Label and annotate
```
kubectl label pods my-pod new-label=awesome
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq
```

Patch
```
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'
```

Save in notepad
```
k config set-context context-name --namespace=nsname
```

Get api resources
```
kubectl api-resources --namespaced=true      # All namespaced resources
kubectl api-resources --namespaced=false     # All non-namespaced resources
kubectl api-resources -o name                # All resources with simple output (only the resource name)
kubectl api-resources -o wide                # All resources with expanded (aka "wide") output
kubectl api-resources --verbs=list,get       # All resources that support the "list" and "get" request verbs
kubectl api-resources --api-group=extensions # All resources in the "extensions" API group
```

Resource quota
```
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

Return to specific revision
```
kubectl rollout undo deploy nginx --to-revision=2
```

Check details of specific revision
```
kubectl rollout history deploy nginx --revision=4
```

Autoscale deployment
```
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
# view the horizontalpodautoscalers.autoscaling for nginx
kubectl get hpa nginx
```

Job terminate if it takes more than 30 seconds to complete
```
activeDeadlineSeconds: 30
```

Create cronjob
```
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
# startingDeadlineSeconds: 17 # time deadline for starting the execution
```

Create configmap from with key
```
kubectl create cm configmap4 --from-file=special=config4.txt
data:
  special: |
    var3=val3
    var4=val4
```

Run pod and set resources
```
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

Run pod and expose service
```
k run nginx --image nginx --port 80 --expose
```

Run container with command
```
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
```

Patch a resource
```
kubectl patch svc nginx -p '{"spec":{"type":"NodePort"}}'
```



