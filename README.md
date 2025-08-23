# 🐳 Guía Completa de Comandos Docker y Docker Compose

Esta guía contiene todos los comandos esenciales para trabajar con Docker y Docker Compose(Contenedor de Servicios de DB), organizados por categorías para facilitar su consulta.

## 📋 Tabla de Contenidos

- [Comandos Docker Compose Específicos](#compose)
- [Monitoreo y Logs](#monitoreoLogs)
- [Red y Conectividad](#redConectividad)
- [Gestión de Volúmenes](#gestionVolumenes)
- [Acceso a Bases de Datos por Consola](#dbConsola)
- [Backup y Restore](#backupRestore)
- [Limpieza del Sistema](#limpiezaSistema)
- [Comandos Docker Generales Útiles](#general)

> Ver si un puerto esta ocupado: netstat -ano | findstr :4200

---

<h2 id="compose">🚀 Comandos Docker Compose Específico DB</h2>

### Gestión Básica de Servicios

```bash
#------------------
# RUN
#------------------
# Levantar todos los servicios en modo detached
docker compose up -d

# Levantar con recreación forzada de contenedores
docker compose up -d --force-recreate

# Levantar con reconstrucción de imágenes
docker compose up -d --build

# Recrear solo un servicio específico
docker compose up -d --force-recreate sqlserver

# Levantar solo un servicio
docker compose start mysql

#------------------
# RESTART
#------------------
# Reinicar todos los servicios
docker compose restart mysql

# Reinicar solo un servicio
docker compose restart mysql

#------------------
# STOP
#------------------
# Detener todos los servicios
docker compose stop

# Detener solo un servicio
docker compose stop mysql

#------------------
# REMOVE
#------------------
# Detener y eliminar el contenedor
docker compose down

# Detener y y eliminar contenedor con sus volúmenes
docker compose down -v

# Detener y eliminar el contenedor (pero conserva volúmenes)
docker compose rm -s mysql

# -s = stop antes de remover
# -f = force (sin confirmación)
docker compose rm -sf mysql
```

### Validación y Configuración

```bash
# Validar la configuración del docker compose.yml
docker compose config

# Ver el estado de todos los servicios
docker compose ps

# Ver el estado de un servicio específico
docker compose ps mysql
```

---

<h2 id="monitoreoLogs">📊 Monitoreo y Logs</h2>

### Visualización de Logs

```bash
# Ver logs de un servicio específico
docker compose logs mysql

# Ver logs en tiempo real
docker compose logs -f

# Ver últimas 50 líneas de logs
docker compose logs --tail 50 sqlserver
```

---

<h2 id="redConectividad">🌐 Red y Conectividad</h2>

### Inspección de Redes

```bash
# Ver las IPs asignadas a la red del proyecto
docker network inspect <project>_db_network

# Ver la configuración de red de un contenedor específico
docker inspect cnt_mysql | grep IPAddress
```

### Pruebas de Conectividad

```bash
# Probar conectividad entre contenedores
docker exec cnt_adminer ping mysql
docker exec cnt_mysql ping postgres
```

---

<h2 id="gestionVolumenes">💾 Gestión de Volúmenes</h2>

### Listado e Inspección

```bash
# Listar todos los volúmenes
docker volume ls

# Inspeccionar volúmenes específicos
docker volume inspect databases_mysql_data
docker volume inspect databases_postgres_data
docker volume inspect databases_sqlserver_data

# Ver la ruta de montaje de un volumen
docker volume inspect databases_mysql_data --format '{{.Mountpoint}}'

# Ver uso de espacio detallado
docker system df -v
```

### Exploración de Contenido

```bash
# Explorar contenido de volúmenes desde dentro del contenedor
docker exec -it cnt_mysql ls -la /var/lib/mysql
docker exec -it cnt_postgres ls -la /var/lib/postgresql/data
docker exec -it cnt_sqlserver ls -la /var/opt/mssql
```

### Copia de Archivos

```bash
# Del volumen al host
docker cp cnt_mysql:/var/lib/mysql/testdb ./backup_mysql/

# Del host al volumen
docker cp ./mi_backup.sql cnt_mysql:/tmp/
```

---

<h2 id="dbConsola">🖥️ Acceso a Bases de Datos por Consola</h2>

### Conexión Directa a las Bases de Datos

#### MySQL
```bash
# Acceder a MySQL desde dentro del contenedor
docker exec -it cnt_mysql mysql -u user -p'tuContraseña' testdb

# Acceder como root
docker exec -it cnt_mysql mysql -u root -p'rootpass'

# Ejecutar comando SQL directamente
docker exec cnt_mysql mysql -u user -p'tuContraseña' testdb -e "SHOW TABLES;"

# Conectar desde el host (requiere cliente MySQL instalado)
mysql -h localhost -P 3306 -u user -p'tuContraseña' testdb
```

#### PostgreSQL
```bash
# Acceder a PostgreSQL desde dentro del contenedor
docker exec -it cnt_postgres psql -U user -d testdb

# Acceder con comando específico
docker exec -it cnt_postgres psql -U user -d testdb -c "\dt"

# Ejecutar comando SQL directamente
docker exec cnt_postgres psql -U user -d testdb -c "SELECT version();"

# Conectar desde el host (requiere cliente psql instalado)
psql -h localhost -p 5432 -U user -d testdb
```

#### SQL Server
```bash
# OPCIÓN 1: Usar sqlcmd (ubicación actualizada para SQL Server 2022)
docker exec -it cnt_sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'tuContraseña' -C

# OPCIÓN 2: Si sqlcmd no está disponible, acceder via bash y verificar ubicación
docker exec -it cnt_sqlserver bash
# Dentro del contenedor buscar sqlcmd:
find / -name sqlcmd 2>/dev/null

# OPCIÓN 3: Instalar sqlcmd dentro del contenedor (temporal)
docker exec -it cnt_sqlserver bash
apt-get update && apt-get install -y curl
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
apt-get update && ACCEPT_EULA=Y apt-get install -y mssql-tools18
export PATH="$PATH:/opt/mssql-tools18/bin"

# Ejecutar comando SQL directamente (con ubicación actualizada)
docker exec cnt_sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'tuContraseña' -C -Q "SELECT @@VERSION;"

# Listar bases de datos
docker exec cnt_sqlserver /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'tuContraseña' -C -Q "SELECT name FROM sys.databases;"

# Conectar desde el host (requiere sqlcmd instalado)
sqlcmd -S localhost,1433 -U sa -P 'tuContraseña' -C
```

> **⚠️ Nota importante para SQL Server 2022**: 
> - La ubicación cambió de `/opt/mssql-tools/bin/` a `/opt/mssql-tools18/bin/`
> - Se requiere el parámetro `-C` para confiar en el certificado del servidor
> - Si sqlcmd no está disponible, se puede instalar dentro del contenedor

### Comandos SQL Útiles por Motor

#### MySQL - Comandos Básicos
```sql
-- Ver bases de datos
SHOW DATABASES;

-- Usar base de datos
USE testdb;

-- Ver tablas
SHOW TABLES;

-- Describir tabla
DESCRIBE nombre_tabla;

-- Ver usuarios
SELECT User, Host FROM mysql.user;
```

#### PostgreSQL - Comandos Básicos
```sql
-- Ver bases de datos
\l

-- Conectar a base de datos
\c testdb

-- Ver tablas
\dt

-- Describir tabla
\d nombre_tabla

-- Ver usuarios
\du

-- Salir
\q
```

#### SQL Server - Comandos Básicos
```sql
-- Ver bases de datos
SELECT name FROM sys.databases;

-- Usar base de datos
USE testdb;

-- Ver tablas
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE';

-- Describir tabla
sp_help nombre_tabla;

-- Ver información del servidor
SELECT @@VERSION;
```

---

<h2 id="backupRestore"> Backup y Restore</h2>

### Creación de Backups

```bash
# Backup de MySQL
docker exec cnt_mysql mysqldump -u user -p'tuContraseña' testdb > backup_mysql.sql

# Backup de PostgreSQL
docker exec cnt_postgres pg_dump -U user testdb > backup_postgres.sql

# Backup de SQL Server
docker exec cnt_sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'tuContraseña' -Q "BACKUP DATABASE [master] TO DISK = N'/var/opt/mssql/backup.bak'"
```

---

<h2 id="limpiezaSistema">🧹 Limpieza del Sistema</h2>

### Limpieza de Volúmenes

```bash
# Ver espacio usado por Docker
docker system df

# Limpiar volúmenes no utilizados
docker volume prune

# Limpiar volúmenes sin confirmación
docker volume prune -f

# Eliminar volúmenes específicos (¡CUIDADO!)
docker volume rm databases_mysql_data
docker volume rm databases_postgres_data
docker volume rm databases_sqlserver_data
```

### Limpieza Específica del Proyecto

```bash
# Método seguro: detener contenedores primero
docker compose down

# Luego eliminar volúmenes específicos
docker volume rm databases_mysql_data databases_postgres_data databases_sqlserver_data

# Limpiar contenedores problemáticos
docker rm cnt_postgres cnt_sqlserver
```

### Limpieza Completa del Sistema

⚠️ **ATENCIÓN**: Estos comandos eliminan TODOS los recursos de Docker

```bash
# Ver todos los volúmenes antes de limpiar
docker volume ls

# Parar TODOS los contenedores
docker stop $(docker ps -aq)

# Eliminar TODOS los volúmenes
docker volume rm $(docker volume ls -q)

# Limpieza completa y agresiva
docker volume prune -a -f

# Limpieza completa del sistema Docker
docker system prune -a --volumes -f
```

### Verificación Post-Limpieza

```bash
# Verificar que se eliminaron los volúmenes
docker volume ls

# Ver espacio liberado
docker system df

# Ver qué contenedores usan cada volumen
docker ps -a --format "table {{.Names}}\t{{.Mounts}}"
```

---

<h2 id="general">🔧 Comandos Docker Generales Útiles</h2>

### Gestión de Imágenes

```bash
# Crear imagen con tag específico
docker build -t nombre_imagen:tag .

# Crear imagen con múltiples tags
docker build -t nombre_imagen:latest -t nombre_imagen:v1.0 .

# Listar imágenes
docker images

# Eliminar una imagen
docker rmi <image_name>

# Descargar una imagen
docker pull <image_name>:tag

# Construir una imagen desde Dockerfile
docker build -t <image_name> .
```

### Gestión de Contenedores

```bash
#------------------
# CREATE
#------------------
# Crear un contenedor simple
docker run -d --name mi_contenedor nombre_imagen

# Crear contenedor con puerto mapeo de puerto
docker run -d -p 8080:80 --name mi_web nginx

# Crear contenedor con volúmenes
docker run -d -v /ruta/host:/ruta/contenedor --name mi_contenedor imagen

# Crear contenedor con variables de entorno
docker run -d -e VAR1=valor1 -e VAR2=valor2 --name mi_contenedor imagen

#------------------
# VIEW
#------------------
# Listar contenedores en ejecución
docker ps

# Listar todos los contenedores (incluidos los detenidos)
docker ps -a

#------------------
# OPTIONS
#------------------
# Acceder a un contenedor en modo interactivo
docker exec -it cnt_sqlserver bash

# Detener un contenedor específico
docker stop <container_name>

# Reiniciar un contenedor
docker restart <container_name>

# Eliminar un contenedor
docker rm <container_name>
```

### Información del Sistema

```bash
# Ver información general de Docker
docker info

# Ver versión de Docker
docker version

# Ver uso de espacio
docker system df

# Ver eventos en tiempo real
docker events
```

### Limpieza General

```bash
# Limpiar contenedores detenidos
docker container prune

# Limpiar imágenes no utilizadas
docker image prune

# Limpiar redes no utilizadas
docker network prune

# Limpieza completa del sistema
docker system prune -a
```

---

## 📝 Notas Importantes

- **⚠️ Precaución**: Los comandos de limpieza pueden eliminar datos permanentemente
- **🔄 Recreación**: Usa `--force-recreate` cuando tengas problemas de conectividad
- **📋 Logs**: Siempre revisa los logs cuando un servicio no funcione correctamente
- **🔍 Inspección**: Usa `docker inspect` para obtener información detallada de cualquier recurso
- **⏰ Health Checks**: Los servicios tienen health checks configurados, úsalos para verificar el estado

---

## 🎯 Comandos de Solución Rápida

```bash
# Reinicio completo del proyecto
docker compose down && docker compose up -d

# Ver estado de salud rápido
docker compose ps

# Logs en tiempo real de todos los servicios
docker compose logs -f

# Verificar conectividad de red
docker network ls && docker network inspect <project>_db_network
```
