# Cloud-Native CI/CD Template

Универсальный параметризованный CI/CD pipeline для веб-сервисов (frontend + backend) с использованием GitHub Actions, Docker, Kubernetes и Helm.

---

## 🚀 Обзор

Проект представляет собой шаблон DevOps-инфраструктуры, позволяющий автоматизировать:

- Сборку и тестирование
- Контейнеризацию (Docker)
- Деплой в Kubernetes (Helm)
- Деплой через SSH
- Откат и удаление релиза
- **Опциональную интеграцию PostgreSQL**

Основная цель — **стандартизация релизного процесса**.

---

## 🧠 Основные возможности

- Переиспользуемые GitHub Actions
- Параметризованный pipeline
- Helm-деплой в Kubernetes
- SSH-деплой (Docker)
- Поддержка rollback
- Поддержка dev/prod
- Многоуровневые values-файлы
- **Поддержка PostgreSQL для stateful приложений**
- Опциональный этап security-сканирования (DevSecOps)

---

## 🏗 Архитектура

### Репозиторий шаблонов (`cloud-native-cicd-template`)

Содержит:
- переиспользуемые workflows GitHub Actions
- универсальный Helm chart
- **Helm-шаблоны для PostgreSQL**

### Репозитории сервисов
Содержат:
- код
- Dockerfile
- специфичные для проекта Helm values
- легковесные обертки для workflows

---

## 🔄 CI/CD процесс

1. Push в main/develop
2. Сборка образа
3. Опциональное security-сканирование (по умолчанию не валит pipeline)
4. Публикация в registry
5. Деплой:
    - Kubernetes (Helm)
    - или SSH (Docker run)
6. Проверка
7. Rollback при необходимости

---

## 🛠 Включение PostgreSQL

Чтобы включить PostgreSQL в ваш проект:

1. Обновите файл `values.yaml`, чтобы включить PostgreSQL:

    ```
    postgresql:
      enabled: true
      username: your-username
      password: your-password
      database: your-database
    ```

2. Убедитесь, что шаблоны `postgresql` включены в ваш Helm chart:

    ```yaml
    # Пример: templates/postgresql-statefulset.yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgresql
    spec:
      ...
    ```

3. Разверните приложение с обновленным Helm chart:

    ```bash
    helm upgrade --install your-release-name ./helm/web-service -f values.yaml
    ```

4. Убедитесь, что StatefulSet и Service для PostgreSQL работают в вашем Kubernetes-кластере.

### Постоянные тома для PostgreSQL

Вы можете настроить постоянные тома с помощью Cloudfleet на Hetzner или Longhorn в любом Kubernetes-кластере, чтобы обеспечить сохранность данных для вашей базы данных PostgreSQL.
Вот [инструкция](https://cloudfleet.ai/tutorials/cloud/use-persistent-volumes-with-cloudfleet-on-hetzner/) по настройке Persistent Volume Claim:

---

## 🔧 Использование BUILD_ARGS при сборке

`BUILD_ARGS` используется для передачи параметров сборки в Dockerfile через pipeline. Это позволяет настраивать процесс сборки в зависимости от окружения или других переменных.

### Пример использования

1. Обновите ваш Dockerfile, чтобы использовать аргументы сборки:

    ```dockerfile
    ARG EXAMPLE_ARG
    ENV EXAMPLE_ENV=$EXAMPLE_ARG
    ```

2. Убедитесь, что в вашем pipeline передаются необходимые аргументы:

    ```yaml
    build-args: |
      ${{ inputs.build_args }}
      ${{ secrets.BUILD_ARGS }}
    ```
---

## 📁 Структура

```text
.github/workflows/
  deploy-web-service.yml
  rollback-web-service.yml
  uninstall-web-service.yml

helm/web-service/
  Chart.yaml
  values.yaml
  templates/
    _helpers.tpl
    configmap.yaml
    deployment.yaml
    hpa.yaml
    ingress.yaml
    secret.yaml
    service.yaml
    postgresql-configmap.yaml
    postgresql-secret.yaml
    postgresql-service.yaml
    postgresql-statefulset.yaml

example/
  backend-service/
  frontend-service/
```
