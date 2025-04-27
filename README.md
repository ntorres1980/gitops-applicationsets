# GitOps con ArgoCD

Este repositorio configura un flujo de trabajo GitOps para desplegar microservicios en Kubernetes utilizando ArgoCD y Helm.

## Despliegue en dev y prod

Los microservicios `microservicio-a` y `microservicio-b` se desplegarán automáticamente en sus respectivos entornos de `dev` y `prod`.

### Para probar después del despliegue:

- Accede a `microservicio-a` en el entorno de desarrollo:
  ```bash
  curl http://microservicio-a.dev.example.com


# Ventajas de usar ApplicationSet

1. Automatización de múltiples aplicaciones
    - Podés generar automáticamente muchas aplicaciones a partir de una fuente común (como un Git repo o lista de clusters).
    - Evitás declarar cada Application manualmente.

2. Reducción de duplicación
    - Usás una plantilla (template) para definir cómo se crean las apps, y un generador (generator) para definir qué apps crear.
    - DRY (Don't Repeat Yourself): mismo código para muchas apps.

3. Actualización centralizada
    - Si cambiás la plantilla en el ApplicationSet, todas las apps se actualizan automáticamente.
    - Muy útil para mantener consistencia en configuraciones (como política de sincronización, target revision, etc).

4. Soporte para múltiples fuentes de generación
    - Podés generar aplicaciones desde:
    - Directorios en Git (git.directories)
    - Archivos (git.files)
    - Lista de clusters (cluster)
    - Lista de valores (list)
    - Desde una fuente externa vía plugin

5. Escalabilidad
    - Ideal para entornos con muchos microservicios o varios entornos (dev, qa, prod).
    - Más fácil de escalar que mantener cada Application por separado.

6. Dynamic discovery
    - Detecta automáticamente nuevas carpetas, archivos o clusters y genera apps nuevas sin que tengas que hacer cambios adicionales.

# ¿Qué hace este ApplicationSet?
    - Escanea todos los directorios bajo apps/*/values.yaml (apps/microservicio-a/values-dev.yaml, apps/microservicio-b/values-dev.yaml, etc).
    - Crea una Application para cada uno.
    - El nombre de la aplicación será el nombre del microservicio (por ejemplo microservicio-a).
    - El namespace en Kubernetes será el mismo nombre que el servicio.
    - Si no existe el namespace, ArgoCD lo crea automáticamente (CreateNamespace=true).
    - Se actualiza automáticamente si se agregan más servicios (carpetas nuevas) en Git.
    - Activa prune y selfHeal: elimina recursos viejos y corrige desviaciones en el clúster

# Instalar ArgoCD

## 1. Instalar Helm (si no lo tenés)

Primero, instalamos Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

```
Verificamos:

```bash
helm version
```

## 2. Agregar el repo de charts de ArgoCD
Ahora agregamos el repositorio oficial de Helm de Argo:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```
## 3. Crear el namespace argocd
Igual que antes:

```bash
kubectl create namespace argocd
```

## 4. Instalar ArgoCD con Helm
Instalamos el chart de argo-cd:

```bash
helm install argocd argo/argo-cd -n argocd
```
Esto:

- Despliega todos los componentes de ArgoCD
- Los organiza en el namespace argocd
- Usa los valores por defecto (que después podemos sobreescribir con un values.yaml si querés).

Verificamos:

```bash
kubectl get pods -n argocd
```
Deberías ver todos los pods arrancando (argocd-server, argocd-repo-server, argocd-application-controller, etc.).

### 4.1 Salida de la instalación
```
NAME: argocd
LAST DEPLOYED: Sun Apr 27 12:14:28 2025
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-1-ssl-passthrough
      - Set the `configs.params."server.insecure"` in the values file and terminate SSL at your ingress: https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)
```

# Instalar el NGINX Ingress Controller

## 1. Agregar el repositorio de Helm de NGINX

Primero, necesitas agregar el repositorio de Helm para el NGINX Ingress Controller.

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
# 2. Actualizar el repositorio de Helm
Luego, actualiza tu lista de charts de Helm:

```bash
helm repo update
```
# 3. Instalar el Ingress Controller
Ahora puedes instalar el NGINX Ingress Controller usando Helm. Como estás utilizando Kind, puedes especificar el espacio de nombres ingress-nginx y otros parámetros si lo deseas. El comando básico es:

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```
Este comando hará lo siguiente:

Instalará el Ingress Controller en el espacio de nombres ingress-nginx.

Si el espacio de nombres no existe, lo creará.

# 4. Verificar que el Ingress Controller está corriendo
Después de instalarlo, puedes verificar que el controlador de Ingress está corriendo con el siguiente comando:

```bash
kubectl get pods -n ingress-nginx
```
También puedes verificar los servicios del Ingress Controller:

```bash
kubectl get svc -n ingress-nginx
```
Si estás utilizando Kind, los servicios típicamente serán de tipo NodePort, por lo que necesitarás acceder a través del puerto asignado.