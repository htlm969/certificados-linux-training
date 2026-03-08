# Laboratorio 08 — TLS en servicios

## Inspeccionar la conexión TLS del servidor

En el ejercicio anterior iniciamos un servidor HTTPS utilizando NGINX y el certificado emitido por nuestra PKI.
Ahora analizaremos la conexión TLS que establece el cliente con ese servidor.

Utilizaremos herramientas como **OpenSSL** y **curl** para observar qué certificados se envían y qué parámetros criptográficos se negocian.

Asegúrate de tener el servidor NGINX del ejercicio anterior en marcha. Los comandos de este ejercicio los ejecutas en la terminal del Codespace (en el host), no dentro de ningún contenedor; OpenSSL y curl se conectan a `localhost:8443` donde escucha NGINX.

---

### Paso 1 — Conectarse al servidor utilizando OpenSSL

Utiliza OpenSSL para establecer una conexión TLS con el servidor que ejecutamos en Docker.

```bash
openssl s_client -connect localhost:8443
```

La salida mostrará el proceso completo de negociación TLS.

Busca líneas similares a:

```
CONNECTED
SSL handshake has read ...
```

Estas líneas indican que el cliente y el servidor han completado el handshake TLS.

---

### Paso 2 — Identificar el certificado presentado por el servidor

Dentro de la salida del comando anterior localiza las líneas:

```
subject=
issuer=
```

El **subject** debería coincidir con el nombre que utilizaste al generar el certificado.

El **issuer** debería corresponder a la autoridad intermedia que firmó el certificado del servidor.

Esto confirma que el servidor está utilizando el certificado generado en los laboratorios anteriores.

---

### Paso 3 — Observar la cadena de certificados enviada

Desplázate por la salida hasta encontrar la sección:

```
Certificate chain
```

Debajo aparecerán varios certificados.

Normalmente el servidor envía:

* el certificado del servidor
* el certificado de la autoridad intermedia.

Esto permite al cliente reconstruir la cadena de confianza.

---

### Paso 4 — Identificar el protocolo TLS y el cifrado negociado

En la parte inferior de la salida busca líneas similares a:

```
Protocol  : TLSv1.3
Cipher    : TLS_AES_256_GCM_SHA384
```

Estas líneas indican:

* la versión del protocolo TLS utilizada
* el algoritmo de cifrado negociado.

El cliente y el servidor acuerdan estos parámetros durante el handshake TLS.

---

### Paso 5 — Comparar la conexión utilizando curl

Ahora realiza la misma conexión utilizando `curl`.

```bash
curl -v https://localhost:8443
```

Observa en la salida líneas similares a:

```
SSL connection using TLSv1.3
Server certificate:
```

Esto confirma que `curl` también establece una conexión TLS con el servidor.

Si el certificado no está en el almacén de confianza del sistema, `curl` mostrará un error de verificación.

Puedes forzar la conexión utilizando:

```bash
curl -k https://localhost:8443
```

Esto ignora la verificación del certificado y permite observar el comportamiento del servidor HTTPS.

---

En este ejercicio hemos analizado una conexión TLS real con el servidor configurado en el laboratorio anterior y hemos observado cómo se presentan los certificados y se negocian los parámetros criptográficos.
