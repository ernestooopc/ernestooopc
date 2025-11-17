<h1 align="center">📜 DSpace Handle Server – Bitácora de Diagnóstico y Corrección</h1> <p align="center"> <strong>Servidor:</strong> srvdspace (WSP01) • <strong>Fecha:</strong> 15/11/2025 • <strong>Técnicos:</strong> Aldo</p>
🧩 Fase 1: Conexión Inicial al Servidor

🔌 Identificación de IP (Tailscale):

100.90.157.8


📡 Conexión SSH:

ssh danmer@100.90.157.8


✔ Resultado:
Acceso exitoso al servidor WSP01 (Debian GNU/Linux).

🚨 Fase 2: Diagnóstico y Resolución del Problema de Disco

📦 Verificación del uso del disco:

df -h


Diagnóstico:
La partición / estaba al 100%, saturada por logs.

🔎 Investigación de uso en /dspace:

sudo du -h /dspace --max-depth=1 | sort -hr


👉 Se detectó que la carpeta /dspace/log ocupaba 41 GB.

🧹 Limpieza de Logs

Detener Tomcat

sudo systemctl stop tomcat9


Eliminar logs antiguos

sudo find /dspace/log -type f -mtime +107 -delete


Reiniciar Tomcat

sudo systemctl start tomcat9


✔ Resultado:
Espacio liberado: ~30 GB
/dspace/log bajó de 41 GB → 11 GB.

🔧 Fase 3: Diagnóstico del Handle Server

🔍 Verificación de puertos:

sudo netstat -tulnp | grep '2641\|8000'


📌 Se encontró que el proceso Java del Handle Server estaba escuchando solo en:

192.168.0.17


🛑 Esto impedía recibir conexiones externas.

🛠️ Fase 4: Corrección del Bind Address y Reinicio del Servicio
📂 Backup de configuración
sudo cp /dspace/handle-server/config.dct /dspace/handle-server/config.dct.respaldo

✏️ Edición (3 instancias de bind_address)

De:

192.168.0.17


A:

0.0.0.0

🔑 Reconfiguración del prefijo
/dspace/bin/make-handle-config

🔄 Reinicio del Handle Server

Detener proceso antiguo

sudo kill 85825


Iniciar nueva instancia

nohup java -Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom \
-classpath /dspace/lib/*:/dspace/config -Ddspace.log.init.disable=true \
-Dlog4j.configuration=log4j-handle-plugin.properties \
net.handle.server.Main /dspace/handle-server &

🔬 Verificación Final
sudo netstat -tulnp | grep '2641\|8000'


✔ El servidor ahora escucha correctamente en:

:::2641
:::8000


Aceptando conexiones desde todas las interfaces (0.0.0.0) y funcionando correctamente.

<h2 align="center">✅ Estado Final: Sistema Operativo y Estable</h2> <p align="center">El Handle Server quedó completamente funcional, y se solucionó la saturación crítica del disco.</p>
