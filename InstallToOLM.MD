# TODO 
 ```
 operator-sdk generate csv --csv-version 0.2.0 --update-crds
 pip3 install operator-courier
 operator-courier verify --ui_validate_io todoapp-operator/deploy/olm-catalog/todoapp-operator 
 #need to fix some missing things
 oc apply -f deploy/olm-catalog/todoapp-operator/0.2.0/todoapp-operator.v0.2.0.clusterserviceversion.yaml

```
