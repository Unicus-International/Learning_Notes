

### Get a resource as a yaml file
```kubectl get pods -n <namespace> <resource name> -o yaml```  
If you want to write it to a file to review it in an IDE add ti output indicator:  
```kubectl get pods -n <namespace> <resource name> -o yaml > review.yaml```  
This overwrites review.yaml

### Enter the shell of the pod
```kubectl exec -n <namespace> -it <podname> -- /bin/bash```  

### Helm
```helm install -n <namespace> <chart-name> /path-to-chart/```  
```helm uninstall -n <namespace> <chart-name>```  
```helm upgrade -n <namespace> <chart-name> /path-to-chart/```  

### Set kubectl environment to azure cluster
```az aks get-credentials -g <resource-group> -n <name-of-resource> --overwrite-existing --admin```  
