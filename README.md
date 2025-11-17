📜 Bitácora de Diagnóstico y Corrección: DSpace Handle Server (srvdspace)

🗓️ Fecha: 15 de Noviembre de 2025

👤 Técnico: Aldo / Danmer

🖥️ Servidor: srvdspace (Alias WSP01)

📡 Servicio Afectado: Handle System (Prefijo 20.500.13097)

Fase 1: 🔌 Conexión Inicial al Servidor

Se establece la conexión al servidor WSP01 para el diagnóstico inicial.

Identificación de IP (Tailscale): Se identifica la IP de la máquina WSP01 en la red de Tailscale: 100.90.157.8.

Conexión SSH: Se realiza una conexión SSH exitosa al servidor utilizando la IP de Tailscale para obtener acceso a la terminal.

Comando: ssh danmer@100.90.157.8

Verificación de Servidor: Se confirma el acceso al servidor WSP01 (Debian GNU/Linux).

Fase 2: 🚨 Diagnóstico y Resolución de Crisis de Disco

Inmediatamente después de la conexión, se identifica una inestabilidad crítica del sistema causada por la saturación del disco duro principal.

Comando:

df -h


🎯 Diagnóstico: La partición raíz (/dev/mapper/ubuntu--vg-ubuntu--lv) estaba al 100% de su capacidad (93 GB usados de 98 GB).

🔍 Investigación:

Comando:

sudo du -h /dspace --max-depth=1 | sort -hr


🎯 Hallazgo: Se identificó que la carpeta /dspace/log era la principal responsable, consumiendo 41 GB.

🧹 Acción de Limpieza:

Se detuvo el servicio Tomcat: sudo systemctl stop tomcat9.

Se ejecutó una limpieza de logs antiguos (anteriores a agosto de 2025).

Comando:

sudo find /dspace/log -type f -mtime +107 -delete


✅ Resultado: El tamaño de /dspace/log se redujo de 41 GB a 11 GB, liberando ~30 GB. El uso de disco (df -h) se normalizó.

🔄 Acción: Se reinició el servicio Tomcat: sudo systemctl start tomcat9.

Fase 3: 🔧 Diagnóstico del Servicio Handle (Enlace de Red)

Una vez resuelto el problema de disco, la conectividad del Handle Server seguía fallando.

Comando:

sudo netstat -tulnp | grep '2641\|8000'


🎯 Diagnóstico: Se descubrió que el proceso Java del Handle Server (PID 8988) estaba enlazado (escuchando) exclusivamente a la IP privada interna (192.168.0.17).

🛑 Conclusión: Esta configuración impedía que el servidor aceptara conexiones de cualquier otra interfaz (Tailscale, NAT pública).

Fase 4: 🛠️ Corrección del Enlace (Bind Address) y Configuración

Se procedió a corregir la configuración del Handle Server para que aceptara conexiones de todas las interfaces.

🗄️ Backup: Se realizó una copia de seguridad del archivo de configuración binario:

sudo cp /dspace/handle-server/config.dct /dspace/handle-server/config.dct.respaldo

✏️ Edición de Configuración: Se editó el archivo config.dct (o su fuente) para cambiar las tres (3) instancias de bind_address de "192.168.0.17" a "0.0.0.0".

🔑 Re-configuración de Prefijo: Se ejecutó el script de DSpace para asegurar que las llaves y el prefijo (20.500.13097) estuvieran correctamente generados y firmados.

/dspace/bin/make-handle-config

🔄 Reinicio del Servicio:

Se detuvo el proceso Java antiguo (PID 85825) con sudo kill 85825.

Se reinició el servicio manualmente con el comando completo en segundo plano:

nohup java -Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom -classpath /dspace/lib/*:/dspace/config -Ddspace.log.init.disable=true -Dlog4j.configuration=log4j-handle-plugin.properties net.handle.server.Main /dspace/handle-server &


🔬 Verificación Final del Servidor:

Comando:

sudo netstat -tulnp | grep '2641\|8000'


✅ Resultado: ÉXITO. El nuevo proceso Java escucha en :::2641 y :::8000 (modo dual-stack, aceptando IPv4 en 0.0.0.0).

(Fin de las fases de corrección del servidor)
