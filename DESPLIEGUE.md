# DESPLIEGUE — Evidencias y respuestas

Este documento recopila todas las evidencias y respuestas de la practica.

---

## Parte 1 — Evidencias minimas

### Fase 1: Instalacion y configuracion

1) Servicio Nginx activo
- Que demuestra: El funcionamiento de los contenedores y del servicio del propio contenedor de servidor-nginx.
- Comando: `docker compose ps` y `docker exec servidor-nginx service nginx status`
- Evidencia: ![](evidencias/Fase1-1a.png) ![](evidencias/Fase1-1b.png)

2) Configuracion cargada
- Que demuestra: Que la configuración ha sido cargada correctamente.
- Comando: `docker exec servidor-nginx ls -l /etc/nginx/conf.d/`
- Evidencia: ![](evidencias/Fase1-2.png)

3) Resolucion de nombres
- Que demuestra: Que se le ha asignado un nombre/alias a la dirección en el archivo /etc/hosts.
- Evidencia: ![](evidencias/Fase1-34.png)

4) Contenido Web
- Que demuestra: Que la web requerida ha sido cargado correctamente.
- Evidencia: ![](evidencias/Fase1-34.png)

### Fase 2: Transferencia SFTP (Filezilla)

5) Conexion SFTP exitosa
- Que demuestra: La conexión exitosa mediante FileZilla.
- Evidencia: ![](evidencias/Fase2-1.png)

6) Permisos de escritura
- Que demuestra: Los permisos de escritura para poder subir archivos.
- Evidencia: ![](evidencias/Fase2-2.png)

### Fase 3: Infraestructura Docker

7) Contenedores activos
- Que demuestra: La actividad de los contenedores.
- Comando: `docker compose ps`
- Evidencia: ![](evidencias/Fase3-1.png)

8) Persistencia (Volumen compartido)
- Que demuestra: La funcionalidad del volumen, mostrando la web activa mientras que se ve FileZilla y los archivos subidos anteriormente.
- Evidencia: ![](evidencias/Fase3-2.png)

9) Despliegue multi-sitio
- Que demuestra: El funcionamiento de la página /reloj.
- Evidencia: ![](evidencias/Fase3-3.png)

### Fase 4: Seguridad HTTPS

10) Cifrado SSL
- Que demuestra: Que la página es accesible mediante HTTPS.
- Evidencia: ![](evidencias/Fase4-1a.png) ![](evidencias/Fase4-1b.png)

11) Redireccion forzada
- Que demuestra: Que tras intentar acceder mediante HTTP automaticamente redirige a la página de manera HTTPS
- Evidencia: ![](evidencias/Fase4-2.png)

---

## Parte 2 — Evaluacion RA2 (a–j)

### a) Parametros de administracion

He identificado los parámetros principales del nginx.conf como worker_processes, worker_connections, access_log, error_log y keepalive_timeout. Modifiqué el keepalive_timeout de 65 a 30 segundos en el default.conf para que las conexiones no se queden abiertas tanto tiempo.

- Evidencias:

![](evidencias/a-01-grep-nginxconf.png)
![](evidencias/a-02-nginx-t.png)
![](evidencias/a-03-reload.png)

### b) Ampliacion de funcionalidad + modulo investigado
- Opción elegida: B1 (Gzip)

He activado la compresión gzip para reducir el tamaño de las respuestas HTTP. He creado un archivo gzip.conf con las directivas necesarias y lo monté como volumen en el docker-compose. Esto hace que archivos se compriman antes de enviarse al navegador.

- Evidencias:

![](evidencias/b1-01-gzipconf.png)
![](evidencias/b1-02-compose-volume-gzip.png)
![](evidencias/b1-03-nginx-t.png)
![](evidencias/b1-04-curl-gzip.png)

#### Modulo investigado: ModSecurity WAF

**Para qué sirve:** Firewall de aplicaciones web (WAF) que protege contra ataques como SQL injection, XSS, y otras vulnerabilidades OWASP. Monitoriza y analiza tráfico HTTP en tiempo real.

**Cómo se instala/carga:** Principalmente por compilación desde código fuente (requiere dependencias como libcurl, libxml2, libyajl). Se integra como módulo dinámico con `load_module modules/ngx_http_modsecurity_module.so;` y se activa con `modsecurity on;` en la configuración del server.

**Fuentes:**
- https://blog.nginx.org/blog/compiling-and-installing-modsecurity-for-open-source-nginx
- https://github.com/owasp-modsecurity/ModSecurity
- https://www.linuxbabe.com/security/modsecurity-nginx-debian-ubuntu

### c) Sitios virtuales / multi-sitio

Tengo configurados dos sitios: la web principal en / y el reloj en /reloj. El multi-sitio por path sirve contenido diferente según la ruta, mientras que el multi-sitio por nombre usa diferentes server_name para cada web. Con location / sirvo la web principal y con location /reloj sirvo la aplicación del reloj usando alias.

- Evidencias:

![](evidencias/c-01-root.png)
![](evidencias/c-02-reloj.png)
![](evidencias/c-03-defaultconf-inside.png)

### d) Autenticacion y control de acceso

Protegí la ruta /admin/ con autenticación básica HTTP. Generé un archivo .htpasswd con el usuario admin y configuré auth_basic en el location /admin/ del default.conf. Sin credenciales devuelve 401, con credenciales correctas devuelve 200.

- Evidencias:

![](evidencias/d-01-admin-html.png)
![](evidencias/d-02-defaultconf-auth.png)
![](evidencias/d-03-curl-401.png)
![](evidencias/d-04-curl-200.png)

### e) Certificados digitales

El archivo .crt contiene el certificado público y el .key la clave privada. Usé -nodes para no cifrar la clave privada con contraseña, lo cual es útil en laboratorio pero no recomendable en producción. Los certificados se generaron con openssl y se montaron como volúmenes en el docker-compose.

- Evidencias:

![](evidencias/e-01-ls-certs.png)
![](evidencias/e-02-compose-certs.png)
![](evidencias/e-03-defaultconf-ssl.png)

### f) Comunicaciones seguras

Configuré HTTPS en el puerto 443 con los certificados SSL. Usé dos server blocks: uno escucha en el puerto 80 y redirige con 301 a HTTPS, y otro escucha en 443 y sirve el contenido. Esto asegura que todo el tráfico se cifre.

- Evidencias:

![](evidencias/f-01-https.png)
![](evidencias/f-02-301-network.png)

### g) Documentación

Este documento DESPLIEGUE.md contiene la documentación completa de la infraestructura: arquitectura con servicios nginx y sftp, configuración de sitios virtuales, seguridad con HTTPS y autenticación, y análisis de logs. Todas las evidencias están enlazadas en sus respectivos apartados.

### h) Ajustes para implantación de apps

Para desplegar la app del reloj en /reloj usé alias en lugar de root para que nginx busque los archivos en la subcarpeta correcta. El problema típico con SFTP es que los archivos suben con permisos incorrectos, la solución es configurar bien el usuario en el contenedor sftp para que coincida con el del volumen.

- Evidencias:

![](evidencias/h-01-root.png)
![](evidencias/h-02-reloj.png)

### i) Virtualización en despliegue

Con contenedores Docker puedo desplegar nginx de forma rápida y reproducible, sin tener que instalar nada en el sistema operativo. Los volúmenes permiten modificar la configuración y el contenido sin entrar al contenedor. Si necesito cambiar algo borro el contenedor y lo vuelvo a crear sin perder datos.

- Evidencias:

![](evidencias/i-01-compose-ps.png)

### j) Logs: monitorización y análisis

Los logs de nginx muestran todas las peticiones HTTP que recibe el servidor. Con docker compose logs puedo ver los logs en tiempo real y analizar el tráfico. Se pueden ver las URLs más visitadas, los códigos de estado y los errores 404 para saber qué archivos faltan.

- Evidencias:

![](evidencias/j-01-logs-follow.png)
![](evidencias/j-02-metricas.png)

---

## Checklist final

### Parte 1
- [x] 1) Servicio Nginx activo
- [x] 2) Configuracion cargada
- [x] 3) Resolucion de nombres
- [x] 4) Contenido Web (Cloud Academy)
- [x] 5) Conexion SFTP exitosa
- [x] 6) Permisos de escritura
- [x] 7) Contenedores activos
- [x] 8) Persistencia (Volumen compartido)
- [x] 9) Despliegue multi-sitio (/reloj)
- [x] 10) Cifrado SSL
- [x] 11) Redireccion forzada (301)

### Parte 2 (RA2)
- [x] a) Parametros de administracion
- [x] b) Ampliacion de funcionalidad + modulo investigado
- [x] c) Sitios virtuales / multi-sitio
- [x] d) Autenticacion y control de acceso
- [x] e) Certificados digitales
- [x] f) Comunicaciones seguras
- [x] g) Documentacion
- [x] h) Ajustes para implantacion de apps
- [x] i) Virtualizacion en despliegue
- [x] j) Logs: monitorizacion y analisis
