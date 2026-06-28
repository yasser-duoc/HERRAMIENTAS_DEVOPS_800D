# Innovatech Chile - Plataforma de Gestión de Alimentos 🐶



## 🛠️ Arquitectura del Repositorio

A continuación se detalla la estructura de directorios del proyecto, organizando de forma independiente cada componente de la aplicación y sus respectivos manifiestos de configuración:

*   **`.github/workflows/`**: Contiene el pipeline automatizado (`deploy.yml`) para la integración y entrega continua.
*   **`backend/`**: Código fuente de la API desarrollado en Node.js (`server.js`), empaquetado mediante un `Dockerfile`.
*   **`db/`**: Configuración de la base de datos relacional (MySQL) junto con el script de inicialización (`init.sql`).
*   **`frontend/`**: Interfaz de usuario web optimizada con Nginx (`default.conf`, `index.html`).
*   **`k8s/`**: Manifiestos declarativos de Kubernetes para la gestión de despliegues, redes, secretos y autoescalado.

---

## 🚀 Tecnologías Utilizadas

*   **Orquestador:** Amazon EKS (Elastic Kubernetes Service)
*   **CI/CD Pipeline:** GitHub Actions
*   **Registro de Imágenes:** Amazon ECR (Elastic Container Registry)
*   **Servidores & Entornos:** Node.js, Nginx, MySQL, Docker
*   **Métricas de Escalado:** Kubernetes HPA (Horizontal Pod Autoscaler)

---

## 📂 Componentes e Infraestructura de Código

### 1. Dockerización (Microservicios)
Cada componente posee su propio ciclo de vida y empaquetado aislado:
*   **Backend:** Construido a partir de `backend/Dockerfile`, expone los endpoints de la lógica de negocio.
*   **Frontend:** Construido a partir de `frontend/Dockerfile`, montado sobre Nginx para servir los recursos estáticos de manera eficiente.
*   **Base de Datos:** Configurada con `db/Dockerfile`, carga automáticamente el esquema inicial `init.sql` al arrancar.

### 2. Orquestación y Manifiestos de Kubernetes (`k8s/`)
El despliegue dentro de Amazon EKS se maneja de manera declarativa a través de los siguientes archivos:
*   `namespace.yaml`: Aísla los recursos del proyecto bajo el espacio de nombres lógico `tienda`.
*   `mysql-secret.yaml`: Almacena y cifra de forma segura las credenciales de acceso a la base de datos.
*   `mysql-deployment.yaml` & `mysql-service.yaml`: Despliega la base de datos y expone su puerto interno para el backend.
*   `backend-deployment.yaml` & `backend-service.yaml`: Levanta los pods de la API e implementa la resolución de nombres DNS.
*   `frontend-deployment.yaml` & `frontend-service.yaml`: Pone en marcha la interfaz y la expone mediante un LoadBalancer (ALB de AWS) con acceso público.
*   `backend-hpa.yaml` & `frontend-hpa.yaml`: Define las reglas de Horizontal Pod Autoscaler (escalamiento reactivo basado en uso de CPU).

---

## 🔄 Pipeline CI/CD (`.github/workflows/deploy.yml`)

El flujo automatizado se dispara ante cada evento de `push` en la rama `main` y realiza los siguientes pasos de manera secuencial:

1.  **Checkout Code:** Clona el código fuente en el runner de GitHub.
2.  **Configure AWS Credentials:** Autentica de forma segura con AWS Academy utilizando secretos cifrados.
3.  **Login to Amazon ECR:** Establece conexión con el registro de contenedores privado de AWS.
4.  **Build and Push Docker Images:** Construye las imágenes de Docker para Front, Back y DB, etiquetándolas y subiéndolas a ECR.
5.  **Connect to EKS Cluster:** Configura el contexto local de `kubectl` apuntando al endpoint activo en AWS.
6.  **Apply Kubernetes Manifests:** Actualiza dinámicamente los despliegues en el clúster.
7.  **Verify Rollout Status:** Monitorea los Pods hasta asegurar que se encuentren en estado *Running*.

---

## 🔧 Configuración de Variables y Secretos

Para que el pipeline opere de manera correcta sin exponer datos sensibles, se deben configurar los siguientes **GitHub Secrets** en el repositorio:

*   `AWS_ACCESS_KEY_ID`: Llave de acceso de la cuenta de AWS Academy.
*   `AWS_SECRET_ACCESS_KEY`: Llave secreta de la cuenta de AWS.
*   `AWS_SESSION_TOKEN`: Token de sesión temporal de AWS Academy.
*   `AWS_REGION`: Región de AWS donde se encuentra el clúster (ej. `us-east-1`).

---

## 💻 Instrucciones de Despliegue Local / Manual

En caso de requerir un despliegue manual o pruebas de diagnóstico, ejecute el script provisto en la raíz:

```bash
# Otorgar permisos de ejecución al script
chmod +x deploy.sh

# Ejecutar el flujo de despliegue en el clúster local/remoto
./deploy.sh
