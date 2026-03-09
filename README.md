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

<img width="1920" height="1080" alt="Снимок экрана (2916)" src="https://github.com/user-attachments/assets/3723a23f-ab3c-4b16-9386-cab7507cc438" />

## 3. Обновление версии приложения

### Для изменения версии образа фронтенда была выполнена команда обновления:

```bash
helm upgrade my-app . --set frontend.image.tag="1.27.0"
```

<img width="1920" height="1080" alt="Снимок экрана (2918)" src="https://github.com/user-attachments/assets/8b4a7fc5-0c43-4291-828f-7eea6f0a8b4f" />

## 4. История релизов

```bash
helm history my-app
```

<img width="1920" height="1080" alt="Снимок экрана (2919)" src="https://github.com/user-attachments/assets/6afb061b-a662-4262-86c8-07a298d9beb8" />


## Задание 2. Запустить две версии в разных неймспейсах

   1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
   2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
   3. Продемонстрируйте результат.

## Ответ:

## 1. Создание неймспейсов

```bash
kubectl create namespace app1
kubectl create namespace app2
```

<img width="1920" height="1080" alt="Снимок экрана (2920)" src="https://github.com/user-attachments/assets/20cfb883-c05f-46bd-a226-db8d75c9bc34" />

## 2. Деплой приложений

### Были выполнены команды установки трех независимых релизов с переопределением параметров:

   * Релиз 1 (Базовый) в app1: `helm install app1-v1 ./my-multi-app --namespace app1 --create-namespace`
   * Релиз 2 (с измененным тегом frontend) в app1: `helm install app1-v2 ./my-multi-app --namespace app1 --set                    frontend.image.tag="1.27.0"`
   * Релиз 3 (с измененным тегом backend) в app2: `helm install app2-prod ./my-multi-app --namespace app2 --create-               namespace      --set backend.image.tag="0.5.0"`

<img width="1920" height="1080" alt="Снимок экрана (2922)" src="https://github.com/user-attachments/assets/efe52efb-b279-43fc-a9f7-0120a7700074" />

## 3. Результаты проверки

### Проверка неймспейса `app1`:
```bash
kubectl get pods -n app1
```
### Проверка неймспейса `app2`:
```bash
kubectl get pods -n app2
```
### Список всех релизов Helm:
```bash
helm list -A
```

<img width="1920" height="1080" alt="Снимок экрана (2923)" src="https://github.com/user-attachments/assets/7bd7fc88-95ce-4f08-b2e5-a8466f326fca" />

## Вывод

### Результаты показывают, что использование уникальных имен для ресурсов (через `{{ .Release.Name }}-...)` позволяет запускать несколько релизов одного чарта в одном неймспейсе без конфликтов имен, а использование --namespace обеспечивает логическую изоляцию ресурсов в кластере.

## Манифесты:

### `templates/backend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-backend
  labels:
    app: backend
    instance: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
      instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: backend
        instance: {{ .Release.Name }}
    spec:
      containers:
      - name: backend
        image: {{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}
        ports:
        - containerPort: 80
```

### `templates/frontend-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-frontend
  labels:
    app: frontend
    instance: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
      instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: frontend
        instance: {{ .Release.Name }}
    spec:
      containers:
      - name: frontend
        image: {{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}
        ports:
        - containerPort: 80
```










