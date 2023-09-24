# RBAC
Role-based access control (RBAC) is a way of granting users granular access to Kubernetes API resources. RBAC is a security design that restricts access to Kubernetes resources based on the role the user holds.

API Objects for configuring RBAC: `Role`, `ClusterRole`, `RoleBinding` and `ClusterRoleBinding`.
`Role/ClusterRole` only say what can be done, while who can do what is defined in a `RoleBinding/ClusterRoleBinding`.
```
kubectl get ns -A
```
To see all the present namespaces in the environment.
Now we will create a new namespace named `test`.
```
kubectl create ns test
vi serviceaccount.yml
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: foo
  namespace: test
```
```
kubectl apply -f serviceaccount.yml
```
Post creating the namespace and serviceaccount, we will use the below command to check access of the account.
```bash
kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test
```
The output is `no` as we have only created the ns and account but have not configured it yet and no roles are associated with it.

We will create role associated with the ns and the service account.

## Role

Role defines what can be done to Kubernetes Resources.
Role contains one or more rules that represent a set of permissions.
Permissions are additive. There are no deny rules.
Roles are namespaced, meaning Roles work within the constraints of a namespace. It would default to the default namespace if none was specified.
After creating a Role, you assign it to a user or group of users by creating a RoleBinding.

* apiGroups- core api access
* resources - all resources like pods, deployments, rs, rc, ds
* verbs - all verbs as in create, delete, apply, edit
```bash
vi role.yml
```
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
```bash
kubectl apply -f role.yml 
kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test
```
still no as the role is not binded yet.

## RoleBinding

Role Binding is used for granting permission to a Subject.
RoleBinding holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted.
Role and RoleBinding are used in namespaced scoped.
RoleBinding may reference any Role in the same namespace.
After you create a binding, you cannot change the Role or ClusterRole that it refers to. If you do want to change the roleRef for a binding, you need to remove the binding object and create a replacement.
```bash
vi rolebinding.yml
```
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
```bash
kubectl apply -f rolebinding.yml 

kubectl auth can-i --as system:serviceaccount:test:foo get pods -n test
```
Now the answer is yes, and the newly created service account has a role now and can perform operations in k8s.
```bash
kubectl auth can-i --as system:serviceaccount:test:foo create pods -n test

kubectl auth can-i --as system:serviceaccount:test:foo delete pods -n test

kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n test
```
But what if we change the namespace.
```bash
kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n kube-system
```
It is failing as we assiged a role and binded it to only test namespace, but we did not assign it to any cluster role and cluster role binding.

## ClusterRole

ClusterRole works the same as Role, but they are applied to the cluster as a whole.
ClusterRoles are not bound to a specific namespace. ClusterRole give access across more than one namespace or all namespaces.
After creating a ClusterRole, you assign it to a user or group of users by creating a RoleBinding or ClusterRoleBinding.
ClusterRoles are typically used with service accounts.
Because ClusterRoles are cluster-scoped, you can use ClusterRoles to control access to different kinds of resources than you can with Roles.

Cluster-scoped resources (e.g. Nodes, PersistentVolumes)
Non-resource endpoints (e.g./healthz).
Namespaced resources (e.g. Pods across the entire cluster), across all namespaces.
Default ClusterRole:

* cluster-admin: Cluster wide super user.
* admin: Full access within a Namespace.
* edit: Read/write within a Namespace.
* view: Read-only within a Namespace.


## ClusterRoleBinding

ClusterRole and ClusterRoleBinding function like Role and RoleBinding, except they have wider scope.
RoleBinding grants permissions within a specific namespace, whereas a ClusterRoleBinding grants access cluster-wide and to multiple namespaces.
ClusterRoleBinding is binding or associating a ClusterRole with a Subject (users, groups, or service accounts).
```bash
vi clusterrolebinding.yml
```
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
```bash
kubectl auth can-i --as system:serviceaccount:test:foo create deployments -n kube-system
kubectl auth can-i --as system:serviceaccount:test:foo delete deployments -n kube-system
kubectl auth can-i --as system:serviceaccount:test:foo delete deployments -n default
```
