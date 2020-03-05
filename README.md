refer to:

https://github.com/operator-framework/operator-sdk-samples/tree/master/ansible

https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md

https://github.com/burrsutter/quarkus-todo-app/tree/master/kubefiles

# memcache operator
Install operator sdk
```
$ RELEASE_VERSION=v0.15.2

$ curl -LO https://github.com/operator-framework/operator-sdk/releases/download/${RELEASE_VERSION}/operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu

$ chmod +x operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu && sudo mkdir -p /usr/local/bin/ && sudo cp operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu /usr/local/bin/operator-sdk && rm operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu
```
Create Project
```
$ operator-sdk new memcached-operator --api-version=cache.example.com/v1alpha1 --kind=Memcached --type=ansible

$ cd memcached-operator
```
Modify Watches.yaml

ConfigMaps owned by a Memcached CR will not be watched or cached.

```
- version: v1alpha1
  group: cache.example.com
  kind: Memcached
  role: /opt/ansible/roles/memcached
  blacklist:
    - group: ""
      version: v1
      kind: ConfigMap
```

 roles/memcached/defaults/main.yml
``` 
Size: 1
```

roles/memcached/tasks/main.yml 
```
- name: start memcached
  k8s:
    definition:
      kind: Deployment
      apiVersion: apps/v1
      metadata:
        name: '{{ meta.name }}-memcached'
        namespace: '{{ meta.namespace }}'
      spec:
        replicas: "{{size}}"
        selector:
          matchLabels:
            app: memcached
        template:
          metadata:
            labels:
              app: memcached
          spec:
            containers:
            - name: memcached
              command:
              - memcached
              - -m=64
              - -o
              - modern
              - -v
              image: "docker.io/memcached:1.4.36-alpine"
              ports:
                - containerPort: 11211
```
Deploy
```
oc new-project testopmemch

oc create -f deploy/crds/cache.example.com_memcacheds_crd.yaml

operator-sdk build quay.io/jaysonzhao/memcached-operator:v0.0.1

docker push quay.io/jaysonzhao/memcached-operator:v0.0.1
```

deploy/operator.yaml
   
   Replace this with the built image name
   ```
          image: "quay.io/jaysonzhao/memcached-operator:v0.0.1"
          imagePullPolicy: "Always"
          volumeMounts:
          - mountPath: /tmp/ansible-operator/runner
            name: runner
            readOnly: true
        - name: operator
          # Replace this with the built image name
          image: "quay.io/jaysonzhao/memcached-operator:v0.0.1"
          imagePullPolicy: "Always"
```

```
$ oc create -f deploy/service_account.yaml

$ oc create -f deploy/role.yaml

$ oc create -f deploy/role_binding.yaml

$ oc create -f deploy/operator.yaml

```

deploy/crds/cache.example.com_v1alpha1_memcached_cr.yaml
```
apiVersion: "cache.example.com/v1alpha1"
kind: "Memcached"
metadata:
  name: "example-memcached"
spec:
  size: 5
```
```
 oc apply -f deploy/crds/cache.example.com_v1alpha1_memcached_cr.yaml
```


# Todoapp Operator

Install operator sdk
```
$ RELEASE_VERSION=v0.15.2

$ curl -LO https://github.com/operator-framework/operator-sdk/releases/download/${RELEASE_VERSION}/operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu

$ chmod +x operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu && sudo mkdir -p /usr/local/bin/ && sudo cp operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu /usr/local/bin/operator-sdk && rm operator-sdk-${RELEASE_VERSION}-x86_64-linux-gnu
```
Create Project
```
$ operator-sdk new todoapp-operator --api-version=todoapp.example.com/v1alpha1 --kind=Todoapp --type=ansible

$ cd todoapp-operator 
```
Modify Watches.yaml
ConfigMaps owned by a Memcached CR will not be watched or cached.
```
---
- version: v1alpha1
  group: todoapp.example.com
  kind: Todoapp
  role: /opt/ansible/roles/todoapp
  blacklist:
    - group: ""
      version: v1
      kind: ConfigMap
```

Modify deploy/role.yaml
Add the following part
```
- apiGroups:
  - route.openshift.io
  resources:
  - routes
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```

 roles/todoapp/defaults/main.yml
``` 
size: 1
```

roles/todoapp/tasks/main.yml 

Please refer to :

https://raw.githubusercontent.com/jaysonzhao/operatordemo/master/todoapp-operator/roles/todoapp/tasks/main.yml


Deploy
```
oc new-project testoptodo

oc create -f deploy/crds/todoapp.example.com_todoapps_crd.yaml

operator-sdk build quay.io/jaysonzhao/todoapp-operator:v0.0.1

docker push quay.io/jaysonzhao/todoapp-operator:v0.0.1
```

deploy/operator.yaml
Replace this with the built image name
```
          image: "quay.io/jaysonzhao/todoapp-operator:v0.0.1"
          imagePullPolicy: "Always"
          volumeMounts:
          - mountPath: /tmp/ansible-operator/runner
            name: runner
            readOnly: true
        - name: operator
          # Replace this with the built image name
          image: "quay.io/jaysonzhao/todoapp-operator:v0.0.1"
          imagePullPolicy: "Always"

```
```
$ oc create -f deploy/service_account.yaml

$ oc create -f deploy/role.yaml

$ oc create -f deploy/role_binding.yaml

$ oc create -f deploy/operator.yaml
```


deploy/crds/todoapp.example.com_v1alpha1_todoapp_cr.yaml
```
apiVersion: "todoapp.example.com/v1alpha1"
kind: "Todoapp"
metadata:
  name: "mytodo"
spec:
  size: 1
```
```
 oc apply -f deploy/crds/todoapp.example.com_v1alpha1_todoapp_cr.yaml
```



Destroy
```

oc delete -f deploy/crds/todoapp.example.com_v1alpha1_todoapp_cr.yaml
```
