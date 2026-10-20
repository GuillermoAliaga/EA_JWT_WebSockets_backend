REGISTRO DEL USO DE IA DURANTE EL DESARROLLO DEL EJERCICIO

1. Inicio del ejercicio y metodología de trabajo

Prompt: Pegué el enunciado del ejercicio y pedí que no me proporcionara el código resuelto, sino únicamente las indicaciones necesarias para intentarlo por mi cuenta.

Problema: No sabía por dónde empezar y, después de intentarlo, seguía bastante perdido.

Solución: La IA me explicó la idea general de la autenticación mediante tokens: mantener el access token en memoria y enviar el refresh token mediante una cookie. Después, propuso dividir el ejercicio en partes y comenzar por el login del backend. Antes de modificar el código, tuve que localizar los ficheros correspondientes del repositorio, concretamente AuthService y controllers/Auth.ts.


2. Funcionamiento de las cookies y envío del refresh token

Prompts: "¿Cómo se haría y explica cómo funcionan las cookies?" y "Quiero que el refreshToken se envíe mediante cookie, pero si lo hago así lo enviaría dos veces, porque va en la cookie y también dentro del result".

Problema: No entendía completamente el funcionamiento de las cookies ni cómo evitar que el refresh token se enviara tanto en la cookie como en el cuerpo de la respuesta. También tuve dudas al configurar la opción path.

Solución: Aprendí que el servidor envía la cookie mediante Set-Cookie, el navegador la almacena y la reenvía en las peticiones que cumplen sus condiciones. También entendí la función de opciones como httpOnly, sameSite, secure, path y maxAge.

En el login, el refresh token se envía mediante res.cookie(...), mientras que el cuerpo de la respuesta contiene únicamente el access token y los datos del usuario. Además, corregí la configuración de path para utilizar /auth y solucioné un error de sintaxis causado por una coma que faltaba.


3. Implementación del logout y revocación del refresh token

Prompts: "¿Cómo puedo hacer un logout? No me lo hagas, solo dame una pista".

Problema: No sabía cómo eliminar la cookie durante el logout ni por qué era necesario modificar la base de datos si la cookie ya se había borrado del navegador.

Solución: Aprendí que Express proporciona res.clearCookie(...) para solicitar al navegador que elimine una cookie. Para hacerlo correctamente, hay que utilizar el mismo nombre y las opciones relevantes con las que se creó, especialmente path.

También entendí que borrar la cookie no invalida por sí solo un refresh token que alguien haya copiado previamente. Por eso, el servicio verifica el token, identifica al usuario y establece refreshTokenHash a null. De esta forma, el refresh token anterior deja de coincidir con el valor almacenado y no puede utilizarse para renovar la sesión.


4. Adaptación de la ruta refresh al uso de cookies

Prompts: Como el refreshtoken ahora no se envía en el json se debe eliminar ValidateJoi(Schemas.auth.refresh), el esquema refresh de Joi.ts y las referencias refreshInput y RefreshInput de swagger.ts.

Problema: La ruta /auth/refresh seguía esperando un cuerpo de petición con el refresh token, aunque este ya se enviaba mediante una cookie.

Solución: Eliminé la validación del body que había dejado de ser necesaria y revisé las referencias asociadas al esquema anterior. También retiré los tipos y las definiciones de documentación que ya no correspondían al nuevo funcionamiento.

Con este cambio, el backend obtiene el refresh token de la cookie en lugar de esperar que el cliente lo envíe en el cuerpo de la petición.


5. Actualización y depuración de Swagger

Prompts: "En el Swagger no aparece la ruta logout" y consultas sobre el error Unexpected token '"', '"string"' is not valid JSON al probar refresh.

Problema: La ruta de logout existía en Express, pero no aparecía en la documentación de Swagger. Además, Swagger seguía enviando un body de ejemplo para refresh, aunque la ruta ya no lo necesitaba.

Solución: Aprendí que Swagger se genera a partir de las definiciones de documentación del proyecto y que crear una ruta en Express no implica que aparezca automáticamente en Swagger.

Añadí la definición OpenAPI de /auth/logout, sin requestBody y con una respuesta 204. También eliminé el requestBody obsoleto de refresh y comprobé los cambios reiniciando el servidor y recargando la documentación.


6. Configuración de Express y acceso a las cookies

Prompt: "¿Por qué me sale Cannot read properties of undefined (reading 'refreshToken')?".

Problema: La ruta /auth/refresh intentaba acceder a req.cookies.refreshToken, pero req.cookies era undefined. Esto provocaba un error interno del servidor en lugar de gestionar correctamente la ausencia de un token.

Solución: Identifiqué que faltaba el middleware encargado de interpretar las cookies. Instalé cookie-parser y sus tipos, lo importé en server.ts y configuré cookieParser() antes de las rutas que necesitan acceder a las cookies.

Así, Express puede interpretar la cabecera Cookie de las peticiones y proporcionar los valores correspondientes en req.cookies.


7. Depuración del refresh token y consultas a MongoDB

Prompts: "Cuando inicio sesión y hago refresh sale 401".

Problema: Después de iniciar sesión, la ruta refresh devolvía un error 401, aunque el hash del refresh token se había guardado correctamente en el usuario.

Solución: No caí en que puse refreshTokenHash como select:false basicamente como la contraseña para que no se envie simepre y solo se envie cunaod la necesite y claro como necesitaba el refreshTokenHash necesitaba hacer en la consulta findById un .select('+refreshTokenHash').

También entendí que el prefijo + permite incluir expresamente un campo que está excluido por defecto en las consultas. El mismo mecanismo se utilizaba ya con .select('+password') durante el login.

Por último, comprobé el comportamiento después del logout. La ausencia de un refresh token puede producir un 401, aunque es preferible comprobar previamente si existe para devolver un mensaje de error más claro.


8. Integración con Angular y comprobación de las cookies

Prompt: "En la aplicación no me sale la cookie cuando estoy iniciado".

Problema: Al iniciar sesión desde Angular, la cookie no aparecía en las herramientas de desarrollo del navegador, aunque el funcionamiento parecía correcto al probar el backend desde Swagger.

Solución: Aprendí a comprobar la pestaña Network para verificar si la respuesta del login incluía la cabecera Set-Cookie, en lugar de limitarme a mirar la tabla de cookies almacenadas.

También revisé la configuración de withCredentials en Angular y la configuración de CORS del backend, especialmente el origen permitido mediante CORS_ORIGIN y la opción credentials: true.

Esto me permitió entender que configurar correctamente la cookie en el backend no siempre es suficiente: también hay que tener en cuenta cómo se realizan las peticiones desde el frontend.


9. Depuración de eventos del socket y conceptos de RxJS

Prompts: "Han pasado días y ahora no muestra cuando admin crea un libro y no he tocado nada".

Problema: El evento book:created había dejado de llegar al otro navegador. Además, tenía dudas sobre el origen y la función de operadores como map, tap y finalize.

Solución: Descubrí que el backend se estaba ejecutando con npm start, que utilizaba la carpeta build. Por tanto, después de modificar el código fuente era necesario volver a ejecutar npm run build para actualizar los archivos compilados. Durante el desarrollo, npm run dev evitaba tener que realizar manualmente ese paso en cada cambio.



CONCLUSIÓN

Este ejercicio ha sido el que más me ha costado. Para hacerlo es en el que mas IA he utilizado principalmente para entenderlo todo bien.