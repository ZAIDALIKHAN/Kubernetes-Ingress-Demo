Flow of what actually is happening, Should View this File in Edit mode.

┌──────────────────────────────────────────────────────────────┐
│                        EC2 HOST MACHINE                       │
│                (Ubuntu on your actual EC2 instance)           │
│                                                              │
│   User Browser → http://EC2-PUBLIC-IP                        │
│                          │                                   │
│                          ▼                                   │
│                     HostPort: 80                             │
│                          │                                   │
│        (Port forwarded into Docker Container by Kind)        │
│                          │                                   │
└───────────────┬──────────────────────────────────────────────┘
                │  Kind ExtraPortMapping
                ▼
┌──────────────────────────────────────────────────────────────┐
│         DOCKER CONTAINER (THIS IS YOUR KIND NODE)            │
│                                                              │
│   Kind forwards:  hostPort 80  →  containerPort 30080        │
│                                                              │
│   ┌────────────────────────────────────────────────────────┐ │
│   │        Kubernetes (inside this same Docker container)  │ │
│   │                                                       │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │  NodePort Service (ingress-nginx-controller)     │ │ │
│   │  │      nodePort: 30080                             │ │ │
│   │  │          │                                       │ │ │
│   │  │          ▼                                       │ │ │
│   │  │  Ingress Controller Pod (nginx-ingress)          │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                 │                                      │ │
│   │                 ▼                                      │ │
│   │       Ingress Resource (just config, no pod)           │ │
│   │          - host: app.example.com                       │ │
│   │          - backend: svc-app1:80                        │ │
│   │                 │                                      │ │
│   │                 ▼                                      │ │
│   │  ┌──────────────────────────────────────────────────┐ │ │
│   │  │         ClusterIP Service (svc-app1)             │ │ │
│   │  │             port: 80                             │ │ │
│   │  │             targetPort: 80                        │ │ │
│   │  └──────────────────────────────────────────────────┘ │ │
│   │                 │                                      │ │
│   │                 ▼                                      │ │
│   │         App Deployment → App Pods                      │ │
│   │            containerPort: 80                           │ │
│   │                                                       │ │
│   └────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘




User → EC2:80 → Kind Docker Container:30080 → NodePort(30080) → Ingress Controller Pod → Ingress Rules → ClusterIP Service(svc-app1:80) → App Pod(containerPort 80)

Notes:

⭐ Why Ingress Controller service uses NodePort?
In Kubernetes:

IngressController must be reachable from outside cluster
→ so the controller's service = NodePort (or LoadBalancer if cloud)

⭐ Why App Service uses ClusterIP?
Because:

App should NOT be exposed outside cluster directly.

Ingress controller will internally reach app using "svc-app1" (ClusterIP).

⭐ Production Reality: Adding a New Service (= New App)

In a real company, when a new microservice is created, teams follow exactly the same steps:

1️⃣ Create Deployment

Every new service gets its own deployment.

Example:
payment-service-deployment.yaml
order-service-deployment.yaml
auth-service-deployment.yaml

This defines:

pods

replica count

container image

ports

resources

probes

✔ EXACTLY what you did for app1 & app2.

2️⃣ Create Service (ClusterIP)

Each app must have its own Service.

Example:
svc-payment
svc-orders
svc-auth

Why?

✔ Internal communication
✔ Load balancing across pods
✔ Stable virtual IP

✔ Same pattern you followed — svc-app1, svc-app2.
