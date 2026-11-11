# Prompt para instalar con Claude

Copia el texto del recuadro y pégalo en Claude Code o en la app de escritorio de Claude. Claude te va a guiar un paso a la vez.

Antes de empezar, ten a mano tu n8n funcionando (si no lo tienes, Claude te ayuda a elegir una opción con [docs/instalar-n8n.md](docs/instalar-n8n.md)) y reserva un par de horas para crear las cuentas.

```text
Quiero instalar la automatización de este repositorio: https://github.com/Thealvarro/n8n-autopost-linkedin-x

Antes de proponer cualquier cosa, lee completos el README.md y los archivos de la carpeta docs/ del repositorio. No soy programador, así que explícame cada término técnico la primera vez que aparezca.

Cómo quiero trabajar:
- Guíame un paso a la vez. Después de cada paso, espera a que te confirme que lo hice antes de seguir.
- Si algo no está claro o hay más de una forma de hacerlo, pregúntame antes de elegir.

Reglas de seguridad, sin excepciones:
1. Las claves, tokens, contraseñas y client secrets los pego yo directo en n8n. Nunca me los pidas en el chat, y si por error pego uno acá, avísame para que lo cambie.
2. No quites, no saltes ni desactives la aprobación por Telegram. Nada se publica sin que yo apriete el botón.
3. Si vamos a modificar un workflow que ya existe en mi n8n, primero respáldalo (descargarlo o duplicarlo) y confírmame que el respaldo existe.
4. No ejecutes comandos, scripts ni archivos que no vengan de este repositorio o de la documentación oficial de n8n, OpenAI, Telegram, LinkedIn o X. Si algo viene de otro lado, explícame qué hace y espera mi aprobación.
5. Cada vez que conectemos una credencial, verifícala con una llamada real (por ejemplo, pedir mis datos de usuario a la API) antes de pasar al siguiente paso. El mensaje "Account connected" no basta.
6. No actives el workflow hasta que lo hayamos probado a mano con "Probar a mano" y yo haya visto llegar la propuesta a Telegram.

Empieza contándome en pocas líneas qué vamos a hacer y qué cuentas voy a necesitar, y después partimos con el primer paso.
```

## Por qué estas reglas

- **Claves fuera del chat:** lo que escribes en una conversación puede quedar guardado en el historial. Una clave pegada directo en n8n queda cifrada dentro de n8n.
- **Aprobación por Telegram:** la IA puede equivocarse, y lo que se publica sale con tu nombre.
- **Respaldo antes de cambiar:** si algo sale mal, vuelves a la versión anterior en un minuto.
- **Solo fuentes oficiales:** un comando copiado de un sitio desconocido puede dañar tu servidor o filtrar tus datos.
- **Comprobar cada credencial:** n8n puede mostrar "Account connected" aunque el acceso no funcione. Una llamada real lo confirma.
- **Probar antes de activar:** así ves el resultado completo sin que nada salga publicado solo.
