RBAC


kubectl get ns -A
to see all the present namespaces in the environment


kubectl create ns test
```yaml
vi serviceaccount.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: foo
  namespace: test
```
kubectl apply -f serviceaccount.yml

this command can be used to check access of any account 
kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test

the output is no as we have only created the ns and account but have not configured it yet and no roles associated with it

We will create role associated withe the ns

apiGroups- core api access
resources - all resources like pods, deployments, rs, rc, ds
verbs - all verbs as in create, delete, apply, edit
vi role.yml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test
  name: testadmin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```
kubectl apply -f role.yml 
kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test

still no

now a role has been created but it is not binded with service account
vi rolebinding.yml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: testadminbinding
  namespace: test
subjects:
- kind: ServiceAccount
  name: foo
  apiGroup: ""
roleRef:
  kind: Role
  name: testadmin
  apiGroup: ""
```
kubectl apply -f rolebinding.yml 

kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test
now the answer is yes, and the newly created service account has a role now and can perform actions in k8s

kubectl auth can-i --as system:serviceaccount:test:foo create pods -n test

kubectl auth can-i --as system:serviceaccount:test:foo delete pods -n test

kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n test

what if we change the namespace
kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n kube-system

it is failing as we assiged a role and binded it but we did not assign it to any cluster role and cluster role binding

vi clusterrolebinding.yml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: testadminclusterbinding
subjects:
- kind: ServiceAccount
  name: foo
  apiGroup: ""
  namespace: test
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
```

kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n kube-system
kubectl auth can-i --as system:serviceaccount:test:foo delete deployments -n kube-system
kubectl auth can-i --as system:serviceaccount:test:foo delete deployments -n default
