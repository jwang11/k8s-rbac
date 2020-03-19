# k8s-rbac
show k8s rbac usage

# subject -user
## create user private key
```
$ openssl genrsa -out jwang.key 2048
```

## generate certificate signing request
```
$ openssl req -new -key jwang.key -out jwang.csr -subj "/CN=jwang/O=intelssp"
```

## generate CA-signed certificate - 500 days
```
$ sudo openssl x509 -req -in jwang.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.ke
y -CAcreateserial -out jwang.crt -days 500

# print certificate content
$ openssl x509 -in jwang.crt -text -noout
```

## create credentials of new user
```
$ kubectl config set-credentials jwang --client-certificate=jwang.crt --client-key=jwang.key
```

## create k8s context for new user
```
$ kubectl config set-context jwang-context --cluster=kubernetes --namespace=kube-system --user=jwang
# check config status
$ kubectl config view
```

## talk with apiserver to get pods via new user.
```
$ kubectl get pods --context=jwang-context
```
It is forbidden as new user not grant any role

## create role which is allowed to operate pod 
```
$ kubectl apply -f jwang-role.yml
```

## bind user with role
```
$ kubectl create -f jwang-rolebindingl.yml
```

## test role again
```
$$  kubectl get pods --context=jwang-context
```
you can see right result of pod, however get service still forbidden
```
$ kubectl get svc --context=jwang-context
```
## create a sa(ServiceAccount)
```
$ kubectl create sa haimaxy-sa -n kube-system
```

## create sa role which is allowed to operate pod 
```
$ kubectl apply -f jwang-sa-role.yml
```

## bind sa user with role
```
$ kubectl create -f jwang-sa-rolebindingl.yml
```

## check sa secret
```
$ kubectl get secret -n kube-system |grep jwang-sa
haimay-sa-token-nxgqx                  kubernetes.io/service-account-token   3         47m
# get base64 token
$ kubectl get secret jwang-sa-token-nxgqx -o jsonpath={.data.token} -n kube-system |base64 -d
```
