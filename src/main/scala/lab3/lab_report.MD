
University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4113c  
Author: Semisalov Daniil 
Lab: Lab3  
Date of create: 16.11.2023  
Date of finished: dd.mm.yyyy
---
# Сертификаты и "секреты" в Minikube, безопасное хранение данных
## Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
## Ход работы
### 1. Cоздание configMap с переменными: REACT_APP_USERNAME, REACT_APP_COMPANY_NAME
Переменные REACT_APP_USERNAME, REACT_APP_COMPANY_NAME передаются в Deployment
Создаем `ConfigMap` с переменными `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: lab3
  labels:
    app: lab3
data:
  react_app_username: "Semisalov Daniil"
  react_app_company_name: "ITMO"
```

### 2. Создание Deployment с 2 репликами контейнера и Service
Прописываем `envForm`, чтобы переменные были взяты из `ConfigMap`.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab3
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab3
  template:
    metadata:
      labels:
        app: lab3
    spec:
      containers:
        - name: itdt-contained-frontend
          image: ifilyaninitmo/itdt-contained-frontend:master
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: lab3
```

Создаем LoadBalancer

```yml
apiVersion: v1
kind: Service
metadata:
  name: lab3-service
  labels:
    app: lab3
spec:
  selector:
    app: lab3
  ports:
    - protocol: TCP
      port: 8200
      targetPort: 3000
  type: LoadBalancer
```

### 3. Генерация TLS сертификата
Сгенерируем TLS сертификат
```
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out cert.crt -keyout cert.key
```

![certificate](/pic/pic3)

Далее создаем секрет, добавляя сертификат в `minikube` с помощью команды:
```
kubectl create secret tls valeria-lab3-tls --key="cert.key" --cert="cert.crt"
```

### 4. Создание Ingress
Чтобы подключить `ingress`, необходимо выполнить команду:
```
minikube addons enable ingress
```

Далее необходимо написать манифест для `ingress`

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  labels:
    app: lab3
spec:
  tls:
    - hosts:
        - danya1234.com
      secretName: app-tls
  rules:
    - host: danya1234.com
      http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: lab3-service
                port:
                  number: 8200
```

### 5. Создание Secret
```yml
apiVersion: v1
kind: Secret
metadata:
  name: lab3-tls
type: kubernetes.io/tls
stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----

    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN PRIVATE KEY-----

    -----END PRIVATE KEY-----
```
Так же нужно написать 
```
kubectl create secret tls daniil-lab3-tls --key="cert.key" --cert="cert.crt"
```

### 6. Запуск написанного файла содержащего
Для применения всего написанного в манифесте выполним команду:
```
kubectl apply -f deployment.yaml
```

### 6. Перейдем в браузер
Для начала необходимо добавить запись в файл `hosts`, чтобы связать ip-адрес и доменное имя:
![hosts](/pic/pic4)

Подключаемся к Ingress командой с помощью команды:
```
minikube tunnel
```

Открываем страницу с адресом  `https://danya1234.com/`

![open_site](/pic/pic1)

### 7. Проверка наличия сертификата
Ниже представленны данные сертификата:

![view_certificate](/pic/pic2)

## Схема

![scheme](/pic/)