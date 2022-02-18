# K8s Examples
Some examples of K8s yaml that has a lot of different options and properties setup. Review them thoroughly.

Start an nginx pod
```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

```
export POD=$(kubectl get pods --namespace test -o jsonpath="{.items[0].metadata.name}")
ke $POD test
```

ubuntu
```
ka https://gist.githubusercontent.com/pjh/d7e51b75ee152a5aed7cf1b6df2a72f6/raw/be6c879821013e9b82d6b4980926256cd77d6737/linux-ubuntu-deployment.yaml
```

secret
```
apiVersion: v1
kind: Secret
metadata:
  name: testsecret
annotations:
  sealedsecrets.bitnami.com/cluster-wide: "true"
type: Opaque
data:

```

sealed secret
```
kubeseal < .ci/charts/templates/secret.yaml > .ci/charts/templates/sealedsecret2.yaml ~/Development/env/sealed-secrets-cert.pem --format=yaml
```

Get jsonpath property
```
kubectl -n dev1 get secret document-library-elasticsearch -o=jsonpath='{.data.es\.properties}' | base64 -d
```

Cannot delete a resource
- Need to remove the finalizer
```
kubectl patch crd/postgresexporter-sample -p '{"metadata":{"finalizers":[]}}' --type=merge
k patch postgresexporter/postgresexporter-sample2 -p '{"metadata":{"finalizers":[]}}' --type=merge
```

Get pod name
```
k get pods -o=jsonpath={.items[0].metadata.name}
```

Copy files
```
kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar
```

Copy all files in a folder in pod
```
files= $(kubectl exec -ti -n [namespace] [pod] -- /bin/ls /[directory]/)
for file in $files
do
     kubectl cp -n [namespace] [pod]:/[directory]/$file ./$file
done
```

Move secret from one namespace to another 
```
kubectl get secret btp -n cic-system -o yaml \
| sed s/"namespace: cic-system"/"namespace: dev"/\
| kubectl apply -n dev -f -
```

Pod with all options
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  restartPolicy: Never
  serviceAccountName: myuser
  secrutyContext:
    runAsUser: 101
  containers:
  - name: nginx
    image: nginx:1.14.2

    ports:
    - containerPort: 80

    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]

    readinessProbe:
      httpGet: # can also be tcpSocket (no path necessary)
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 60
      timeoutSeconds: 10
      successThreshold: 1
      failureThreshold: 3

    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60

    env:
    - name: ENV_VAR1
      valueFrom:
        configMapKeyRef:
          name: cmname
          key: cmkey
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password

    envFrom:
    - secretRef:
        name: mysecret
    - configMapRef:
        name: mycm

    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: app_type
              operator: In
              values:
              - beta
    volumeMounts:
    - name: cm-volume
      mountPath: /etc/config

    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"

    - mountPath: /test-ebs
      name: aws-volume

    - mountPath: /cache
      name: cache-volume

    - mountPath: /hostvol
      name: host-volume

  volumes:
  - name: cm-volume
    configMap:
      name: cm
      items:
      - key: cm_key
        path: log_level

  - name: secret-volume
    secret:
      secretName: dotfile-secret

  - name: aws-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: "<volume id>"
      fsType: ext4

  - name: cache-volume
    emptyDir: {}

  - name: host-volume
    hostPath:
      path: /data  # directory location on host
      type: Directory
```

Example secret for env
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

Example secret file
```
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
```

Example configmap for env
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

Persistent Volume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```

Network Policy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

Policies that allow dns resolution
```
- ports:
  - port: 53
    protocol: TCP
  - port: 53
    protocol: UDP
```

Job
```
apiVersion: batch/v1
kind: Job
metadata:
  name: job1
spec:
  backoffLimit: 6
  completions: 10
  parallelism: 1
  selector:
    matchLabels:
      job-name: whalesay
  template:
    metadata:
      labels:
        job-name: whalesay
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - cowsay I am going to ace CKAD!
        image: docker/whalesay
        name: whalesay
      restartPolicy: Never
```

PVC example
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: earth-project-earthflower-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Volumes/Data

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: earth-project-earthflower-pvc
  namespace: earth
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: project-earthflower
  namespace: earth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: project-earthflower
  template:
    metadata:
      labels:
        app: project-earthflower
    spec:
      containers:
      - name: project-earthflower
        image: httpd:2.4.41-alpine
        volumeMounts:
        - mountPath: /tmp/project-data
          name: mount
      volumes:
      - name: mount
        persistentVolumeClaim:
          claimName: earth-project-earthflower-pvc
```
