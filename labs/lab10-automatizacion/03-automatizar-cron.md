# Laboratorio 10 — Automatización

## Programar la renovación de certificados con cron

En entornos donde no se utiliza un gestor automático como certmonger, es habitual implementar **scripts de renovación ejecutados periódicamente mediante cron**.

En este ejercicio automatizaremos el proceso de renovación del certificado utilizando el script creado anteriormente.

El objetivo es:

* ejecutar periódicamente la renovación
* registrar el resultado
* recargar el servicio que utiliza el certificado.

---

### Paso 1 — Ir al directorio del servicio

Sitúate en el directorio donde se encuentra el script de renovación.

```bash
cd ~/pki-labs/web-server
```

Comprueba que el script existe.

```bash
ls renew-cert.sh
```

Este script será ejecutado automáticamente por cron.

---

### Paso 2 — Crear un script de renovación completo

Crea un nuevo script que ejecute la renovación y recargue el servicio.

```bash
nano renew-and-reload.sh
```

Añade el siguiente contenido:

```bash
#!/bin/bash

SERVICE_DIR=~/pki-labs/web-server
LOGFILE=~/pki-labs/web-server/renew.log

echo "---- $(date) ----" >> $LOGFILE

$SERVICE_DIR/renew-cert.sh >> $LOGFILE 2>&1

echo "Recargando servicio TLS..." >> $LOGFILE

docker restart $(docker ps -q --filter ancestor=nginx) >> $LOGFILE 2>&1

echo "Proceso finalizado" >> $LOGFILE
```

Guarda el archivo.

Este script:

* ejecuta la renovación del certificado
* registra la salida en un log
* reinicia el servidor NGINX para aplicar el nuevo certificado.

---

### Paso 3 — Hacer ejecutable el script

Cambia los permisos del archivo.

```bash
chmod +x renew-and-reload.sh
```

Comprueba que el script funciona.

```bash
./renew-and-reload.sh
```

Consulta el archivo de log.

```bash
cat renew.log
```

---

### Paso 4 — Crear una tarea programada con cron

> **En Codespaces y contenedores: cron no está instalado ni corriendo**
>
> Los entornos de GitHub Codespaces y muchos contenedores Docker no incluyen
> `cron` por defecto. Si `crontab` no se encuentra, instálalo y arráncalo
> manualmente:
>
> ```bash
> # Instalar cron
> sudo apt update && sudo apt install -y cron
>
> # Arrancar el demonio cron (no hay systemd en Codespaces)
> sudo cron
> ```
>
> Comprueba que está corriendo:
>
> ```bash
> ps aux | grep cron
> ```
>
> Si no ves un proceso `/usr/sbin/cron`, vuelve a ejecutar `sudo cron`.
>
> Ten en cuenta que en Codespaces el demonio cron **no sobrevive al reinicio
> del contenedor**. Si recreas el Codespace, tendrás que arrancarlo de nuevo.

Edita la tabla de tareas programadas.

```bash
crontab -e
```

Añade la siguiente línea al final del archivo.

```bash
0 2 * * * /home/codespace/pki-labs/web-server/renew-and-reload.sh
```

Esta configuración ejecutará el script **cada día a las 02:00**.

Guarda el archivo y sal del editor.

---

### Paso 5 — Verificar la tarea programada

Comprueba las tareas registradas.

```bash
crontab -l
```

Deberías ver la entrada creada.

Esto confirma que el sistema ejecutará automáticamente el proceso de renovación según la planificación establecida.

---

Este tipo de automatización permite mantener certificados actualizados sin intervención manual y es una práctica común en infraestructuras donde múltiples servicios dependen de TLS.
