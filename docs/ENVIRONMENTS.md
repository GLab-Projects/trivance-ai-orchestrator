# 🎛️ Sistema de Environments - Guía Completa

## 🤔 ¿Qué es esto?

El sistema de environments de Trivance te permite cambiar entre diferentes configuraciones (local, QA, producción) con **UN SOLO COMANDO**. Es como tener diferentes "modos" para tu aplicación:

- **🏠 Local**: Tu computadora, para desarrollo
- **🧪 QA**: Servidor de pruebas
- **🚀 Producción**: Servidor real con usuarios

## 🎯 ¿Cómo funciona?

### La magia en 3 pasos:

1. **Configuración centralizada**: Todo está en `trivance-dev-config/config/environments.json`
2. **Generación automática**: El sistema crea archivos `.env` para cada servicio (incluidos Docker)
3. **Un comando para cambiar**: Cambias TODOS los servicios de golpe (Docker + PM2)

### 🐳 Arquitectura Híbrida Docker + PM2:
- **Backends y DBs**: En contenedores Docker (PostgreSQL, MongoDB, APIs NestJS)
- **Frontend**: Con PM2 para hot-reload instantáneo
- **Integración automática**: Los environments manejan ambos sistemas

### Ejemplo visual:
```
Tu comando: ./trivance-dev-config/scripts/envs.sh switch qa

Lo que pasa:
├── ms_trivance_auth/.env           → Cambia a config de QA
├── ms_level_up_management/.env      → Cambia a config de QA
├── level_up_backoffice/.env         → Cambia a config de QA
├── trivance-mobile/.env             → Cambia a config de QA
├── docker/.env.docker-local         → Cambia a config de QA para Docker
└── docker/.env.docker-auth-local    → Cambia a config de QA para Docker

¡TODO sincronizado! 🎉 (Docker + PM2)
```

## 📋 Guía Rápida - Lo que necesitas saber

### 1️⃣ Ver en qué environment estás
```bash
./trivance-dev-config/scripts/envs.sh status
```

Te dirá algo como:
```
✅ Environment actual: local
📁 Archivos de configuración en: /tu-proyecto/envs
```

### 2️⃣ Cambiar de environment

**Para desarrollo local** (tu computadora):
```bash
./trivance-dev-config/scripts/envs.sh switch local
```

**Para servidor de pruebas** (QA):
```bash
./trivance-dev-config/scripts/envs.sh switch qa
```

**Para producción** (¡CUIDADO! 🚨):
```bash
./trivance-dev-config/scripts/envs.sh switch production
# Te pedirá confirmación porque es PRODUCCIÓN REAL
```

### 3️⃣ Después de cambiar

Reinicia los servicios:
```bash
./start-all.sh
```

## 🔐 Seguridad - MUY IMPORTANTE

### Para Local (tu computadora)
- ✅ **TODO ES AUTOMÁTICO**: Los secrets se generan solos
- ✅ **Es seguro**: Cada instalación tiene secrets únicos
- ✅ **No necesitas hacer nada**: Just works™

### Para QA/Producción
- ⚠️ **CONFIGURACIÓN MANUAL REQUERIDA**
- 📝 Pasos:
  1. Copia el archivo local como plantilla:
     ```bash
     cp envs/local.management.env envs/qa.management.env
     ```
  2. Edita con valores REALES de QA:
     - URLs reales del servidor QA
     - Credenciales de base de datos QA
     - API keys de servicios externos
  3. **NUNCA** subas estos archivos a Git

## 🗂️ ¿Dónde están los archivos?

```
tu-proyecto/
├── envs/                          # 📁 Aquí están TODAS las configuraciones
│   ├── local.management.env       # Config local del backend
│   ├── local.auth.env            # Config local de auth
│   ├── local.backoffice.env      # Config local del frontend
│   ├── local.mobile.env          # Config local de la app
│   ├── qa.*.env                  # Configs de QA (crearlas manualmente)
│   └── production.*.env          # Configs de producción (crearlas manualmente)
└── .trivance-secrets             # 🔐 Secrets autogenerados (NO SUBIR A GIT)
```

## 🛠️ Comandos Completos

### Básicos
```bash
# Ver estado actual
./trivance-dev-config/scripts/envs.sh status

# Cambiar environment
./trivance-dev-config/scripts/envs.sh switch [local|qa|production]

# Ver ayuda
./trivance-dev-config/scripts/envs.sh help
```

### Avanzados
```bash
# Validar que todo esté bien configurado
./trivance-dev-config/scripts/envs.sh validate

# Comparar dos environments
./trivance-dev-config/scripts/envs.sh diff local qa

# Sincronizar con environments.json
./trivance-dev-config/scripts/envs.sh sync
```

## ❓ Preguntas Frecuentes

### "¿Por qué no puedo editar el .env directamente?"
Los `.env` se generan automáticamente desde `environments.json`. Si los editas manualmente, se perderán los cambios al cambiar de environment.

### "¿Cómo agrego una nueva variable?"
1. Agrégala en `trivance-dev-config/config/environments.json`
2. Ejecuta: `./trivance-dev-config/scripts/envs.sh sync`
3. ¡Listo! Ya está en todos los servicios

### "¿Qué pasa si cambio a producción por error?"
- El sistema te pide confirmación (debes escribir "yes")
- Si ya lo hiciste, simplemente cambia de vuelta: `switch local`

### "No encuentro el archivo de QA"
Es normal. Los archivos de QA y producción NO vienen incluidos por seguridad. Debes crearlos copiando los locales y editándolos.

## 🚀 Workflow Típico de Desarrollo

### Mañana - Empezar a trabajar:
```bash
cd ~/tu-proyecto
./start.sh              # Inicia todo en local automáticamente (Docker + PM2)
```

### Necesitas probar en QA:
```bash
./trivance-dev-config/scripts/envs.sh switch qa
./start.sh              # Ahora apunta a servidores QA
```

### Volver a desarrollo local:
```bash
./trivance-dev-config/scripts/envs.sh switch local
./start.sh              # De vuelta a tu máquina
```

## 🆘 Solución de Problemas

### "Me dice que faltan archivos de QA"
```bash
# Crear los archivos QA copiando los locales
cp envs/local.management.env envs/qa.management.env
cp envs/local.auth.env envs/qa.auth.env
cp envs/local.backoffice.env envs/qa.backoffice.env
cp envs/local.mobile.env envs/qa.mobile.env

# Ahora edita cada uno con valores de QA
```

### "Los servicios no se conectan después de cambiar"
```bash
# Asegúrate de reiniciar después de cambiar environment
./start.sh              # El menú te permitirá detener y reiniciar servicios
# O manualmente:
# docker-compose down && docker-compose up -d  # Para servicios Docker
# pm2 restart all                               # Para el frontend
```

### "No sé en qué environment estoy"
```bash
./trivance-dev-config/scripts/envs.sh status
# O mira el archivo:
cat envs/.current_environment
```

## 🐳 Gestión de Docker con Environments

### ¿Cómo se integra Docker?

Cuando cambias de environment, el sistema también genera archivos `.env` específicos para Docker:

```bash
# Cambiar environment también configura Docker
./trivance-dev-config/scripts/envs.sh switch qa

# Esto genera automáticamente:
# ├── docker/.env.docker-local      # Para Management API
# └── docker/.env.docker-auth-local # Para Auth API
```

### Comandos específicos para Docker:

```bash
# Ver contenedores corriendo
docker ps

# Reiniciar servicios Docker después de cambiar environment
docker-compose -f trivance-dev-config/docker/docker-compose.yaml down
docker-compose -f trivance-dev-config/docker/docker-compose.yaml up -d

# Ver logs de Docker
docker logs trivance_management
docker logs trivance_auth
```

### Servicios en Docker vs PM2:

| Servicio | Tecnología | Puerto | Comando |
|----------|------------|---------|----------|
| PostgreSQL | Docker | 5432 | `docker logs trivance_postgres` |
| MongoDB | Docker | 27017 | `docker logs trivance_mongodb` |
| Auth API | Docker | 3001 | `docker logs trivance_auth` |
| Management API | Docker | 3000 | `docker logs trivance_management` |
| Frontend | PM2 | 5173 | `pm2 logs backoffice` |

## 📚 Para Aprender Más

- **Archivo maestro**: `trivance-dev-config/config/environments.json`
- **Script principal**: `trivance-dev-config/scripts/envs.sh`
- **Documentación técnica**: `trivance-dev-config/README.md`
- **Docker**: `trivance-dev-config/docs/DOCKER.md`

---

💡 **Tip Final**: El 90% del tiempo usarás solo estos comandos:
- `status` - Ver dónde estás
- `switch local` - Volver a desarrollo
- `switch qa` - Ir a pruebas
- `./start.sh` - Iniciar todos los servicios (Docker + PM2)

¡Eso es todo! 🎉