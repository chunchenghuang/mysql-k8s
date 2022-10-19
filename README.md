# MySQL Examples with K8s
Practicing setting up mysql services

## Deploy
`cd` to dp-with-nodeport or statefulset directory
```
kubectl apply -f .
```

## Secrets
Recommend using HashiCorp Vault or see also [here](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).

## Testing
### For simple deployment with nodeport
Resources include:
- Namespace
- Secret
- Deployment
- NodePort

**Run mysql-client**
```
kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -p -h mysql-service.mysql.svc.cluster.local
```
Then enter your password
```
# sql commands
create database justfortest;
exit;
```

Check the [website](https://dev.mysql.com/doc/refman/8.0/en/password-security-user.html) for ways to input password:

### For statefulset
Resources include:
- Namespace
- Secret
- StatefulSet
- Headless Service
- Persistent Volume
- Persistent Volume Claim
- Storage Class

**Check the dns resolution of statefulset pods**
```
# go inside any pod
kubectl run -it --rm --restart=Never dnsutils2 --image=tutum/dnsutils --command -- bash

# inside the bash
nslookup mysql-service.mysql
```
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql-service.mysql.svc.cluster.local
Address: 172.17.0.3
Name:   mysql-service.mysql.svc.cluster.local
Address: 172.17.0.4
Name:   mysql-service.mysql.svc.cluster.local
Address: 172.17.0.5
```
# then nslookup your address endpoint
nslookup 172.17.0.3
```
Server:         10.96.0.10
Address:        10.96.0.10#53

3.0.17.172.in-addr.arpa name = mysql-0.mysql-service.mysql.svc.cluster.local.

You get your FQDN `mysql-0.mysql-service.mysql.svc.cluster.local`

**Run mysql-client**
```
kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -p -h mysql-0.mysql-service.mysql.svc.cluster.local
```
Then enter your password
```
# sql commands
create database justfortest;
exit;
```

Same for other pods, just swap the name of mysql-0 to others.

## Delete

```
kubectl delete ns mysql
kubectl delete pv --all -n mysql
kubectl delete sc --all -n mysql
```

## Comments
- `kubernetes.io/no-provisioner` was used for local volumes, which did not support dynamic provisioning, therefore we still needed to create 3 pvs with different host paths (stateful applications have sticky identity therefore need unique storage for each pod) to match the number of replicas.
- Still need to set up the data cloning and data sync as StatefulSets only help you to set up a stateful application.
