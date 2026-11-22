# Configuración de clúster Kubernetes (P2) — Grafana

Práctica individual. Archivos de apoyo para desplegar un clúster local de **3 nodos** con **kind**, y ejecutar un **Deployment de Grafana con 2 réplicas** y exposición por Service NodePort.

## Requisitos
- Docker Desktop o Docker Engine iniciado.
- `kind` y `kubectl` instalados.
- Puerto local TCP 30030 disponible, conexión para descargar las imágenes y recursos suficientes.

## Pasos de ejecución

```bash
# 1. Crear clúster local (1 control-plane y 2 workers)
kind create cluster --name practica-p2 --config kind-cluster.yaml

# 2. Verificar el clúster
kubectl get nodes -o wide

# 3. Preparar namespace y clave de administrador (NO compartir la clave)
kubectl create namespace monitoreo
kubectl -n monitoreo create secret generic grafana-admin --from-literal=password='REEMPLAZAR_POR_CLAVE_PROPIA'

# 4. Crear Deployment y Service
kubectl apply -f grafana.yaml
kubectl -n monitoreo rollout status deployment/grafana

# 5. Comprobar los objetos creados
kubectl -n monitoreo get deployments,replicasets,pods,services -o wide
kubectl -n monitoreo describe deployment grafana
```

Abre **http://localhost:30030**. Usuario `admin`; contraseña: la clave elegida en el paso 3. En kind, la opción `extraPortMappings` conecta el NodePort 30030 con el puerto 30030 del equipo anfitrión.

### Validación adicional de ReplicaSet
- Confirmar `READY 2/2` en el Deployment y dos Pods `Running`/`Ready`.
- Modificar el escalado para comprobar el controlador: `kubectl -n monitoreo scale deployment grafana --replicas=3`; comprobar `kubectl -n monitoreo get rs,pods -o wide`. Para regresar: `kubectl -n monitoreo scale deployment grafana --replicas=2`.
- Para probar autorecuperación, eliminar **un Pod** y volver a listar Pods: `kubectl -n monitoreo delete pod <nombre-del-pod>`; el ReplicaSet debe crear otro mientras el Deployment esté activo.

## Capturas solicitadas para el informe
1. `kind create cluster` y `kubectl get nodes -o wide`: tres nodos `Ready`.
2. Manifiestos `kind-cluster.yaml` y `grafana.yaml` con los ajustes importantes.
3. `kubectl -n monitoreo get deployments,replicasets,pods,services -o wide`: dos réplicas y NodePort 30030.
4. Navegador con Grafana abierto en `http://localhost:30030` (no incluir contraseñas).

## Publicación
Publica **estos archivos y este README** en un repositorio de GitHub configurado como **public**. Luego copia el enlace real al apartado «Repositorio público» del Word.

## Alcance y seguridad
Este despliegue es de **laboratorio**: la autenticación de Grafana se maneja con un Secret de Kubernetes, pero su estado local no se comparte entre Pods (SQLite por defecto y sin volumen persistente). No demuestra alta disponibilidad de la base de datos. Evitar exposición fuera de localhost; no subir secretos ni contraseñas a GitHub.

Fuentes: https://kind.sigs.k8s.io/docs/user/quick-start/ ; https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ ; https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/ ; https://kubernetes.io/docs/concepts/services-networking/service/
