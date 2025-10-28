# Traefik Docker Compose con TLS

Este repositorio contiene un ejemplo completo de implementación de **Traefik** como proxy reverso usando Docker Compose con configuración TLS/SSL para desarrollo local.

## 📋 Descripción

Este proyecto demuestra cómo configurar Traefik v3.1.2 como proxy reverso con:

- ✅ **Generación automática de certificados SSL** auto-firmados
- ✅ **Redirección automática HTTP → HTTPS**
- ✅ **Dashboard de Traefik** con autenticación
- ✅ **Múltiples servicios** con routing basado en host
- ✅ **Red Docker personalizada** para aislamiento
- ✅ **Configuración dinámica** via archivos y labels

## 🏗️ Arquitectura

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Internet      │───▶│     Traefik      │───▶│   Services      │
│   :8081/:8443   │    │   Proxy/LB       │    │   client,       │
└─────────────────┘    │                  │    │   whoami        │
                       └──────────────────┘    └─────────────────┘
                              │
                       ┌──────────────────┐
                       │  cert-generator  │
                       │  (SSL Certs)     │
                       └──────────────────┘
```

## 🚀 Inicio Rápido

### Prerrequisitos

- [Docker](https://docs.docker.com/get-docker/) (20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (2.0+)

### 1. Clonar e Iniciar

```bash
# Clonar el repositorio
git clone <tu-repo-url>
cd traefik-docker


# Iniciar todos los servicios
docker-compose up -d
```

### 2. Verificar Servicios

Una vez iniciado, puedes acceder a:

| Servicio | URL | Descripción |
|----------|-----|-------------|
| **Dashboard Traefik** | https://dashboard.docker.localhost:8443 | Panel de control de Traefik |
| **Cliente Web** | https://web.docker.localhost:8443 | Aplicación web de ejemplo |
| **WhoAmI** | https://whoami.docker.localhost:8443 | Servicio de información |

> **Nota:** Acepta el certificado auto-firmado en tu navegador para acceder a los servicios HTTPS.

## 📁 Estructura del Proyecto

```
traefik-docker/
├── docker-compose.yaml      # Configuración principal
├── dynamic/
│   └── tls.yaml            # Configuración TLS dinámica
├── client/                 # Archivos del cliente web
│   └── index.html         # Página web de ejemplo
└── README.md              # Este archivo
```

## 🔧 Servicios Configurados

### 1. **cert-generator**

- Genera certificados SSL auto-firmados para `*.docker.localhost`
- Se ejecuta una sola vez al inicio (`restart: "no"`)
- Válido por 365 días

### 2. **proxy (Traefik)**

**Características principales:**
- **EntryPoints:** Puerto 8081 (HTTP) → 8443 (HTTPS)
- **Redirección automática:** HTTP a HTTPS
- **Providers:** Docker + configuración dinámica
- **Dashboard:** Habilitado en `dashboard.docker.localhost`
- **Métricas:** Prometheus habilitado

### 3. **client (Nginx)**

- Servidor web simple con contenido estático
- Accesible en `web.docker.localhost`

### 4. **whoami**
```yaml
image: traefik/whoami
```
- Servicio de prueba que muestra información de la petición
- Útil para debugging y verificación

## 🔒 Configuración TLS

### Certificados Auto-Firmados

El servicio `cert-generator` crea:
- **Certificado:** `local.crt` (wildcard para `*.docker.localhost`)
- **Clave privada:** `local.key`
- **Algoritmo:** RSA 2048 bits
- **Validez:** 365 días

### Configuración Dinámica

El archivo `dynamic/tls.yaml` configura Traefik para usar los certificados:

```yaml
tls:
  certificates:
    - certFile: /certs/local.crt
      keyFile: /certs/local.key
```

## 🌐 Routing y Labels

### Configuración de Routing

Los servicios usan **labels Docker** para configurar el routing:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"
  - "traefik.http.routers.whoami.entrypoints=websecure"
  - "traefik.http.routers.whoami.tls=true"
```

### Hosts Configurados


```
127.0.0.1 dashboard.docker.localhost
127.0.0.1 web.docker.localhost
127.0.0.1 whoami.docker.localhost
```

## 🔧 Comandos Útiles

### Gestión de Servicios

```bash
# Iniciar servicios
docker-compose up -d

# Ver logs
docker-compose logs -f

# Parar servicios
docker-compose down -v

# Recrear certificados
docker-compose restart cert-generator

# Ver estado de servicios
docker-compose ps
```


## 📚 Referencias y Documentación

### Documentación Oficial

- **[Traefik Docker Provider](https://doc.traefik.io/traefik/reference/install-configuration/providers/docker/)** - Configuración del provider Docker
- **[Docker Compose con Traefik](https://docs.docker.com/guides/traefik/)** - Guía oficial de Docker
- **[Traefik TLS](https://doc.traefik.io/traefik/https/tls/)** - Configuración TLS/SSL
- **[Traefik EntryPoints](https://doc.traefik.io/traefik/routing/entrypoints/)** - Configuración de puntos de entrada
- **[Traefik Dashboard](https://doc.traefik.io/traefik/operations/dashboard/)** - Panel de control

### Recursos Adicionales

- **[Traefik v3 Migration](https://doc.traefik.io/traefik/migration/v2-to-v3/)** - Guía de migración
- **[Docker Compose Spec](https://compose-spec.io/)** - Especificación de Docker Compose
- **[OpenSSL Certificate Creation](https://www.openssl.org/docs/man1.1.1/man1/openssl-req.html)** - Documentación de OpenSSL

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit tus cambios (`git commit -am 'Agrega nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver el archivo `LICENSE` para más detalles.

## 🏷️ Tags

`traefik` `docker-compose` `reverse-proxy` `ssl` `tls` `https` `load-balancer` `microservices` `development`

---

### 💡 Próximos Pasos

- [ ] Configurar autenticación básica para el dashboard
- [ ] Agregar middleware de rate limiting
- [ ] Implementar certificados Let's Encrypt para producción
- [ ] Agregar monitoreo con Prometheus/Grafana
- [ ] Configurar logging centralizado

