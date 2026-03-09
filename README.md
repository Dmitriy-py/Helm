# Домашнее задание к занятию «Helm»

## ` Дмитрий Климов `

## Задание 1. Подготовить Helm-чарт для приложения

   1. Необходимо упаковать приложение в чарт для деплоя в разные окружения.
   2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
   3. В переменных чарта измените образ приложения для изменения версии.

## Ответ:

## Цель

### 1. Подготовка чарта

#### Для деплоя приложения был создан Helm-чарт `my-multi-app`. Структура чарта:

```
my-multi-app/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── backend-deployment.yaml
    └── frontend-deployment.yaml
```

## Код манифестов

### ` templates/frontend-deployment.yaml `

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: {{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}
```
### ` templates/backend-deployment.yaml `

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: {{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}
```

## 2. Установка приложения

### Для установки релиза выполнена команда: `helm install my-app` .





























