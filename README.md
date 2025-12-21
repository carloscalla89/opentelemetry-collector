🔭 Observabilidad: OpenTelemetry Collector
Esta implementación despliega el OpenTelemetry Collector (Contrib) en modo DaemonSet, asegurando que exista una instancia del colector en cada nodo del clúster Kubernetes para recolectar, procesar y exportar telemetría (Logs, Métricas y Trazas).


📄 Archivos de Configuración
otel-values.yaml: Configuración principal del Helm Chart.
otel-service-monitor.yaml: Recurso para integración con Prometheus Operator.

⚙️ Funcionalidades Implementadas
La configuración define un pipeline de observabilidad robusto con las siguientes características:

1. Recolección de Datos (Receivers)
OTLP (gRPC/HTTP): Recepción estándar de trazas y métricas desde las aplicaciones.

Hostmetrics: Monitoreo de infraestructura del nodo (CPU, Memoria, Disco, Red, Paginación).

Filelog: Lectura de logs de contenedores directamente desde /var/log/pods, excluyendo los logs propios del colector para evitar bucles.

2. Procesamiento y Enriquecimiento (Processors)
Kubernetes Attributes: Enriquece automáticamente la telemetría con metadatos del clúster (k8s.pod.name, k8s.namespace.name, k8s.deployment.name, etc.) basándose en la IP o UID del Pod.

Resource Detection: Detecta el entorno (demo-minikube) y añade información del host.

Log Filtering (Business Logic): Se aplica un filtro estricto (Allowlist) para procesar únicamente los logs provenientes del namespace default. El resto de logs son descartados para optimizar costos y almacenamiento.

3. Exportación y Destinos (Exporters)
El colector actúa como un agente unificado enviando datos a múltiples backends:

Elastic Cloud (APM): Destino principal para Trazas y Logs (comprimidos con gzip).

Datadog:

Envío de métricas y metadatos de host.

Uso del Datadog Connector para derivar métricas a partir de las trazas recibidas.

Prometheus: Exposición de métricas en el puerto 8889 con conversión de atributos de recursos a etiquetas de Prometheus.


🔍 Integración con Prometheus Operator
Se incluye un manifiesto ServiceMonitor que permite al Prometheus Operator descubrir y raspar (scrape) automáticamente las métricas del OTel Collector.

Endpoint: /metrics en puerto 8889.

Discovery: Basado en la etiqueta app.kubernetes.io/name: opentelemetry-collector.

Namespace: Configurado para ser detectado por la instancia de Prometheus corriendo en el stack de monitoreo (label release: prometheus-stack).