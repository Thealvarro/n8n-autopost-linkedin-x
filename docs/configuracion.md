# Configuración paso a paso

Esta guía asume que ya tienes n8n funcionando en una dirección `https://`. Si no, empieza por [instalar-n8n.md](instalar-n8n.md).

En todo el documento, `TU-N8N` es la dirección de tu n8n, por ejemplo `n8n.tudominio.com`.

Haz los pasos en orden. No actives el workflow hasta el último paso.

## Paso 1. Importar los dos workflows

Un workflow es una automatización de n8n. Este repositorio trae dos:

- `workflows/autopost-linkedin-x.json`: la automatización principal, con 26 nodos (bloques)
- `workflows/aviso-errores-telegram.json`: te avisa por Telegram cuando algo falla, con 2 nodos

Para cada uno:

1. Descarga el archivo desde GitHub (botón Download raw file).
2. En n8n, crea un workflow nuevo.
3. Abre el menú de los tres puntos arriba a la derecha y elige Import from File.
4. Elige el archivo y guarda.

Verás varios nodos con advertencias. Es normal: les faltan las cuentas y los datos que vas a completar en los pasos siguientes.

## Paso 2. Crear la tabla de temas

Los temas viven en una Data Table, una tabla dentro de n8n. La documentación oficial está en [Data tables](https://docs.n8n.io/build/work-with-data/data-tables/).

1. Descarga `datos/temas-ejemplo.csv`. Un CSV es una tabla guardada como texto, que se abre con cualquier planilla.
2. En tu proyecto de n8n, abre la pestaña Data tables.
3. Usa el botón de arriba a la derecha y elige Create Data table.
4. Ponle de nombre `linkedin_posts`.
5. Elige Import CSV y sube el archivo.
6. Revisa que la tabla tenga las columnas `tema`, `pilar`, `enfoque`, `estado`, `texto`, `fecha_publicacion`, `link_post` y `url_recurso`, todas de tipo texto. Si el importador te deja elegir el tipo de cada columna, elige texto (string) en todas. Si las crea con otro tipo, cámbialo; esto está por confirmar según tu versión de n8n.

n8n agrega solo una columna `id` a cada fila. La automatización la usa para saber qué fila actualizar y para rotar los estilos de imagen.

Los 3 temas del ejemplo son de prueba. Cámbialos por los tuyos, y cambia `https://TU-DOMINIO.com/guia-ejemplo` en la columna `url_recurso` por el link que quieres que vaya al final de cada post.

Ahora conecta la tabla con el workflow. Abre estos 4 nodos y en el campo Data Table elige `linkedin_posts` de la lista:

- `Siguiente tema`
- `Guardar borrador`
- `Marcar publicado`
- `Marcar descartado`

## Paso 3. Crear las credenciales

Una credencial es un acceso guardado dentro de n8n a otro servicio, como tu cuenta de OpenAI o de LinkedIn. Las claves se pegan solo ahí, nunca en el workflow ni en un chat.

En el workflow, cada nodo que usa una cuenta trae una credencial de ejemplo llamada `CONFIGURAR`. Al abrir el nodo, elige la credencial real que creaste. Más abajo, en la tabla de marcadores, está la lista de nodos.

Después de crear cada credencial, compruébala con una llamada real, como se indica en cada sección. El mensaje "Account connected" de n8n puede engañar: dice que la conexión se guardó, no que funcione.

Para hacer esas comprobaciones, agrega un nodo HTTP Request suelto en un workflow de prueba, elige la credencial, pon la dirección que se indica con el método GET y aprieta Execute step. Bórralo cuando termines.

### 3.1 OpenAI

OpenAI es la empresa detrás de los modelos de IA que redactan y dibujan. Necesitas una API key, que es la clave con la que n8n usa tu cuenta.

1. Entra a [platform.openai.com](https://platform.openai.com), carga saldo y crea una API key en la sección API keys.
2. En n8n, crea una credencial de tipo OpenAI y pega la key.
3. Comprobación: GET a `https://api.openai.com/v1/models` con esta credencial (en el nodo HTTP Request, Authentication: Predefined Credential Type, tipo OpenAI). Si devuelve una lista de modelos, funciona.

Para generar imágenes con gpt-image-2, OpenAI puede pedirte verificar tu organización. Si la imagen falla con un error de permisos, revisa eso en la configuración de tu cuenta de OpenAI (por confirmar según tu cuenta).

### 3.2 Telegram

Telegram es el canal por donde te llegan las propuestas. Necesitas un bot de Telegram (una cuenta automática que manda mensajes) y el número que identifica tu chat.

1. En Telegram, abre una conversación con [@BotFather](https://t.me/BotFather), escribe `/newbot` y sigue las instrucciones. Al final te entrega un token, que es la clave del bot.
2. En n8n, crea una credencial de tipo Telegram y pega el token.
3. Abre una conversación con tu bot nuevo y mándale cualquier mensaje.
4. Para saber el id de tu chat, abre en tu navegador `https://api.telegram.org/botTU_TOKEN/getUpdates`, reemplazando `TU_TOKEN` por el token. Busca `"chat":{"id":` y anota ese número. No compartas esa dirección con nadie, porque lleva el token.
5. Comprobación: en un workflow de prueba, crea un nodo Telegram con la operación Send Message, elige esta credencial, pon tu id de chat y un texto cualquiera, y ejecútalo. Si te llega el mensaje, funciona.

Los botones de publicar y descartar de Telegram abren una dirección de tu n8n. Por eso n8n tiene que ser accesible desde internet con `https://`.

### 3.3 LinkedIn

LinkedIn pide una app de desarrollador para dejar que otro programa publique por ti. Una app de desarrollador es un registro que dice qué programa va a usar tu cuenta y con qué permisos.

1. Entra a [linkedin.com/developers](https://www.linkedin.com/developers/apps) y crea una app. LinkedIn exige asociarla a una página de empresa; si no tienes una, crea una página antes.
2. En la pestaña Products de la app, agrega "Share on LinkedIn" y "Sign In with LinkedIn using OpenID Connect".
3. En la pestaña Auth, en Authorized redirect URLs, agrega `https://TU-N8N/rest/oauth2-credential/callback`.
4. Copia el Client ID y el Client Secret de esa misma pestaña.
5. En n8n, crea una credencial de tipo LinkedIn OAuth2 API. Pega el Client ID y el Client Secret. Deja **apagadas** las opciones Legacy y Organization Support. Aprieta Connect y acepta los permisos en la ventana de LinkedIn.
6. Comprobación y dato que necesitas: GET a `https://api.linkedin.com/v2/userinfo` con esta credencial (Authentication: Predefined Credential Type, tipo LinkedIn OAuth2 API). Si devuelve tu nombre, funciona. Anota el valor del campo `sub`: es tu id de persona en LinkedIn y lo vas a usar en el paso 4.

El permiso de LinkedIn dura 60 días. Cuando vence, la publicación falla, te llega un aviso por Telegram y tienes que volver a apretar Connect en esta credencial.

El nodo de LinkedIn que trae n8n usa una versión de la API ya dada de baja. Por eso el workflow usa nodos HTTP Request con el encabezado `LinkedIn-Version: 202606`. Si LinkedIn da de baja esa versión en el futuro, habrá que subir ese número en los nodos `Iniciar subida` y `Publicar en LinkedIn`.

### 3.4 X

X cobra por usar su API: es pago por uso con saldo prepagado, cerca de 0,20 dólares por post con link según sitios de terceros. Una suscripción como X Premium no lo incluye.

1. Entra a [console.x.com](https://console.x.com), crea una app y carga saldo. Te recomendamos dejar la recarga automática apagada: si el saldo llega a cero, X deja de publicar pero no te cobra de más.
2. En la configuración de autenticación de la app, elige permisos "Read and write" y tipo de app "Web App, Automated App or Bot".
3. Como Callback URI pon `https://TU-N8N/rest/oauth2-credential/callback`. Te pedirá también un sitio web; pon el tuyo.
4. Copia el Client ID y el Client Secret de OAuth 2.0.
5. En n8n, crea una credencial de tipo **OAuth2 API** (la genérica, no la de X que trae n8n, porque esa pide más permisos de los necesarios) con estos valores:

| Campo | Valor |
|---|---|
| Grant Type | PKCE |
| Authorization URL | `https://x.com/i/oauth2/authorize` |
| Access Token URL | `https://api.x.com/2/oauth2/token` |
| Client ID | el que copiaste |
| Client Secret | el que copiaste |
| Scope | `tweet.read tweet.write users.read offline.access` |
| Authentication | Header |

6. Aprieta Connect y acepta en la ventana de X.
7. Comprobación: GET a `https://api.x.com/2/users/me` con esta credencial (Authentication: Generic Credential Type, tipo OAuth2 API). Si devuelve tu usuario de X, funciona.

Si no quieres publicar en X, borra el nodo `Publicar en X` y salta esta sección. El texto corto igual se generará y te llegará a Telegram; para no gastar en él, borra también `Redactar para X` y `Armar texto X`, conecta `Armar texto` directo con `Guardar borrador` y quita la parte "Versión para X" del mensaje del nodo `Aprobar en Telegram`.

## Paso 4. Reemplazar los marcadores

Un marcador es un texto de ejemplo que tienes que cambiar por tu dato real. Esta es la lista completa de los que trae el repositorio:

| Marcador | Dónde está | Con qué se reemplaza |
|---|---|---|
| Credencial `CONFIGURAR` (OpenAI) | `Redactar post`, `Redactar para X`, `Director de arte`, `Imagen con persona`, `Imagen sin persona` | Tu credencial de OpenAI |
| Credencial `CONFIGURAR` (Telegram) | `Enviar imagen`, `Aprobar en Telegram` y, en el workflow de errores, `Avisar por Telegram` | Tu credencial de Telegram |
| Credencial `CONFIGURAR` (LinkedIn) | `Iniciar subida`, `Subir imagen`, `Publicar en LinkedIn` | Tu credencial LinkedIn OAuth2 API |
| Credencial `CONFIGURAR` (X) | `Publicar en X` | Tu credencial OAuth2 API de X |
| Data Table vacía | `Siguiente tema`, `Guardar borrador`, `Marcar publicado`, `Marcar descartado` | La tabla `linkedin_posts` |
| `TU_CHAT_ID_DE_TELEGRAM` | Campo Chat ID de `Enviar imagen`, `Aprobar en Telegram` y, en el workflow de errores, `Avisar por Telegram` | El id de tu chat (paso 3.2) |
| `TU_ID_DE_LINKEDIN` | Dentro del cuerpo JSON de `Iniciar subida` y de `Publicar en LinkedIn`, en `urn:li:person:TU_ID_DE_LINKEDIN` | El valor `sub` de tu LinkedIn (paso 3.3). Cambia solo esa parte y deja `urn:li:person:` |
| `[TU NOMBRE]` | Instrucciones de `Redactar post`, `Redactar para X` y `Director de arte` | Tu nombre |
| `[A QUÉ TE DEDICAS]` | Instrucciones de `Redactar post`, `Redactar para X` y `Director de arte` | Tu oficio, por ejemplo "diseñadora web" |
| `[TU SITIO]` | Instrucciones de `Redactar post`, `Redactar para X` y `Director de arte` (en este último aparece dos veces, la segunda es el texto que se dibuja en la esquina de la imagen) | Tu sitio web o tu marca |
| `[DESCRIBE CÓMO ERES: edad aproximada, pelo, barba, ropa habitual]` | Instrucciones de `Director de arte` | Una descripción corta de ti, para que la imagen se parezca |
| `https://TU-DOMINIO.com/fotos-referencia/` | Código del nodo `Descargar fotos` | La dirección de la carpeta donde subiste tus fotos (paso 5), terminada en `/` |
| `https://TU-DOMINIO.com/guia-ejemplo` | Columna `url_recurso` de la tabla (viene del CSV) | El link que va al final de cada post |

Hay otros textos que no son marcadores pero conviene revisar:

- las instrucciones del `Director de arte` dicen "el hombre de las fotos de referencia", y varios estilos del nodo `Elegir estilo` dicen "la persona sentado". Ajústalos si no te representan
- las instrucciones de los nodos de texto mencionan jerga chilena que la IA debe evitar; ajústalas a tu país
- la zona horaria del workflow es `America/Santiago` (en la configuración del workflow, Settings) y también aparece en el nodo `Marcar publicado`. Cámbiala por la tuya, por ejemplo `America/Mexico_City` o `Europe/Madrid`

Una forma rápida de no olvidar nada: descarga el workflow ya editado y busca en el archivo las palabras `TU_`, `TU-DOMINIO`, `[TU` y `CONFIGURAR`. No debería quedar ninguna.

## Paso 5. Subir las fotos de referencia

Los estilos que llevan persona usan 4 fotos tuyas para que la imagen se parezca a ti.

1. Elige 4 fotos con tu cara bien visible, con buena luz y de frente o tres cuartos.
2. **Quítales los datos de ubicación GPS.** Las fotos del celular suelen guardar el lugar exacto donde se tomaron, y estas fotos quedan públicas. Expórtalas sin metadatos: por ejemplo, en Mac con Vista Previa (Herramientas, Mostrar inspector, pestaña de ubicación) o con la opción de quitar la ubicación al compartir desde el celular. Revisa el resultado antes de subirlas.
3. Renómbralas exactamente `ref1.jpg`, `ref2.jpg`, `ref3.jpg` y `ref4.jpg`. Tienen que ser JPG.
4. Súbelas a una carpeta de tu sitio web o de tu hosting, con un nombre difícil de adivinar.
5. Comprueba que se abren en el navegador, por ejemplo `https://TU-DOMINIO.com/tu-carpeta/ref1.jpg`.
6. Pon esa dirección de carpeta en el nodo `Descargar fotos` (paso 4).

Si no quieres aparecer en las imágenes, en el nodo `Elegir estilo` cambia `persona: true` por `persona: false` en todos los estilos. Así nunca se usan las fotos.

## Paso 6. Conectar el workflow de errores

1. Guarda y activa el workflow `Aviso de errores (Telegram)`. Este sí se activa desde el principio, porque solo actúa cuando otro workflow falla.
2. Abre el workflow principal, entra a su configuración (Settings) y en Error Workflow elige `Aviso de errores (Telegram)`.
3. Guarda.

## Paso 7. Probar a mano

El workflow trae un nodo `Probar a mano` para correrlo sin esperar el horario.

1. Revisa que la tabla tenga al menos un tema en `pendiente`.
2. En el workflow principal, aprieta Execute workflow (el botón del nodo `Probar a mano`).
3. Mira cómo se pinta en verde cada nodo. La imagen puede tardar un par de minutos.
4. Te debe llegar a Telegram la imagen y después un mensaje con el post para LinkedIn, la versión para X y los dos botones.
5. Para la primera prueba, aprieta **Descartar**. Revisa que el tema quede como `descartado` en la tabla. Así compruebas todo sin publicar.
6. Cuando estés conforme, vuelve el tema a `pendiente`, corre otra prueba y aprieta **Publicar**. Revisa que salga el post en LinkedIn, que el tema quede `publicado` con su fecha y link, y que salga el post en X.

Si un nodo se pone rojo, ábrelo y lee el error. Los más comunes:

- error 401 o "unauthorized": la credencial no funciona o venció. Vuelve a conectarla
- error al leer el brief en `Leer brief`: la IA no devolvió un formato válido. Vuelve a probar; si se repite, revisa las instrucciones del `Director de arte`
- error en `Descargar fotos`: alguna foto no se encuentra en la dirección que pusiste
- error en `Publicar en X` por saldo: carga saldo en console.x.com

## Paso 8. Activar

Cuando la prueba completa haya funcionado, activa el workflow principal con el interruptor de arriba. Desde ahí corre solo cada 2 días a las 10:00, en la zona horaria del workflow.

La propuesta queda esperando tu respuesta en Telegram hasta 46 horas. Recuerda que el permiso de LinkedIn vence cada 60 días.
