# Install to OLM
 ```
 operator-sdk generate csv --csv-version 0.2.0 --update-crds
 pip3 install operator-courier
 operator-courier verify --ui_validate_io todoapp-operator/deploy/olm-catalog/todoapp-operator 
 #need to fix some missing things
 # https://github.com/operator-framework/getting-started/issues/39
 # issue gone in 0.17.0
 
 # https://github.com/operator-framework/getting-started/pull/36
 # https://github.com/operator-framework/operator-sdk-samples/issues/57
 # still need to apply operatorgroup in 17.0
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

#0.17.1+  added manefest package feature
operator-sdk generate csv --make-manifests=false --csv-version 0.2.0

Create CatalogSource
Replace ${REGISTRY} with YOUR_REGISTRY_URL
echo "apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: internal-mirrored-operatorhub-catalog
  namespace: openshift-marketplace
spec:
  displayName: My Mirrored Operator Catalog
  sourceType: grpc
  image: ${REGISTRY}/openshift-marketplace/mirrored-operator-catalog
" > internal-mirrored-operatorhub-catalog.yaml

Apply the CatalogSource
```
oc create -f internal-mirrored-operatorhub-catalog.yaml
```


 oc create -f deploy/service_account.yaml
 oc create -f deploy/role.yaml
 oc create -f deploy/role_binding.yaml
 oc apply -f deploy/olm-catalog/operatorGroup.yaml
 oc apply -f deploy/olm-catalog/todoapp-operator/manifest/todoapp-operator.v0.2.0.clusterserviceversion.yaml

```
