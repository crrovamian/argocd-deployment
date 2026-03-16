# argocd-example

## Instalación de ArgoCD

### 1. Crear namespace para ArgoCD

```bash
kubectl create namespace argocd
```

### 2. Instalar ArgoCD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Obtener la contraseña inicial

La contraseña por defecto es el nombre del servidor (admin).

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 4. Acceder a la UI

#### Puerto local con kubectl port-forward

```bash
kubectl port-forward -n argocd svc/argocd-server 8080:443
```

Acceder a: http://localhost:8080

Usuario: `admin`
Contraseña: (la obtenida en el paso 3)

#### Alternativa: Cambiar contraseña

```bash
argocd login localhost:8080
argocd account update-password
```

## Estructura del repositorio

```
argocd-example/
├── apps/                          # Manifiestos de las aplicaciones
│   └── app/
│       ├── configmap.yaml         # Script que genera HTML dinámico
│       ├── deployment.yaml        # Deployment: example-app
│       ├── service.yaml           # Service: example-service
│       ├── ingress-traefik.yaml   # Ingress para Traefik
│       └── ingress-nginx.yaml     # Ingress para Nginx (comentado)
└── argocd/                        # Definiciones de Application ArgoCD
    └── example-app.yaml           # Application: example-app
```

## Flujo de GitOps

1. **Build**: CI construye imagen y la sube al registry
2. **Update**: Se actualiza la versión de la imagen en el repo Git
3. **Sync**: ArgoCD detecta el cambio y despliega al cluster

### Ejemplo: probar GitOps

El ejemplo usa imagen `nginx:1.25` con un HTML que muestra información del pod (hostname e IP). Cuando el pod se reinicia o se despliega una nueva versión, la información cambia automáticamente.

## Aplicar una Application

```bash
kubectl apply -f argocd/example-app.yaml
```

Esto se ejecuta **una sola vez**. ArgoCD se encarga automáticamente de detectar cambios en Git y sincronizar al cluster.

## Campos principales de una Application

| Campo | Descripción |
|-------|-------------|
| `repoURL` | URL del repositorio Git |
| `targetRevision` | Rama a monitorear (HEAD = main) |
| `path` | Ruta donde están los manifiestos YAML |
| `namespace` | Namespace donde se despliega |
| `prune` | Elimina recursos que ya no están en Git |
| `selfHeal` | Resincroniza si alguien cambia el cluster manualmente |

## Probar con Minikube y traefik en local

```bash
minikube tunnel
```

### Configurar hosts locales

Para acceder al servicio desde el navegador, añade el host a tu `/etc/hosts`:

```bash
echo "$(kubectl get svc traefik -o jsonpath='{.status.loadBalancer.ingress[0].ip}') local.example.com" | sudo tee -a /etc/hosts # Tener activado el tunnel y corriendo traefik
```

