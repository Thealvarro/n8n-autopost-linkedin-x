# Instalar n8n

## Qué es n8n

n8n es un programa para crear automatizaciones uniendo bloques en pantalla, sin escribir código. Cada automatización se llama workflow y cada bloque se llama nodo. Un nodo puede, por ejemplo, leer una tabla, pedirle un texto a una IA o mandar un mensaje a Telegram.

Para esta automatización, n8n tiene que estar encendido todo el día y accesible desde internet con una dirección propia y segura (que empiece con `https://`). Hay dos razones: el horario de las 10:00 solo se dispara si n8n está funcionando, y los botones de Telegram, LinkedIn y X necesitan poder volver a tu n8n.

Esta automatización se probó en n8n 1.123.92, en la versión Community (gratis), instalada por cuenta propia (en inglés, self-hosted) con Node 22. Node es el motor que necesita n8n para funcionar. Usa las Data Tables de n8n, que son tablas que viven dentro del mismo n8n. Si tu versión es mucho más antigua, puede que no las tenga.

## Tres formas de tenerlo

### 1. La nube de n8n

n8n te lo deja funcionando y tú solo entras con usuario y contraseña. Es lo más fácil y no tienes que mantener nada. Es de pago mensual: parte cerca de 20 euros al mes según sitios de terceros. Revisa el precio actual en [n8n.io/pricing](https://n8n.io/pricing).

Si eliges esta opción, salta directo a [configuracion.md](configuracion.md).

### 2. Un VPS

Un VPS es un servidor virtual que arriendas por mes. Es como un computador en internet que es solo tuyo. Hay opciones baratas (el precio exacto depende del proveedor y está por confirmar), pero tienes que instalar y mantener n8n tú mismo: actualizaciones, respaldos y seguridad.

La documentación oficial de n8n tiene guías para esto:

- [Opciones de instalación](https://docs.n8n.io/deploy/host-n8n/install-options/), para ver todas las formas
- [Instalar con Docker Compose](https://docs.n8n.io/deploy/host-n8n/install-options/install-using-docker-compose/), la forma que la documentación recomienda. Docker es un programa que empaqueta n8n con todo lo que necesita para funcionar
- [Usar un proveedor de nube](https://docs.n8n.io/deploy/host-n8n/install-options/use-a-cloud-provider/), con guías para DigitalOcean, Hetzner, AWS, Google Cloud y otros

Según la documentación oficial, Docker Compose pide al menos 4 GB de memoria y 2 procesadores virtuales. Revisa los requisitos al momento de contratar.

### 3. Un hosting compartido con cPanel

Un hosting compartido es un plan donde tu sitio comparte el servidor con otros clientes. cPanel es el panel de control que traen muchos de estos planes. El autor usa esta opción porque ya pagaba un hosting así para su sitio web, y de esta forma no paga la mensualidad de la nube de n8n.

No todos los planes lo permiten. Tu hosting tiene que dejarte correr aplicaciones de Node desde cPanel (suele aparecer como "Setup Node.js App") y ofrecer Node 22. Pregúntale a tu proveedor antes de empezar.

La documentación oficial de n8n no tiene una guía para cPanel. La base es la [instalación con npm](https://docs.n8n.io/deploy/host-n8n/install-options/install-with-npm/). npm es el programa que instala paquetes de Node. Según esa página, n8n necesita una versión de Node entre la 20.19 y la 24, y la instalación con npm deja de estar disponible desde n8n 3.0. Tenlo en cuenta si piensas actualizar en el futuro.

Los pasos exactos dentro de cPanel (cómo crear la aplicación de Node, qué archivo de inicio poner y qué variables de entorno configurar) dependen de cada proveedor y están por confirmar. Lo que sí está comprobado:

- Desde n8n 1.123.51 hace falta Node 22, aunque el paquete diga que basta con Node 20.19. Si n8n no arranca después de actualizar, revisa primero la versión de Node.
- Antes de instalar, actualizar o reiniciar n8n, respalda su base de datos y su configuración. Con la configuración por defecto, n8n guarda todo en una carpeta `.n8n` dentro de la carpeta de tu usuario, en un archivo llamado `database.sqlite`. Si se pierde la configuración, también se pierde la llave con que n8n cifra tus credenciales.

## Lección real: dos propuestas iguales

Esto le pasó al autor en un hosting compartido con cPanel (servidor LiteSpeed con CloudLinux, n8n corriendo como aplicación de Node).

**Lo que se ve:** a las 10:00 llegan dos propuestas a Telegram con pocos segundos de diferencia, del mismo tema.

**La causa:** después de un rato sin visitas, el servidor suelta el proceso de n8n pero no lo apaga. Un proceso es un programa en funcionamiento. La siguiente visita levanta un proceso nuevo, y quedan dos copias de n8n funcionando a la vez sobre la misma base de datos. Cada copia dispara el horario por su cuenta, y por eso llegan dos propuestas. Una de las copias puede incluso tener una versión antigua del workflow.

**Cómo revisarlo:** desde la Terminal de cPanel, con este comando, que solo muestra información y no cambia nada:

```bash
ps -u TU_USUARIO -o pid,ppid,lstart,cmd | grep -i lsnode
```

`lsnode` es el nombre que tienen los procesos de Node en estos servidores. Si aparece más de uno de n8n, hay copias de sobra. Un proceso sano tiene como "padre" (la columna `ppid`) al servidor LiteSpeed. Uno abandonado tiene `ppid` igual a 1.

**El arreglo que se aplicó:**

1. Respaldar la base de datos y la configuración de n8n.
2. Revisar que no haya ejecuciones esperando respuesta (por ejemplo, una propuesta sin contestar en Telegram), porque se cortarían.
3. Detener todos los procesos de n8n, confirmar con el comando de arriba que no queda ninguno y abrir la dirección de n8n en el navegador para que arranque uno solo.
4. Agregar un cron que visite n8n cada 5 minutos, para que nunca pase un rato sin tráfico. Un cron es una tarea programada del servidor; en cPanel está en la sección "Cron Jobs". La línea es:

```bash
*/5 * * * * curl -fsS -m 25 -o /dev/null https://TU-N8N/healthz
```

Reemplaza `TU-N8N` por la dirección de tu n8n. `/healthz` es una dirección de n8n que solo responde si está funcionando.

**Estado:** este arreglo está en observación. Al momento de escribir esto todavía no está confirmado que baste para que no vuelva a pasar. Si vuelve a pasar, el paso siguiente sería una tarea que revise los procesos y deje vivo solo el más nuevo.

Mientras tanto, si te llegan dos propuestas iguales: primero descarta una y después publica la otra.

Si usas la nube de n8n o un VPS con Docker, este problema no debería aparecer, porque ahí n8n corre como un solo servicio que no se apaga por falta de visitas.

## Antes de seguir

Cuando tengas n8n funcionando, comprueba esto:

- entras a tu n8n con una dirección `https://`
- al abrir tu proyecto en n8n aparece la pestaña Data tables
- la zona horaria de n8n es la tuya. El workflow trae `America/Santiago`; se cambia en la configuración del workflow

Después sigue con [configuracion.md](configuracion.md).
