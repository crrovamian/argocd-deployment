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
│   └── billing-service/
│       ├── configmap.yaml         # HTML con el mensaje
│       ├── deployment.yaml
│       ├── service.yaml
│       └── ingress.yaml
└── argocd/                        # Definiciones de Application ArgoCD
    └── billing-service-app.yaml
```

## Flujo de GitOps

1. **Build**: CI construye imagen y la sube al registry
2. **Update**: Se actualiza la versión de la imagen en el repo Git
3. **Sync**: ArgoCD detecta el cambio y despliega al cluster

### Ejemplo: actualizar mensaje para probar GitOps

El ejemplo usa imagen `nginx:1.25` con un HTML que muestra un mensaje. Para probar el flujo de GitOps, solo cambia el mensaje en `apps/billing-service/configmap.yaml`:

```yaml
data:
  index.html: |
    <h1>Billing Service - Hola mundo v1</h1>  # Cambia este valor
```

Cambia a `Hola mundo v2`, haz commit y push → ArgoCD sincroniza automáticamente → actualiza el HTML del pod.

## Aplicar una Application

```bash
kubectl apply -f argocd/billing-service-app.yaml
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

## Configurar hosts locales

Para acceder al servicio desde el navegador, añade el host a tu `/etc/hosts`:

```bash
echo "127.0.0.1 billing.example.com" | sudo tee -a /etc/hosts
```

Esto te permitirá acceder a la aplicación en `http://billing.example.com` (el puerto del Ingress/Traefik, usualmente 80).