INSTRUCTION.md
# How to validate RBAC (list secrets via curl from the Pod)

## 0) Pre-req
- kind cluster from `cluster.yml`
- Manifests applied: `./bootstrap.sh`

## 1) Get the pod name
```bash
kubectl get pods -n default -l app=todo-app

2) Exec into the pod (install curl if needed)
# якщо curl немає в образі:
kubectl exec -n default -it <POD_NAME> -- sh -lc 'apt-get update && apt-get install -y curl || (apk add --no-cache curl || yum install -y curl || true)'

3) Call K8s API with SA token
kubectl exec -n default -it <POD_NAME> -- sh -lc '
API="https://kubernetes.default.svc"
TOKEN="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
CACERT="/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
NAMESPACE="$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)"
curl --silent --show-error --cacert "$CACERT" \
  -H "Authorization: Bearer $TOKEN" \
  "$API/api/v1/namespaces/$NAMESPACE/secrets?limit=500"
'


Очікувано: JSON з "kind": "SecretList" і масивом "items".

4) Screenshot

Зроби скрін терміналу з результатом curl і прикріпи до PR.

Notes

Role дозволяє list (і get) секрети в default.

RoleBinding прив’язує роль до todo-sa.

Deployment використовує serviceAccountName: todo-sa.