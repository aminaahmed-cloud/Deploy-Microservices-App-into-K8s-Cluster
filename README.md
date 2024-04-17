# Deploy Microservices App into K8s Cluster

## Step by Step Deployment Process

### Key Information you need to know:
- What microservices are you deploying?
- How are they connected?
- Any 3rd party services or databases?
- Which service is accessible from outside the cluster?
- What environment variables each microservice expects?
- On which port each microservice starts?
- Developer teams are working on the service so we need to deploy Microservices into 1 Namespace.

---

### Steps:

1. **Create Deployment Config for each Microservice**
2. **Create Service Config for each Microservice**
3. **Deploy Microservices application on K8s cluster**

---

### 1. Create Deployment & Service Configurations

Note that we need to provide volume for the redis container as Redis persists its data in memory, we’ll use volume emptyDir:
- Is initially empty
- First created when a pod is assigned to a Node
- Exists as long as Pod is running
- Container crashing does not remove a Pod from a Node
- Therefore, data safe across container crashes!

Create the file `config.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: emailservice
spec:
  selector:
    matchLabels:
      app: emailservice
  template:
    metadata:
      labels:
        app: emailservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/emailservice:v0.8.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        livenessProbe:
          grpc:
            port: 8080
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 8080
          periodSeconds: 5
        resources:
          requests: 
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: emailservice
spec:
  type: ClusterIP
  selector:
    app: emailservice
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommendationservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: recommendationservice
  template:
    metadata:
      labels:
        app: recommendationservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/recommendationservice:v0.8.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
        - name: DISABLE_PROFILER
          value: "1"
        livenessProbe:
          grpc:
            port: 8080
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 8080
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: recommendationservice
spec:
  type: ClusterIP
  selector:
    app: recommendationservice
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productcatalogservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: productcatalogservice
  template:
    metadata:
      labels:
        app: productcatalogservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.8.0
        ports:
        - containerPort: 3550
        env:
        - name: PORT
          value: "3550"
        - name: DISABLE_PROFILER
          value: "1"
        livenessProbe:
          grpc:
            port: 3550
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 3550
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: productcatalogservice
spec:
  type: ClusterIP
  selector:
    app: productcatalogservice
  ports:
  - protocol: TCP
    port: 3550
    targetPort: 3550

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: paymentservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: paymentservice
  template:
    metadata:
      labels:
        app: paymentservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/paymentservice:v0.8.0
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
        - name: DISABLE_PROFILER
          value: "1"
        livenessProbe:
          grpc:
            port: 50051
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 50051
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: paymentservice
spec:
  type: ClusterIP
  selector:
    app: paymentservice
  ports:
  - protocol: TCP
    port: 50051
    targetPort: 50051

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: currencyservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: currencyservice
  template:
    metadata:
      labels:
        app: currencyservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/currencyservice:v0.8.0
        ports:
        - containerPort: 7000
        env:
        - name: PORT
          value: "7000"
        - name: DISABLE_PROFILER
          value: "1"
        livenessProbe:
          grpc:
            port: 7000
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 7000
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: currencyservice
spec:
  type: ClusterIP
  selector:
    app: currencyservice
  ports:
  - protocol: TCP
    port: 7000
    targetPort: 7000

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shippingservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: shippingservice
  template:
    metadata:
      labels:
        app: shippingservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/shippingservice:v0.8.0
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
        livenessProbe:
          grpc:
            port: 50051
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 50051
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: shippingservice
spec:
  type: ClusterIP
  selector:
    app: shippingservice
  ports:
  - protocol: TCP
    port: 50051
    targetPort: 50051

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: adservice
  template:
    metadata:
      labels:
        app: adservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/shippingservice:v0.8.0
        ports:
        - containerPort: 9555
        env:
        - name: PORT
          value: "9555"
        livenessProbe:
          grpc:
            port: 9555
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 9555
          periodSeconds: 5
        resources:
          requests: 
            cpu: 200m
            memory: 180Mi
          limits:
            cpu: 300m
            memory: 300Mi

---
apiVersion: v1
kind: Service
metadata:
  name: adservice
spec:
  type: ClusterIP
  selector:
    app: adservice
  ports:
  - protocol: TCP
    port: 9555
    targetPort: 9555

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cartservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cartservice
  template:
    metadata:
      labels:
        app: cartservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/cartservice:v0.8.0
        ports:
        - containerPort: 7070
        env:
        - name: PORT
          value: "7070"
        - name: REDIS_ADDR
          value: "redis-cart:6379"
        - name: DISABLE_PROFILER
          value: "1"
        livenessProbe:
          grpc:
            port: 7070
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 7070
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: cartservice
spec:
  type: ClusterIP
  selector:
    app: cartservice
  ports:
  - protocol: TCP
    port: 7070
    targetPort: 7070

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cart
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-cart
  template:
    metadata:
      labels:
        app: redis-cart
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        livenessProbe:
          initialDelaySeconds: 5
          tcpSocket:
            port: 6379
          periodSeconds: 5
        readinessProbe:
          initialDelaySeconds: 5
          tcpSocket:
            port: 6379
          periodSeconds: 5
        resources:
          requests: 
            cpu: 70m
            memory: 200Mi
          limits:
            cpu: 125m
            memory: 300Mi
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: redis-cart
spec:
  type: ClusterIP
  selector:
    app: redis-cart
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkoutservice
spec:
  replicas: 2
  selector:
    matchLabels:
      app: checkoutservice
  template:
    metadata:
      labels:
        app: checkoutservice
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/checkoutservice:v0.8.0
        ports:
        - containerPort: 5050
        env:
        - name: PORT
          value: "5050"
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
        - name: SHIPPING_SERVICE_ADDR
          value: "shippingservice:50051"
        - name: PAYMENT_SERVICE_ADDR
          value: "paymentservice:50051"
        - name: EMAIL_SERVICE_ADDR
          value: "emailservice:5000"
        - name: CURRENCY_SERVICE_ADDR
          value: "currencyservice:7000"
        - name: CART_SERVICE_ADDR
          value: "cartservice:7070"
        livenessProbe:
          grpc:
            port: 5050
          periodSeconds: 5
        readinessProbe:
          grpc:
            port: 5050
          periodSeconds: 5
      
---
apiVersion: v1
kind: Service
metadata:
  name: checkoutservice
spec:
  type: ClusterIP
  selector:
    app: checkoutservice
  ports:
  - protocol: TCP
    port: 5050
    targetPort: 5050

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: service
        image: gcr.io/google-samples/microservices-demo/frontend:v0.8.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
        - name: CURRENCY_SERVICE_ADDR
          value: "currencyservice:7000"
        - name: CART_SERVICE_ADDR
          value: "cartservice:7070"
        - name: RECOMMENDATION_SERVICE_ADDR
          value: "recommendationservice:8080"
        - name: SHIPPING_SERVICE_ADDR
          value: "shippingservice:50051"
        - name: CHECKOUT_SERVICE_ADDR
          value: "checkoutservice:5050"
        - name: AD_SERVICE_ADDR
          value: "adservice:955"
        livenessProbe:
          httpGet:
            path: "/_healthz"
            port: 8080
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: "/_healthz"
            port: 8080
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 3. Deploy Microservices on K8s Cluster:
   
**- Create Cluster on Linode**

<img src="https://i.imgur.com/nlonfn0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

- For security practices, we need to set permissions for only my user to use the kubeconfig file:

```bash
% chmod 400 online-shop-microservices-kubeconfig.yaml
```

- Set the kubeconfig file:

```bash
% export KUBECONFIG=/Users/ahmed/k8s-configuration/online-shop-microservices-kubeconfig.yaml
```

- Verify the connection:

```bash
% kubectl get node
```

- Create a namespace for microservices:

```bash
% kubectl create ns microservices
```

- Apply the configurations:

```bash
% kubectl apply -f config.yaml -n microservices
```

<img src="https://i.imgur.com/zq5bv6G.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
