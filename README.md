# Lab 12: Stack de Monitoreo con Grafana + Prometheus + Loki

[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![Grafana](https://img.shields.io/badge/Grafana-10.0.0-F46800?style=flat&logo=grafana&logoColor=white)](https://grafana.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Latest-E6522C?style=flat&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Loki](https://img.shields.io/badge/Loki-2.9.0-F46800?style=flat&logo=grafana&logoColor=white)](https://grafana.com/oss/loki/)

Sistema completo de monitoreo y observabilidad utilizando Docker Compose para la recolección, almacenamiento y visualización de métricas y logs.

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Uso](#uso)
- [Configuración](#configuración)
- [Queries de Ejemplo](#queries-de-ejemplo)
- [Troubleshooting](#troubleshooting)
- [Referencias](#referencias)

## 📖 Descripción

Este proyecto implementa un stack completo de observabilidad que incluye:

- **Grafana**: Plataforma de visualización y análisis de datos
- **Prometheus**: Sistema de monitoreo y base de datos de series temporales
- **Loki**: Sistema de agregación de logs inspirado en Prometheus
- **Promtail**: Agente para recolectar y enviar logs a Loki
- **Aplicación Node.js**: Aplicación de ejemplo que expone métricas y genera logs

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│                      GRAFANA                            │
│                    (Puerto 3001)                        │
│              Visualización y Dashboards                 │
└────────────────┬────────────────────┬───────────────────┘
                 │                    │
                 │                    │
      ┌──────────▼──────────┐  ┌─────▼──────────────┐
      │    PROMETHEUS       │  │       LOKI         │
      │   (Puerto 9090)     │  │   (Puerto 3100)    │
      │  Métricas/Alertas   │  │  Agregación Logs   │
      └──────────┬──────────┘  └─────▲──────────────┘
                 │                   │
                 │            ┌──────┴──────────┐
                 │            │    PROMTAIL     │
                 │            │  Recolector de  │
                 │            │      Logs       │
                 │            └──────▲──────────┘
                 │                   │
      ┌──────────▼───────────────────┴──────────┐
      │         APLICACIÓN NODE.JS              │
      │           (Puerto 3000)                 │
      │   - Expone /metrics (Prometheus)        │
      │   - Genera logs (Docker → Promtail)     │
      └─────────────────────────────────────────┘
```

## 🔧 Requisitos Previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y corriendo
- Docker Compose v2.0+
- 4GB de RAM disponible (mínimo)
- Puertos disponibles: 3000, 3001, 3100, 9090

### Verificar Requisitos

```powershell
# Windows PowerShell
docker --version
docker-compose --version
docker ps
```

```bash
# Linux/Mac
docker --version
docker compose version
docker ps
```

## 🚀 Instalación

### 1. Clonar o Descargar el Proyecto

```bash
git clone <https://github.com/SorayaCrr/Grafana-Prometheus-Loki.git>
```

### 2. Estructura del Proyecto

```
lab12-sab/
├── docker-compose.yaml          # Orquestación de servicios
├── loki-config.yaml             # Configuración de Loki
├── promtail-config.yaml         # Configuración de Promtail
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   ├── prometheus.yml   # Data source Prometheus
│       │   └── loki.yml         # Data source Loki
│       └── dashboards/
│           └── dashboards.yml   # Configuración de dashboards
├── prometheus/
│   └── prometheus.yml           # Configuración de Prometheus
└── src/
    ├── Dockerfile               # Imagen de la app
    ├── index.js                 # Aplicación Node.js
    ├── package.json
    └── package-lock.json
```

### 3. Levantar los Servicios

#### Windows PowerShell

```powershell
# Levantar todos los servicios
docker-compose -f docker-compose.yaml up -d --build

# Verificar estado
docker-compose -f docker-compose.yaml ps

# Ver logs en tiempo real
docker-compose -f docker-compose.yaml logs -f
```

#### Linux/Mac

```bash
# Levantar todos los servicios
docker compose up -d --build

# Verificar estado
docker compose ps

# Ver logs en tiempo real
docker compose logs -f
```

### 4. Verificar que Todo Esté Funcionando

```powershell
# Windows
Invoke-WebRequest -Uri "http://localhost:3001" -UseBasicParsing  # Grafana
Invoke-WebRequest -Uri "http://localhost:9090" -UseBasicParsing  # Prometheus
Invoke-WebRequest -Uri "http://localhost:3100/ready" -UseBasicParsing  # Loki
Invoke-WebRequest -Uri "http://localhost:3000" -UseBasicParsing  # App
```

```bash
# Linux/Mac
curl http://localhost:3001  # Grafana
curl http://localhost:9090  # Prometheus
curl http://localhost:3100/ready  # Loki
curl http://localhost:3000  # App
```

## 💻 Uso

### Acceder a Grafana

1. Abrir navegador en: **http://localhost:3001**
2. Credenciales:
   - **Usuario**: `root`
   - **Password**: `root`

### Explorar Logs en Grafana

1. Click en **Explore** (🧭 ícono de brújula) en el menú lateral
2. Seleccionar **Loki** en el dropdown superior
3. Click en **Code** para escribir queries manualmente

### Ejemplo de Query Básica

```logql
{container="app"}
```

### Acceder a Prometheus

- URL: **http://localhost:9090**
- Explorar métricas de la aplicación: `http_requests_total`

### Acceder a la Aplicación

- URL: **http://localhost:3000**
- Endpoint de métricas: **http://localhost:3000/metrics**

## ⚙️ Configuración

### Data Sources en Grafana

#### Prometheus
```yaml
name: Prometheus
type: prometheus
url: http://prometheus:9090
```

#### Loki
```yaml
name: Loki
type: loki
url: http://loki:3100
```

Ambos data sources están pre-configurados mediante provisioning.

### Configuración de Loki

Ubicación: `loki-config.yaml`

```yaml
auth_enabled: false
server:
  http_listen_port: 3100

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
```

### Configuración de Promtail

Ubicación: `promtail-config.yaml`

**Jobs configurados:**
- `docker`: Recolecta logs de contenedores Docker
- `system`: Recolecta logs del sistema operativo

## 📊 Queries de Ejemplo

### LogQL (Loki)

#### Queries Básicas

```logql
# Ver logs de la aplicación
{container="app"}

# Ver logs de Prometheus
{container="prometheus"}

# Ver logs de todos los contenedores Docker
{job="docker"}

# Ver logs de Loki
{container="loki"}

# Ver logs de Promtail
{container="promtail"}
```

#### Filtrado de Logs

```logql
# Filtrar logs que contengan "Server"
{container="app"} |= "Server"

# Filtrar logs que contengan "error" (case insensitive)
{container="app"} |~ "(?i)error"

# Excluir logs que contengan "debug"
{container="app"} != "debug"

# Múltiples filtros
{container="app"} |= "Server" != "debug"
```

#### Agregaciones

```logql
# Contar logs por segundo
rate({container="app"}[1m])

# Contar líneas totales en 5 minutos
count_over_time({container="app"}[5m])

# Bytes procesados por segundo
rate({container="app"}[1m]) | unwrap bytes
```

#### Queries Avanzadas

```logql
# Agrupar por nivel de log
sum by (level) (rate({container="app"} | json [1m]))

# Top 10 contenedores con más logs
topk(10, sum by (container) (rate({job="docker"}[5m])))

# Logs de errores en las últimas 24 horas
{container="app"} |~ "(?i)error|exception|fatal" [24h]
```

### PromQL (Prometheus)

```promql
# Total de peticiones HTTP
http_requests_total

# Tasa de peticiones por segundo
rate(http_requests_total[5m])

# Uso de CPU
process_cpu_seconds_total

# Uso de memoria
process_resident_memory_bytes
```

## 🛠️ Comandos Útiles

### Docker Compose

```bash
# Iniciar servicios
docker-compose up -d

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes (⚠️ borra datos)
docker-compose down -v

# Ver logs de un servicio específico
docker-compose logs -f grafana
docker-compose logs -f loki
docker-compose logs -f promtail

# Reiniciar un servicio
docker-compose restart grafana

# Reconstruir servicios
docker-compose up -d --build

# Ver estado de contenedores
docker-compose ps

# Ver uso de recursos
docker stats
```

### Generar Tráfico en la App (para logs)

```powershell
# Windows PowerShell
1..20 | ForEach-Object { 
    Invoke-WebRequest -Uri "http://localhost:3000" -UseBasicParsing
    Start-Sleep -Milliseconds 500
}
```

```bash
# Linux/Mac
for i in {1..20}; do
    curl http://localhost:3000
    sleep 0.5
done
```

## 🐛 Troubleshooting

### Problema: Servicios no inician

```bash
# Ver logs detallados
docker-compose logs

# Verificar que Docker Desktop esté corriendo
docker ps

# Reiniciar Docker Desktop
```

### Problema: Puerto ya en uso

```powershell
# Windows: Ver qué proceso usa el puerto
Get-NetTCPConnection -LocalPort 3001

# Cambiar puerto en docker-compose.yaml
# ports:
#   - "3002:3000"  # Cambiar 3001 a 3002
```

```bash
# Linux/Mac: Ver qué proceso usa el puerto
lsof -i :3001

# Matar el proceso
kill -9 <PID>
```

### Problema: Grafana no carga

```bash
# Reiniciar Grafana
docker-compose restart grafana

# Ver logs de Grafana
docker-compose logs -f grafana

# Limpiar caché del navegador
# Ctrl+Shift+Delete o abrir en modo incógnito
```

### Problema: Loki no recibe logs

```bash
# Verificar que Promtail esté corriendo
docker-compose logs -f promtail

# Verificar conectividad Promtail → Loki
docker-compose exec promtail wget -O- http://loki:3100/ready

# Reiniciar Promtail
docker-compose restart promtail
```

### Problema: Descarga de imágenes falla

```bash
# Descargar imágenes manualmente
docker pull grafana/grafana:10.0.0
docker pull prom/prometheus:latest
docker pull grafana/loki:2.9.0
docker pull grafana/promtail:2.9.0
docker pull node:18-alpine

# Luego levantar servicios
docker-compose up -d
```

## 📚 Referencias

- [Documentación de Grafana](https://grafana.com/docs/)
- [Documentación de Loki](https://grafana.com/docs/loki/latest/)
- [LogQL: Lenguaje de Consultas de Loki](https://grafana.com/docs/loki/latest/logql/)
- [Documentación de Prometheus](https://prometheus.io/docs/)
- [PromQL: Lenguaje de Consultas de Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Tutorial de Referencia](https://medium.com/@netopschic/implementing-the-log-monitoring-stack-using-promtail-loki-and-grafana-using-docker-compose-bcb07d1a51aa)

## 📝 Notas Adicionales

### Persistencia de Datos

Los datos de Loki se almacenan en un volumen Docker:
```yaml
volumes:
  loki-data:
```

Para eliminar todos los datos:
```bash
docker-compose down -v
```

### Seguridad

⚠️ **Importante**: Las credenciales por defecto (`root`/`root`) son solo para desarrollo.

Para producción:
1. Cambiar credenciales de Grafana
2. Habilitar HTTPS
3. Configurar autenticación robusta
4. Implementar network policies

### Escalabilidad

Para entornos de producción considerar:
- Loki en modo microservicios
- Almacenamiento en S3/GCS
- Alta disponibilidad con múltiples réplicas
- Configurar retención de logs
- Implementar alertas

## 👨‍💻 Autor

**[Soraya Carruitero Ortiz]**  
