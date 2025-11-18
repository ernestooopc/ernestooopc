<h1 align="center">📜 DSpace Handle Server – Bitácora de Diagnóstico y Corrección</h1> <p align="center"> <strong>Servidor:</strong> srvdspace (WSP01) • <strong>Fecha:</strong> 15/11/2025 • <strong>Técnico:</strong> Aldo </p>

## 🧩 Fase 1: Conexión Inicial al Servidor

### 🔌 Identificación de IP (Tailscale):

100.90.157.8

### 📡 Conexión SSH:

ssh danmer@100.90.157.8


### ✔ Resultado:
Acceso exitoso al servidor WSP01 (Debian GNU/Linux).

## 🚨 Fase 2: Diagnóstico y Resolución del Problema de Disco

### 📦 Verificación del uso del disco:

df -h
lsblk -f


### 📁 Verificación de carpetas con mayor consumo:

sudo du -h / --max-depth=1 | sort -hr
sudo du -h /dspace --max-depth=1 | sort -hr


### 🕵️ Diagnóstico:
La partición / estaba al 100%, saturada por logs.

#### 👉 Se detectó que /dspace/log ocupaba 41 GB.

#### 🧹 Limpieza de Logs

#### Detener Tomcat

sudo systemctl stop tomcat9


#### Eliminar logs antiguos

sudo find /dspace/log -type f -mtime +107 -delete


#### Revisar nuevamente el espacio

df -h
sudo du -sh /dspace/log


#### Reiniciar Tomcat

sudo systemctl start tomcat9


### ✔ Resultado:
Espacio liberado: ~30 GB
/dspace/log bajó de 41 GB → 11 GB.

## 🔧 Fase 3: Diagnóstico del Handle Server

### 🔍 Verificación de puertos (2641 y 8000):

sudo netstat -tulnp | grep -E "2641|8000"
sudo ss -tulpn | grep java


### 📌 Problema encontrado:
El proceso Java del Handle Server solo estaba escuchando en:

192.168.0.17


### 🛑 Esto impedía recibir conexiones desde fuera.

## 🛠️ Fase 4: Corrección del Bind Address y Reinicio del Servicio

### 📂 Backup de configuración

sudo cp /dspace/handle-server/config.dct /dspace/handle-server/config.dct.respaldo


### ✏️ Edición de 3 instancias de bind_address:

De:

192.168.0.17


A:

0.0.0.0


### 🔑 Reconfiguración del prefijo Handle

/dspace/bin/make-handle-config


### 📌 Verificar JSON generado

cat /dspace/handle-server/config.dct | grep bind


### 🔄 Reinicio Completo del Handle Server

#### Detener proceso antiguo

ps aux | grep handle
sudo kill 85825


#### Iniciar nueva instancia

nohup java -Djava.awt.headless=true \
-Djava.security.egd=file:/dev/./urandom \
-classpath "/dspace/lib/*:/dspace/config" \
-Ddspace.log.init.disable=true \
-Dlog4j.configuration=log4j-handle-plugin.properties \
net.handle.server.Main /dspace/handle-server &


#### Verificar si el proceso inició correctamente

ps -ef | grep handle
tail -f /dspace/log/handle-server.log


#### 🔬 Verificación Final

sudo netstat -tulnp | grep -E "2641|8000"
sudo ss -tulnp | grep handle


#### ✔ Salida esperada:

:::2641    LISTEN
:::8000    LISTEN


Ahora el Handle Server escucha correctamente en todas las interfaces (0.0.0.0).

<h2 align="center">✅ Estado Final: Sistema Operativo y Servicios Estables</h2> <p align="center"> El Handle Server quedó completamente funcional, se corrigió el bind address y se solucionó la saturación crítica del disco. </p>
