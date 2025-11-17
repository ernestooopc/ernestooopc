📜 Bitácora de Diagnóstico y Corrección
DSpace Handle Server – Servidor srvdspace (WSP01)

🗓️ Fecha: 15 de Noviembre de 2025
👤 Técnico: Aldo / Danmer
📡 Servicio afectado: Handle System (Prefijo 20.500.13097)

🧩 Fase 1: Conexión Inicial al Servidor

Identificación de IP Tailscale:

100.90.157.8


Conexión SSH al servidor:

ssh danmer@100.90.157.8


Resultado:
✔ Acceso correcto al servidor WSP01 (Debian GNU/Linux).

🚨 Fase 2: Diagnóstico y Resolución de Crisis de Disco

Verificación del uso de disco:

df -h


Diagnóstico:
La partición raíz estaba al 100% de capacidad:
/dev/mapper/ubuntu--vg-ubuntu--lv → 93 GB usados de 98 GB.

🔎 Investigación de directorios pesados
sudo du -h /dspace --max-depth=1 | sort -hr


✔ Se encontró que /dspace/log ocupaba 41 GB.

🧹 Limpieza de logs antiguos

Detener Tomcat:

sudo systemctl stop tomcat9


Eliminar logs anteriores a agosto 2025:

sudo find /dspace/log -type f -mtime +107 -delete


Verificación:
✔ /dspace/log redujo de 41 GB → 11 GB
✔ Se liberaron ~30 GB.

Reiniciar Tomcat:

sudo systemctl start tomcat9

🔧 Fase 3: Diagnóstico del Servicio Handle (Problema de Enlace)

Verificación de puertos:

sudo netstat -tulnp | grep '2641\|8000'


Diagnóstico:
El proceso Java del Handle Server estaba escuchando solo en la IP interna:

192.168.0.17


🛑 Esto impedía conexiones externas (Tailscale / IP pública).

🛠️ Fase 4: Corrección del Bind Address del Handle Server
📂 Backup de configuración
sudo cp /dspace/handle-server/config.dct /dspace/handle-server/config.dct.respaldo

✏️ Ajuste de configuración

Se editaron las 3 instancias de:

bind_address = "192.168.0.17"


por:

bind_address = "0.0.0.0"

🔑 Reconfiguración del prefijo
/dspace/bin/make-handle-config

🔄 Reinicio del servidor Handle

Detener el proceso antiguo:

sudo kill 85825


Levantar el Handle Server nuevamente:

nohup java -Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom \
-classpath /dspace/lib/*:/dspace/config -Ddspace.log.init.disable=true \
-Dlog4j.configuration=log4j-handle-plugin.properties \
net.handle.server.Main /dspace/handle-server &

🔬 Verificación final
sudo netstat -tulnp | grep '2641\|8000'


Resultado:
✔ Proceso Java escuchando correctamente en:

:::2641
:::8000


Aceptando conexiones IPv4/IPv6 y todas las interfaces (0.0.0.0).

Estado Final:

El Handle Server quedó operativo en todas las redes (Tailscale, LAN y pública).
El servidor recuperó su estabilidad tras liberar espacio en disco.
