# 创建admin 账户

- step1: 创建admin 账户
```bash
vim admin-apiservercount.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

kubectl apply -f admin-apiservercount.yaml

kubectl get secrets -n kube-system
```

- step2: 创建 用户-角色绑定关系
```bash
vi admin-clusterrolebinding.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

kubectl apply -f admin-clusterrolebinding.yaml

kubectl get clusterrolebinding
  
```

- Step3: 查看token
```bash
kubectl describe secrets $(kubectl get secrets -n kube-system |grep admin-user |cut -f1 -d ' ') -n kube-system |grep -E '^token' |cut -f2 -d':'|tr -d '\t'|tr -d ' ' # 将输出 赋给 TOKEN 环境变量

kubectl config view |grep server|cut -f 2- -d ":" | tr -d " " # 将输出 赋给 APISERVER 变量
```

- Step4： 访问测试
``` bash
curl -H "Authorization: Bearer $TOKEN" $APISERVER/api  --insecure

```