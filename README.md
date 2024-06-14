

```shell


RO_SECRET_NAME=$(kubectl get secret -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='readonly-sa')].metadata.name}")
RO_TOKEN=$(kubectl get secret $RO_SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode)
RO_CURRENT_CONTEXT=$(kubectl config current-context)
RO_CLUSTER_NAME=$(kubectl config get-contexts $RO_CURRENT_CONTEXT | awk '{print $3}' | tail -n 1)
RO_CLUSTER_SERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name == \"$RO_CLUSTER_NAME\")].cluster.server}")
kubectl config set-context readonly-sa-context --cluster=$RO_CLUSTER_NAME --user=readonly-sa-user --namespace=default
kubectl config set-credentials readonly-sa-user --token=$RO_TOKEN


RW_SECRET_NAME=$(kubectl get secret -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='full-access-sa')].metadata.name}")
RW_TOKEN=$(kubectl get secret $RW_SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode)
RW_CURRENT_CONTEXT=$(kubectl config current-context)
RW_CLUSTER_NAME=$(kubectl config get-contexts $RW_CURRENT_CONTEXT | awk '{print $3}' | tail -n 1)
RW_CLUSTER_SERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name == \"$RW_CLUSTER_NAME\")].cluster.server}")
kubectl config set-context full-access-sa-context --cluster=$RW_CLUSTER_NAME --user=full-access-sa-user --namespace=default
kubectl config set-credentials full-access-sa-user --token=$RW_TOKEN



#1. Deploy Successfully:
kubectl config use-context full-access-sa-context
helm diff upgrade test . --allow-unreleased --debug
helm upgrade --install test .  --debug     

#2. Change the labels from first to second in the accounts.yaml

# 3.Deploy Failed :
kubectl config use-context readonly-sa-context
helm ls # the release is is failed state.
helm diff upgrade test . --allow-unreleased --debug          
helm upgrade --install test .  --debug     
helm ls # the release is is failed state.


# 4. Deployment Issue :
kubectl config use-context full-access-sa-context
helm diff upgrade test . --allow-unreleased --debug
helm upgrade --install test .  --debug     
          
       

 
              
```
