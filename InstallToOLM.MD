# Install to OLM
 ```
 operator-sdk generate csv --csv-version 0.2.0 --update-crds
 pip3 install operator-courier
 operator-courier verify --ui_validate_io todoapp-operator/deploy/olm-catalog/todoapp-operator 
 #need to fix some missing things
 
 # https://github.com/operator-framework/getting-started/pull/36
 cat deploy/olm-catalog/operatorGroup.yaml 
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  annotations:
    olm.providedAPIs: todoapp.example.com
  generation: 2
  name: myog
spec:
  targetNamespaces:
  - testoptodo


 oc create -f deploy/service_account.yaml
 oc create -f deploy/role.yaml
 oc create -f deploy/role_binding.yaml
 oc apply -f deploy/olm-catalog/operatorGroup.yaml
 oc apply -f deploy/olm-catalog/todoapp-operator/0.2.0/todoapp-operator.v0.2.0.clusterserviceversion.yaml

```
