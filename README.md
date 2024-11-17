Descripción
Este proyecto tiene como objetivo mejorar una aplicación FastAPI implementando un pipeline de CI/CD utilizando GitHub Actions, y desplegando la aplicación en un clúster de Kubernetes. Además, se ha configurado monitorización con Prometheus y Grafana para visualizar métricas y el rendimiento de la aplicación. Se han realizado varias modificaciones y configuraciones para asegurar un despliegue eficiente y una monitorización efectiva.

Tabla de Contenidos
Estructura del Proyecto
Requisitos Previos
Configuración del Entorno
Pipeline de CI/CD con GitHub Actions
Despliegue en Kubernetes
Instalación de Helm
Despliegue de Prometheus y Grafana
Configuración de Dashboards en Grafana
Monitorización con Prometheus y Grafana
Reproducción del Proyecto
Recursos Adicionales

proyectosre/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── kube-prometheus-stack/
│   ├── values.yaml
├── src/
│   ├── app.py
│   └── requirements.txt
├── Dockerfile
├── helm-chart/
│   ├── Chart.yaml
│   ├── templates/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── ...
├── README.md
└custom-dashboard.json

Requisitos Previos
Antes de comenzar, asegúrate de tener instalados los siguientes componentes:

Docker: Para construir imágenes de contenedores.
Kubernetes: Configurado con acceso a un clúster (puede ser Minikube, Docker Desktop, o un clúster en la nube).
Helm: Para gestionar paquetes de Kubernetes.
kubectl: Herramienta de línea de comandos para Kubernetes.
GitHub Account: Para alojar el repositorio y configurar GitHub Actions.
Grafana y Prometheus: Para monitorización (se despliegan mediante Helm).

Configuración del Entorno
Clonar el Repositorio:

Configuración del Entorno
Clonar el Repositorio:

bash
git clone https://github.com/tu-usuario/proyectosre.git
cd proyectosre

Configurar Variables de Entorno:
DOCKER_USERNAME=tu_usuario
DOCKER_PASSWORD=tu_contraseña
KUBE_NAMESPACE=monitoring

Pipeline de CI/CD con GitHub Actions
Se han configurado dos workflows de GitHub Actions: uno para CI (pruebas) y otro para CD (despliegue).

1. Workflow de CI
2. Workflow de CD (.github/workflows/cd.yml):

Notas:

Asegúrate de agregar tus credenciales de DockerHub en los Secrets de GitHub.
Ajusta los paths y nombres de archivos según tu estructura de proyecto.
Despliegue en Kubernetes
Instalación de Helm
Instalar Helm:

Si aún no tienes Helm instalado, sigue las instrucciones en Helm Installation Guide.

Agregar Repositorio de Prometheus Community:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

Despliegue de Prometheus y Grafana
Crear Namespace monitoring:

kubectl create namespace monitoring

Desplegar kube-prometheus-stack:
helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring -f kube-prometheus-stack/values.yaml

Archivo de Valores (kube-prometheus-stack/values.yaml):

Asegúrate de personalizar este archivo según tus necesidades. Aquí hay un ejemplo básico:

grafana:
  adminPassword: "admin-password"
  sidecar:
    dashboards:
      enabled: true
      label: grafana_dashboard
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
        - name: 'default'
          orgId: 1
          folder: ''
          type: file
          disableDeletion: false
          updateIntervalSeconds: 10
          options:
            path: /var/lib/grafana/dashboards/default

Configuración de Dashboards en Grafana
Crear ConfigMap para Dashboards Personalizados:            
Aplicar el ConfigMap:
kubectl apply -f kube-prometheus-stack/prometheus-stack-grafana-dashboards-custom.yaml
Verificar que el ConfigMap está montado correctamente:
kubectl get configmap prometheus-stack-grafana-dashboards-custom -n monitoring -o yaml

Revisar los Logs de Grafana:

Para asegurarte de que Grafana está cargando el dashboard correctamente, revisa los logs del pod de Grafana.
kubectl logs deployment/prometheus-stack-grafana -n monitoring

Monitorización con Prometheus y Grafana
Acceso a Grafana
Obtener el Servicio de Grafana:

bash
Copiar código
kubectl get svc -n monitoring
Busca el servicio de Grafana, típicamente llamado prometheus-stack-grafana. Si estás usando Minikube o tienes configurado un Ingress, ajusta el acceso en consecuencia.

Acceder a Grafana:

Si estás usando port-forward:

bash
Copiar código
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring
Luego, accede a http://localhost:3000 en tu navegador.

Iniciar Sesión en Grafana:

Usa las credenciales de administrador que configuraste. Si no las cambiaste, pueden estar en el secreto de Kubernetes.

bash
Copiar código
kubectl get secret prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-user}" | base64 --decode
kubectl get secret prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
Verificar Dashboards
Gestionar Dashboards:

En la interfaz de Grafana, ve a Dashboards > Manage.
Busca tu dashboard personalizado por el título definido en custom-dashboard.json.
Verificar Paneles:

Asegúrate de que los paneles muestran las métricas correctas y que los datos se están recibiendo de Prometheus.

Reproducción del Proyecto
Sigue estos pasos para reproducir el proyecto en tu entorno:

Clonar el Repositorio:

bash
Copiar código
git clone https://github.com/tu-usuario/proyectosre.git
cd proyectosre
Configurar Variables de Entorno:

Crea un archivo .env con las variables necesarias.

env
Copiar código
DOCKER_USERNAME=tu_usuario
DOCKER_PASSWORD=tu_contraseña
KUBE_NAMESPACE=monitoring
Configurar GitHub Secrets:

En tu repositorio de GitHub, ve a Settings > Secrets y agrega las siguientes secrets:

DOCKER_USERNAME
DOCKER_PASSWORD
Construir y Probar la Aplicación Localmente:

**bash**
Copiar código
cd src
python -m venv env
source env/bin/activate  # O .\env\Scripts\activate en Windows
pip install -r requirements.txt
uvicorn app:app --reload
Configurar Kubernetes:

Instalar Helm si aún no lo has hecho.

Desplegar Prometheus y Grafana:

**bash**
Copiar código
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring -f kube-prometheus-stack/values.yaml
Aplicar ConfigMaps de Dashboards:

**bash**
Copiar código
kubectl apply -f kube-prometheus-stack/prometheus-stack-grafana-dashboards-custom.yaml
Desplegar la Aplicación en Kubernetes:

Asegúrate de tener los manifiestos de Kubernetes listos en la carpeta kubernetes/.

**bash**
Copiar código
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
Configurar GitHub Actions:

Asegúrate de que los workflows de GitHub Actions están en .github/workflows/.
Realiza un push a la rama develop para ejecutar el workflow de CI.
Fusiona cambios en main para ejecutar el workflow de CD.
Acceder a Grafana y Verificar Dashboards:

Usa kubectl port-forward para acceder a Grafana.
Inicia sesión y verifica que los dashboards personalizados están cargados y muestran las métricas correctas.

Notas Finales
Este README proporciona una guía completa para configurar y desplegar una aplicación FastAPI con CI/CD utilizando GitHub Actions y Kubernetes, junto con monitorización mediante Prometheus y Grafana. Asegúrate de adaptar los archivos de configuración y valores según las necesidades específicas de tu proyecto.

Si encuentras algún problema durante el despliegue o configuración, revisa los logs de los pods en Kubernetes y verifica que todas las dependencias y configuraciones estén correctamente establecidas.