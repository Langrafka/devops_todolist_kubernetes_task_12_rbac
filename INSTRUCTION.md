# INSTRUCTION.md: Інструкції для перевірки RBAC (list secrets)

Цей Pull Request додає конфігурацію **RBAC**, яка надає Deployment "todoapp" мінімальні дозволи для доступу до ресурсів **Secrets** у Namespace "todoapp" (принцип Least Privileged).

Для цього використовується:
* **ServiceAccount:** `secrets-reader`
* **Role:** `secrets-lister-role` (дозволяє `list secrets`)
* **RoleBinding:** Зв'язує `secrets-reader` з `secrets-lister-role`.

---

## 🛠️ Перевірка коректності RBAC-налаштування

Щоб перевірити, що Service Account має права `list` на `secrets`, виконайте наступні кроки в оболонці:

1.  **Запустіть кластер** та **застосуйте всі маніфести** (включно з `security/rbac.yml` та оновленим `deployment.yml`):
    ```bash
    kubectl apply -f .
    ```

2.  **Знайдіть назву працюючого Pod** та **підключіться до його оболонки** (`sh`):
    ```bash
    POD_NAME=$(kubectl get pods -n todoapp -l app=todoapp -o jsonpath='{.items[0].metadata.name}')
    kubectl exec $POD_NAME -it -n todoapp -- sh
    ```

3.  **Виконайте команду `curl`** всередині Pod для перевірки доступу до API (переконайтеся, що ви знаходитеся в оболонці Pod, `#`):
    ```sh
    # Встановлення змінних Service Account
    APISERVER=[https://kubernetes.default.svc](https://kubernetes.default.svc) # <-- ВИПРАВЛЕНО: Видалено Markdown-форматування
    SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
    TOKEN=$(cat ${SERVICEACCOUNT}/token)
    CACERT=${SERVICEACCOUNT}/ca.crt

    # Запит на список Secrets у Namespace 'todoapp'.
    # Цей запит повинен завершитися успіхом (код 200 OK), підтверджуючи права RBAC.
    curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/todoapp/secrets
    ```
