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
3. Публикация в registry
4. Деплой:
    - Kubernetes (Helm)
    - или SSH (Docker run)
5. Проверка
6. Rollback при необходимости

---

## 🛠 Включение PostgreSQL

Чтобы включить PostgreSQL в ваш проект:

1. Обновите файл `values.yaml`, чтобы включить PostgreSQL:

    ```yaml
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

При сборке Docker-образов для фронтенда вы можете использовать переменную окружения `BUILD_ARGS`, чтобы передать свои пользовательские параметры. Это позволяет гибко настраивать процесс сборки в зависимости от ваших требований.

### Пример использования

1. Добавьте необходимые аргументы сборки в переменную `BUILD_ARGS` в вашем CI/CD pipeline или локально:

    ```bash
    export BUILD_ARGS="API_URL=https://api.example.com NODE_ENV=production"
    ```

2. Убедитесь, что ваш `Dockerfile` поддерживает эти аргументы. Например:

    ```dockerfile
    ARG API_URL
    ARG NODE_ENV

    ENV REACT_APP_API_URL=$API_URL
    ENV NODE_ENV=$NODE_ENV

    RUN npm run build
    ```

3. При сборке образа аргументы будут переданы в контейнер:

    ```bash
    docker build --build-arg API_URL=https://api.example.com --build-arg NODE_ENV=production -t frontend:latest .
    ```

4. В вашем CI/CD pipeline используйте секреты или переменные окружения для передачи значений в `BUILD_ARGS`:

    ```yaml
    build-args: |
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
