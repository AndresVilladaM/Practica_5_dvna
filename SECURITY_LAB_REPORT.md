# Auditoría de seguridad con Snyk sobre una aplicación web vulnerable: detección, corrección y prevención

**[PENDIENTE: nombre completo del estudiante]**

Diplomado en Ciberseguridad Informática y Resiliencia Digital, Politécnico Colombiano Jaime Isaza Cadavid

Docente: Edwin Andrés Ochoa Agudelo

2 de noviembre de 2026

> Los datos que todavía no tengo confirmados aparecen como `[PENDIENTE: ...]`. Los resuelvo antes de entregar.
>
> Las capturas de pantalla y las respuestas a las preguntas de cierre van en el informe en PDF.

**Datos del trabajo**

| Campo | Valor |
|---|---|
| Repositorio auditado | DVNA (*Damn Vulnerable Node Application*), de Appsecco (s. f.). Commit base `9ba473a` |
| Rama de trabajo | `fix/v4-secret-mcp` |
| Tecnologías de la aplicación | Node.js, Express, Sequelize (MySQL) y plantillas EJS |
| Entorno | Windows 10 (build 19045), Node v20.17.0, npm 10.8.1, Snyk CLI 1.1308.3, Git y Cursor con el servidor MCP de Snyk |
| Autorización | Aplicación de laboratorio elegida por mí, como indicó el docente. Todo se analizó en mi computador; no toqué sistemas de terceros |

## Resumen ejecutivo

Para esta actividad hice una auditoría de seguridad a DVNA, una aplicación web que fue construida con fallas a propósito para practicar. Usé Snyk para encontrar los problemas, entender qué tan graves eran de verdad, corregirlos y comprobar con un nuevo escaneo que desaparecieran.

Corregí las tres vulnerabilidades que pedía la guía (Ochoa Agudelo, 2026) y una cuarta con ayuda del agente de Cursor:

- **V1, inyección SQL** (en el código propio): ya no se arma la consulta pegando texto del usuario.
- **V2, ejs 2.7.4 con ejecución remota de código** (en una dependencia): actualicé la librería, y al hacerlo tuve que arreglar 15 plantillas que dejaron de funcionar.
- **V3, inyección de comandos** (en el código propio): dejé de usar un shell para ejecutar `ping`.
- **Extra con MCP, secreto de sesión escrito en el código**: ahora se lee de una variable de entorno.

Al final dejé el proyecto bajo monitoreo continuo con `snyk monitor`, para que Snyk avise si aparece una vulnerabilidad nueva en las librerías que usa.

**Tabla 1**

*Resultados de los escaneos de Snyk, antes y después de las correcciones*

| Métrica | Inicial | Final |
|---|---|---|
| Dependencias (SCA): problemas totales | 136 (180 rutas) | 134 (179 rutas) |
| Dependencias (SCA): graves y críticos | 76 | 75 (14 críticos) |
| Código (SAST): problemas totales | 36 (4 graves, 27 medios, 5 bajos) | 34 (1 grave, 28 medios, 5 bajos) |

*Nota.* Los números salen de `snyk test` y `snyk code test`. Los graves y críticos de SCA los confirmé con `snyk test --severity-threshold=high`: 76 problemas en 91 rutas al inicio y 75 problemas en 90 rutas al final (`evidencias/scan_sca_final_high.txt`).

Lo más importante que aprendí es que **corregir un hallazgo casi no mueve el total**: de 136 a 134, de 36 a 34. El proyecto sigue lleno de problemas. Por eso no basta con contar vulnerabilidades; hay que decidir cuáles son un riesgo real para esta aplicación. Quedan como riesgo aceptado 14 vulnerabilidades críticas en dependencias (`libxmljs`, `mysql2`, `sequelize`, `mathjs`, `node-serialize`, `form-data`) y 1 grave en código (la deserialización con `node-serialize`, que no tiene parche).

## Conceptos clave

Para que el informe se pueda leer sin ser experto, dejo aquí los términos que uso.

**Tabla 2**

*Glosario de los términos usados en el informe*

| Término | En palabras simples |
|---|---|
| SCA | Revisa las librerías de terceros que usa el proyecto y avisa cuáles tienen fallas conocidas. |
| SAST | Lee el código que escribió el equipo, sin ejecutarlo, buscando patrones inseguros. |
| CVE | Código que identifica una falla conocida y pública (The CVE Program, s. f.). |
| CWE | Categoría del tipo de error, por ejemplo "inyección SQL" (MITRE, s. f.-b). |
| CVSS | Puntaje de 0 a 10 de la gravedad teórica de una falla. |
| EPSS | Probabilidad estimada de que una falla se use en ataques reales. |
| Madurez del exploit | Qué tan fácil es conseguir un ataque ya hecho. "Proof of Concept" significa que existe una demostración funcional. |
| Alcanzabilidad (*reachability*) | Si la aplicación realmente llega a usar el código vulnerable. Una falla puede existir y aun así no poder explotarse. |
| *Lockfile* (`package-lock.json`) | Archivo que fija las versiones exactas de todas las librerías, para que todos instalen lo mismo. |
| MCP | Protocolo que le permite al agente de IA de Cursor usar herramientas externas, en este caso Snyk. |
| *Quality Gate* | Puerta automática que bloquea un cambio si trae vulnerabilidades graves. |
| *Shift-left* | Revisar la seguridad desde que se escribe el código, no al final. |

## Metodología y alcance

Seguí el ciclo que propone la guía: detectar, analizar, priorizar, corregir, verificar y prevenir.

1. **Preparación.** Instalé el CLI de Snyk, me autentiqué con `snyk auth` y comprobé la sesión con `snyk whoami`.
2. **Análisis de dependencias (SCA).** Ejecuté `snyk test` y `snyk test --severity-threshold=high` (este último muestra solo graves y críticos).
3. **Análisis del código (SAST).** Ejecuté `snyk code test`.
4. **Selección.** Elegí V1 (SAST), V2 (SCA, con CVE y versión segura clara) y V3 (SAST). Prioricé hallazgos graves, con una ruta alcanzable en el código y que pudiera corregir y verificar sin cambiar de librería.
5. **Corrección y reescaneo.** Cada corrección va en su propio commit, con un escaneo posterior.
6. **Cursor y MCP.** Usé el agente con las herramientas de Snyk para una auditoría y una cuarta corrección.
7. **Monitoreo continuo.** Al final ejecuté `snyk monitor` para dejar el proyecto vigilado en la consola de Snyk.
8. **Prevención.** Armé reglas y controles para que estos errores no vuelvan.

**Reglas de seguridad que respeté.** Trabajé solo con la aplicación de laboratorio. No puse el token de Snyk en ningún archivo ni captura: la configuración de MCP no lleva token porque usa la sesión del CLI. Revisé cada cambio que propuso el agente antes de aceptarlo.

**Limitaciones del entorno.**

- `npm install` normal falló en Windows porque `libxmljs` y `bcrypt` necesitan compilarse con Visual Studio C++. Instalé con `npm install --ignore-scripts --legacy-peer-deps`, que descarga todo sin compilar y le basta a Snyk para leer el árbol de dependencias.
- Por eso **la aplicación no arranca en mi computador** y no pude probar las correcciones en ejecución. Las comprobé con `node --check` (valida la sintaxis), con la compilación de las plantillas EJS y con los reescaneos de Snyk.
- `snyk monitor` solo analiza dependencias (SCA). El total que muestra la consola (134 problemas) coincide con el de `snyk test`, así que los hallazgos del código (SAST) no aparecen allí, solo en la salida de `snyk code test`.

## Matriz de vulnerabilidades

**Tabla 3**

*Las tres vulnerabilidades seleccionadas y la corrección adicional*

| # | Hallazgo | SAST/SCA | CVE / CWE | Severidad | CVSS | Riesgo real (mi valoración) | Estado |
|---|---|---|---|---|---|---|---|
| 1 | Inyección SQL en `userSearch` | SAST | CVE N/A / CWE-89 | Alta | N/A (Snyk Code no lo publica) | Alto: la ruta está al alcance de cualquier usuario con sesión y da acceso a datos de usuarios | Corregido |
| 2 | Ejecución remota de código en `ejs@2.7.4` | SCA | CVE-2022-29078 / CWE-94 | Alta | 8.1 (AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H) | Condicional: exige una condición previa (*Prototype Pollution*) y no demostré una ruta directa | Corregido |
| 3 | Inyección de comandos en `ping` | SAST | CVE N/A / CWE-78 | Alta | N/A | Alto: permite ejecutar comandos en el servidor con una sesión de bajo privilegio | Corregido |
| Extra | Secreto de sesión escrito en el código | SAST | CVE N/A / CWE-547 | Alta | N/A | Medio-alto: quien tenga el código puede firmar cookies de sesión | Corregido |

*Nota.* El CLI de Snyk Code no imprime el CWE. Los CWE-89 y CWE-78 corresponden a las categorías estándar de MITRE para inyección SQL y para inyección de comandos del sistema operativo (MITRE, s. f.-a, s. f.-b). El CWE-547 salió del reporte del agente MCP. Los OWASP Top 10 de V1 y V3 son A03:2021, inyección (OWASP Foundation, 2021).

---

## Vulnerabilidad 1: inyección SQL (SAST)

**En palabras simples.** La búsqueda de usuarios armaba la consulta a la base de datos pegando directamente lo que escribía el usuario. Si alguien escribe algo con la forma de una instrucción SQL, la base de datos la ejecuta como si fuera parte de la consulta.

#### Ficha técnica y puntuación

**Tabla 4**

*Ficha técnica de la vulnerabilidad 1*

| Campo | Valor |
|---|---|
| Hallazgo | SQL Injection |
| Tipo | SAST (Snyk Code) |
| CWE / CVE | CWE-89 / N/A |
| Severidad | Alta |
| CVSS | N/A en SAST |
| Ubicación | `core/appHandler.js`, línea 11, función `userSearch` |
| Finding ID | `7da5a0ab-ccc2-4314-855b-d379abd863e5` |
| OWASP | A03:2021, inyección |

#### Interpretación del riesgo

- **Vector y privilegios.** La ruta es `POST /usersearch` y exige sesión iniciada (`authHandler.isAuthenticated`, en `routes/app.js:50`). No es de acceso anónimo, pero cualquiera con una cuenta puede usarla, y la aplicación tiene formulario de registro (`views/register.ejs`).
- **Qué queda expuesto.** La tabla de usuarios.
- **Impacto en confidencialidad, integridad y disponibilidad (CID).** Confidencialidad alta: se podrían leer datos de los usuarios. Integridad baja o media: depende de si el controlador de base de datos permite varias consultas seguidas, y `mysql2` no lo permite por defecto. Disponibilidad baja.
- **Impacto para el negocio.** Filtración de datos personales y de las credenciales guardadas.
- **Madurez del exploit y alcanzabilidad.** La madurez no aplica a SAST. La ruta sí es alcanzable por un usuario autenticado.

#### Causa raíz

Es un error de implementación (CWE-89): el texto del usuario se concatena dentro de la consulta sin validarlo ni separarlo. La base de datos no puede distinguir entre el dato y la instrucción. Snyk lo describe como entrada del cuerpo de la petición que llega directo a `query`.

#### Escenario de explotación

Un usuario con sesión escribe en el campo `login` un valor que cierra la comilla y agrega una condición que siempre es verdadera, y la consulta devuelve registros que no le corresponden. Con una consulta `UNION` podría leer otras columnas o tablas. No ejecuté la explotación: la aplicación no corre en mi entorno, así que esta descripción es teórica.

#### Código vulnerable (ANTES)

```js
module.exports.userSearch = function (req, res) {
	var query = "SELECT name,id FROM Users WHERE login='" + req.body.login + "'";
	db.sequelize.query(query, {
		model: db.User
	}).then(user => {
```

#### Remediación aplicada

Quité la concatenación y pasé la búsqueda por el ORM (Sequelize). Además agregué una validación de tipo y largo como segunda barrera.

#### Código corregido (DESPUÉS)

```js
module.exports.userSearch = function (req, res) {
        var login = req.body.login
        if (typeof login !== 'string' || login.length === 0 || login.length > 50) {
                req.flash('warning', 'Invalid login')
                return res.render('app/usersearch', {
                        output: null
                })
        }
        db.User.findAll({
                attributes: ['name', 'id'],
                where: { login: login }
        }).then(user => {
```

**Por qué corrige la causa raíz.** El valor ya no se pega a mano en el texto SQL: pasa por el mecanismo de escape del ORM, que lo trata como dato y no como instrucción. El resto de la función (`user[0].name`, `user[0].id`) sigue igual, así que el comportamiento esperado se mantiene. Comprobé la sintaxis con `node --check core\appHandler.js`, sin errores. No hubo prueba en ejecución.

#### Evidencia del reescaneo

Después del cambio, `snyk code test` ya no muestra "SQL Injection" (`Select-String` sin coincidencias en `evidencias/scan_sast_v1_despues.txt`). Los hallazgos graves bajaron de 4 a 3. El total se quedó en 36 porque apareció un hallazgo medio nuevo de "falta de límite de peticiones" sobre la consulta nueva; es ruido de la herramienta, no una regresión de seguridad.

**Riesgo que queda.** La versión `sequelize@4.44.4` todavía tiene fallas de inyección SQL propias, una de ellas crítica (`SNYK-JS-SEQUELIZE-2932027`). Corregir mi código era necesario, pero no reemplaza actualizar la librería. Es un buen ejemplo de por qué SAST y SCA se complementan.

#### Prevención y *shift-left*

Una regla en `.cursorrules` que prohíbe concatenar entradas en SQL, validación de todo lo que llega del usuario, y `snyk code test` en el *pre-commit* y en el *Quality Gate* (ver la sección de controles preventivos).

---

## Vulnerabilidad 2: ejs 2.7.4, ejecución remota de código (SCA)

**En palabras simples.** Esta no es una falla de mi código sino de una librería: la versión de `ejs` (el motor que dibuja las páginas) tiene un fallo que, bajo ciertas condiciones, permite ejecutar código en el servidor. La solución es actualizar la librería, pero eso rompió otras cosas.

#### Ficha técnica y puntuación

**Tabla 5**

*Ficha técnica de la vulnerabilidad 2*

| Campo | Valor |
|---|---|
| Hallazgo | Remote Code Execution (RCE) en ejs |
| Tipo | SCA (Snyk Open Source) |
| Snyk ID | `SNYK-JS-EJS-2803307` |
| CVE / CWE | CVE-2022-29078 (The CVE Program, s. f.) / CWE-94 (MITRE, s. f.-c) |
| Severidad | Alta |
| CVSS v3.1 | 8.1: `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H` |
| Dependencia | `ejs@2.7.4`, directa (`package.json`: `"ejs": "^2.5.7"`) |
| Versión segura | 3.1.7 o superior (apliqué 3.1.10) |
| Publicada | 26 de abril de 2022 |

*Nota.* Datos tomados de la advisory de Snyk (s. f.-b).

#### Interpretación del riesgo

- **Cómo se lee el CVSS.** `AV:N` se ataca por red; `AC:H` es difícil de lograr (exige una condición previa); `PR:N` y `UI:N` no requiere cuenta ni que alguien haga algo; `C:H/I:H/A:H` el impacto es alto en confidencialidad, integridad y disponibilidad.
- **Madurez y probabilidad.** La madurez del exploit es *Proof of Concept* y el EPSS es 32,81 % (percentil 99). Dicho simple: existe una demostración funcional y su probabilidad de uso es alta comparada con otras fallas.
- **La condición clave.** La advisory explica que el fallo está en cómo `renderFile` acepta una opción llamada `outputFunctionName`, y aclara que **solo es explotable si el servidor ya es vulnerable a *Prototype Pollution*** (Snyk, s. f.-b). *Prototype Pollution* es un ataque que modifica propiedades que comparten todos los objetos de JavaScript.
- **Alcanzabilidad en esta aplicación.** Busqué en el código los `res.render(...)` que reciben datos del usuario y solo encontré uno, `routes/app.js:23` (`res.render('app/bulkproducts', {legacy: req.query.legacy})`). Ahí solo se pasa el valor de `legacy`, no un objeto de opciones, así que **no pude demostrar una ruta directa**. Sin embargo, el proyecto tiene fallas de *Prototype Pollution* en otras librerías (`express-fileupload@0.4.0`, `SNYK-JS-EXPRESSFILEUPLOAD-595969`, y `mathjs@3.10.1`, `SNYK-JS-MATHJS-1016401`). No verifiqué si se pueden encadenar. Mi búsqueda además solo detecta `req.` en la misma línea del `render`; es un límite del análisis.
- **Mi valoración.** El riesgo es condicional: el 8.1 sobrestima la exposición directa, pero el EPSS alto y la madurez de PoC justifican corregir de todas formas, porque la actualización es barata.
- **Impacto CID.** Alto en las tres, porque se ejecutaría código en el servidor.

#### Causa raíz

Una dependencia desactualizada que no valida bien las opciones de renderizado (CWE-94, inyección de código). El defecto está en la librería, no en el código del proyecto.

#### Escenario de explotación

Un atacante que lograra contaminar los objetos de la aplicación, o controlar las opciones que llegan al renderizado, podría meter código en `outputFunctionName` y se ejecutaría en el servidor al dibujar una vista, con los permisos del proceso Node. No reproduzco ninguna carga útil ni ejecuté la explotación.

#### Dependencia vulnerable (ANTES)

```json
"ejs": "^2.5.7"   // resuelto: ejs@2.7.4
```

#### Remediación aplicada

Actualicé a `ejs@3.1.10`, que además corrige el hallazgo medio de ejs (la 3.1.7 solo corrige el RCE):

```
npm install ejs@3.1.10 --ignore-scripts --legacy-peer-deps
```

**El problema que apareció.** ejs 3 eliminó la forma antigua de incluir archivos, `<% include archivo %>`, y 15 plantillas la usaban. Lo comprobé compilando todas las plantillas:

- Antes de migrar: 15 plantillas fallaban (`evidencias/compat_ejs_antes.txt`).
- Migré 45 líneas (3 por archivo) a la sintaxis nueva, por ejemplo `<% include common/head %>` pasó a `<%- include('common/head') %>`.
- Después: 54 plantillas compilan y ninguna falla (`evidencias/compat_ejs_despues.txt`).

Esto fue una lección importante: **actualizar una librería no es solo cambiar el número de versión**. Sin esta comprobación habría dejado casi toda la aplicación sin poder dibujar páginas. Como limitación, compilar valida la sintaxis pero no que cada `include` encuentre su archivo al ejecutarse; las tres llamadas dinámicas de `views/vulnerabilities/layout.ejs` son lo más delicado y no las probé en ejecución.

#### Dependencia corregida (DESPUÉS)

```json
"ejs": "^3.1.10"
```

#### Evidencia del reescaneo

`snyk test` ya no muestra `SNYK-JS-EJS-2803307` ni el hallazgo medio de ejs. El total pasó de 136 a 134 problemas, y de 180 a 179 rutas. Analizó 315 dependencias en lugar de 310, porque ejs 3 trajo 5 paquetes nuevos.

El reporte todavía menciona `Upgrade ejs@3.1.10 to ejs@5.0.1`, pero el hallazgo que aparece ahí es un `minimatch` medio que ya existía en el escaneo inicial. Con `npm ls minimatch` comprobé que ejs 3.1.10 sumó una ruta nueva (`ejs > jake@10.9.4 > filelist@1.0.6 > minimatch@5.1.9`), coherente con el paso de 7 a 8 rutas adicionales de ese mismo hallazgo; las demás vienen de `bcrypt > node-pre-gyp`. `jake` es una dependencia de las tareas de ejs y no verifiqué si interviene al renderizar. Lo acepto como riesgo residual medio.

#### Prevención y *shift-left*

Versioné `package-lock.json` (antes estaba en `.gitignore`), instalo con `npm ci` y el *Quality Gate* bloquea las vulnerabilidades críticas con `snyk test`. Política propuesta: revisar semanalmente las dependencias y actualizar primero las que tengan exploit público.

---

## Vulnerabilidad 3: inyección de comandos (SAST)

**En palabras simples.** La función que hace `ping` armaba un comando del sistema pegándole la dirección que escribe el usuario, y lo ejecutaba en un shell. Si el usuario agrega símbolos como `;` o `&&`, puede encadenar otros comandos.

#### Ficha técnica y puntuación

**Tabla 6**

*Ficha técnica de la vulnerabilidad 3*

| Campo | Valor |
|---|---|
| Hallazgo | Command Injection |
| Tipo | SAST (Snyk Code) |
| CWE / CVE | CWE-78 / N/A |
| Severidad | Alta |
| CVSS | N/A en SAST |
| Ubicación | `core/appHandler.js`, línea 39 en el escaneo inicial (46 tras la V1), función `ping` |
| Finding ID | `ae442cac-a810-48ba-be7a-50c7c3842477` |
| OWASP | A03:2021, inyección |

#### Interpretación del riesgo

- **Vector y privilegios.** La ruta es `POST /ping` y exige sesión (`authHandler.isAuthenticated`, `routes/app.js:52`), igual que la V1.
- **Condiciones.** Una sesión activa y la entrada `address` sin validar, ejecutada dentro de un shell.
- **Qué queda expuesto.** El servidor y el proceso Node.
- **Impacto CID.** Alto en las tres: leer y escribir archivos, ejecutar comandos y dejar el servicio caído.
- **Impacto para el negocio.** Compromiso del servidor, acceso a secretos y salto hacia otros sistemas.
- **Alcanzabilidad.** La ruta está al alcance de cualquier usuario autenticado.

#### Causa raíz

Error de implementación (CWE-78). `exec` abre un shell y la dirección del usuario se pega en la línea de comandos, así que símbolos como `;`, `&&` o `|` permiten ejecutar un segundo comando.

#### Escenario de explotación

Un usuario con sesión envía como dirección un valor que agrega otro comando después del `ping`, y el servidor lo ejecuta con los permisos del proceso. No ejecuté la explotación.

#### Código vulnerable (ANTES)

```js
module.exports.ping = function (req, res) {
	exec('ping -c 2 ' + req.body.address, function (err, stdout, stderr) {
		output = stdout + stderr
		res.render('app/ping', {
			output: output
		})
	})
}
```

#### Remediación aplicada

Dejé de usar un shell (`execFile` con los argumentos por separado) y validé la dirección con una lista de caracteres permitidos (una IP o un nombre de host válido).

#### Código corregido (DESPUÉS)

```js
const execFile = require('child_process').execFile;
const net = require('net');

module.exports.ping = function (req, res) {
        var address = req.body.address
        var hostRegex = /^[a-zA-Z0-9]([a-zA-Z0-9.-]{0,251}[a-zA-Z0-9])?$/
        if (typeof address !== 'string' || !(net.isIP(address) || hostRegex.test(address))) {
                req.flash('warning', 'Invalid address')
                return res.render('app/ping', {
                        output: null
                })
        }
        execFile('ping', ['-c', '2', address], function (err, stdout, stderr) {
                var output = stdout + stderr
                res.render('app/ping', {
                        output: output
                })
        })
}
```

**Por qué corrige la causa raíz.** `execFile` no usa un shell: el valor llega como un único argumento y los símbolos especiales dejan de tener significado. La lista permitida además impide que el valor empiece por `-` y `ping` lo tome como una opción. Aproveché para agregar `var` a `output`, que antes era una variable global sin declarar. Comprobé la sintaxis con `node --check`. `ping -c` es sintaxis de Linux y mi aplicación no corre en Windows, así que no hubo prueba en ejecución.

#### Evidencia del reescaneo

El primer reescaneo seguía mostrando el hallazgo, pero la línea `Path` apuntaba a `evidencia_v3_antes_appHandler.js`: **la copia de respaldo del código vulnerable estaba dentro del proyecto y Snyk la escaneó**. Moví las evidencias fuera del repositorio y el nuevo reescaneo ya no muestra "Command Injection" (`evidencias/scan_sast_v3_despues.txt`). Los problemas totales pasaron de 36 a 35 y los graves, de 3 a 2. La lección: las copias de código vulnerable no deben quedar donde las pueda escanear la herramienta. Por eso en el repositorio las guardé con extensión `.txt`.

#### Prevención y *shift-left*

Regla en `.cursorrules` contra `exec` con entradas del usuario, regla ESLint `security/detect-child-process` (propuesta) y `snyk code test` en el *pre-commit* y en el *Quality Gate*.

---

## Comparación antes y después

**Tabla 7**

*Evolución de los hallazgos de código (`snyk code test`)*

| Etapa | Total | Graves | Medios | Bajos |
|---|---|---|---|---|
| Inicial | 36 | 4 | 27 | 5 |
| Tras V1 | 36 | 3 | 28 | 5 |
| Tras V3 | 35 | 2 | 28 | 5 |
| Tras corregir el secreto (MCP) | 34 | 1 | 28 | 5 |

**Tabla 8**

*Evolución de los hallazgos de dependencias (`snyk test`)*

| Etapa | Problemas | Rutas vulnerables | Dependencias analizadas |
|---|---|---|---|
| Inicial | 136 | 180 | 310 |
| Tras V2 (final) | 134 | 179 | 315 |

*Nota.* Graves y críticos de SCA: 76 problemas (91 rutas) con el RCE de ejs y 75 (90 rutas) sin él, de los cuales 14 son críticos. Un mismo problema puede aparecer por varias rutas de dependencias, por eso hay más rutas que problemas. La consola de Snyk (`snyk monitor`) muestra el mismo total de 134 problemas: 14 críticos, 61 graves, 56 medios y 3 bajos.

## Evidencias de reescaneo

**Tabla 9**

*Comprobación de cada corrección con un nuevo escaneo*

| Hallazgo | Comando | Resultado | Archivo |
|---|---|---|---|
| V1, inyección SQL | `snyk code test` | Ya no aparece "SQL Injection"; 36 problemas, 3 graves | `evidencias/scan_sast_v1_despues.txt` |
| V3, inyección de comandos | `snyk code test` | Ya no aparece "Command Injection"; 35 problemas, 2 graves | `evidencias/scan_sast_v3_despues.txt` |
| V2, ejs | `snyk test` | Ya no aparece `SNYK-JS-EJS-2803307`; 134 problemas | `evidencias/scan_sca_v2_despues.txt` |
| Secreto (MCP) | `snyk code test` y `snyk_code_scan` | Ya no aparece `HardcodedNonCryptoSecret`; 34 problemas | `evidencias/scan_sast_v4_despues.txt` |
| Estado final | `snyk test` y `snyk code test` | SCA 134, SAST 34 (1 grave) | `evidencias/scan_sca_final.txt`, `evidencias/scan_sast_final.txt` |

## Integración con Cursor y MCP

**Configuración.** El archivo `.cursor/mcp.json` arranca el servidor MCP que ya viene dentro del CLI de Snyk (`snyk mcp -t stdio`), sin token en el archivo (Snyk, s. f.-a). El material de clase mencionaba un paquete `@snyk/mcp-server`; no lo encontré en la documentación de Snyk, cuyo método oficial es el CLI, así que usé ese. El servidor `project-0-dvna-snyk` quedó en estado *ready* con 15 herramientas: `mcp_auth`, `snyk_aibom`, `snyk_auth`, `snyk_breakability_check`, `snyk_code_scan`, `snyk_container_scan`, `snyk_iac_scan`, `snyk_logout`, `snyk_package_health_check`, `snyk_sbom_scan`, `snyk_sca_scan`, `snyk_secret_scan`, `snyk_send_feedback`, `snyk_trust` y `snyk_version`.

**Tabla 10**

*Uso de las herramientas MCP y revisión humana*

| Paso | Herramienta MCP | Hallazgo | Qué propuso el agente | Mi revisión |
|---|---|---|---|---|
| Auditoría | `snyk_trust`, `snyk_sca_scan` (`severity_threshold=high`, `all_projects=true`), `snyk_code_scan` | 77 graves y críticos (75 SCA, 2 SAST) | Un informe `SECURITY_AUDIT_REPORT.md` | Comparé los conteos con mi escaneo local: los 14 críticos y los 61 graves coinciden, igual que los 2 de SAST |
| Secretos | `snyk_secret_scan` | No aplica | Falló con error 403 (`SNYK-CLI-0016`): el módulo de secretos no está habilitado en mi organización | Lo documenté como limitación; el secreto sí lo detectó Snyk Code |
| Corrección | `snyk_code_scan` | `HardcodedNonCryptoSecret` (CWE-547), `server.js` línea 23 (el texto está en la 24) | Leer `SESSION_SECRET` de `process.env`, detener el arranque si falta, y agregar `.gitignore` y `.env.example` | Vi que la línea `.env` del `.gitignore` tenía un espacio al inicio, lo que la habría dejado sin efecto. También cuestioné que atribuyera el desfase de línea a "indexación" sin prueba. Revisé el diff antes de aplicar |
| Reescaneo | `snyk_code_scan` | Sin el secreto | 34 problemas | Lo confirmé con `snyk code test` por CLI |

**Lo que falló antes de que funcionara** (limitaciones reales de la automatización):

1. El agente corrió en una copia aislada del proyecto (un *worktree*) que no tenía el archivo `.cursor/mcp.json`, porque yo no lo había versionado. No vio el servidor y usó comandos de terminal en lugar de las herramientas MCP. Su reporte dio 90 hallazgos en SCA frente a los 75 problemas únicos de Snyk. El 90 coincide con el número de rutas vulnerables (75 problemas en 90 rutas), así que lo más probable es que contara la misma falla varias veces, una por cada ruta de dependencias. Además reinstaló las dependencias ignorando el *lockfile*, que en ese momento ni siquiera estaba versionado.
2. Cuando le pedí que listara las herramientas disponibles, **inventó una lista que sonaba creíble** (`list_projects`, `fix_project`, etc.) y que no correspondía al servidor. La lista válida es la que muestra el panel de MCP de Cursor.
3. Lo resolví versionando `.cursor/mcp.json`, abriendo un chat nuevo y exigiendo en el *prompt* el uso exclusivo de herramientas MCP.

## Controles preventivos y *Quality Gate*

**Tabla 11**

*Estado de los controles de prevención*

| Control | Estado | Qué hace |
|---|---|---|
| `package-lock.json` versionado | Implementado y verificado (`5f6b333`) | Antes estaba en `.gitignore`, así que el CI y el agente no escaneaban lo mismo que yo |
| `.env` ignorado y `.env.example` | Implementado y verificado (`3e5a0a7`) | El secreto de sesión ya no está en el código |
| `snyk monitor` | Implementado y verificado | Deja el proyecto vigilado en la consola de Snyk, que muestra 134 problemas (14 críticos, 61 graves, 56 medios y 3 bajos), igual que `snyk test` |
| `.cursorrules` | Creado (`c0cc07f`); no medí su efecto en el agente | Le dice al agente qué prácticas seguras usar |
| Workflow `snyk-security-gate.yml` | Creado (`c0cc07f`); no ejecutado en GitHub | Bloquea Pull Requests con vulnerabilidades graves |
| ESLint con `eslint-plugin-security` | Propuesto | Detecta en el editor patrones como `exec` con entradas del usuario |
| *Hook pre-commit* | Propuesto | Corre ESLint y `snyk code test` antes de cada commit |
| Escaneo de secretos | Propuesto | `snyk_secret_scan` falló por el 403; alternativas: gitleaks o el escaneo de secretos de GitHub |
| Regla de protección de rama | Propuesto | Exige que pase el chequeo de Snyk para fusionar |
| Política de dependencias | Propuesto | `npm ci`, revisión semanal y prioridad a las que tienen exploit público |

**El monitoreo continuo** (`snyk monitor`) toma una copia del árbol de dependencias (`package.json` y `package-lock.json`) y la guarda en mi organización de Snyk, que avisa si se publica una vulnerabilidad nueva que afecte esas librerías. El comando solo devolvió el enlace al proyecto. En la consola, los 134 problemas coinciden con `snyk test`. Solo cubre dependencias, no el código propio.

**Las reglas de `.cursorrules`** salen de lo que me pasó en este laboratorio: no concatenar entradas en SQL, no usar `exec` con entradas del usuario, validar toda entrada, leer los secretos de `process.env`, no usar `eval` ni `node-serialize`, verificar que una dependencia nueva exista y no tenga hallazgos graves (para evitar paquetes inventados), usar la sintaxis de EJS 3, escanear con MCP después de cada cambio y esperar mi aprobación para cada diff. Las versiones recientes de Cursor prefieren `.cursor/rules/`, pero la guía pide `.cursorrules`.

**El *Quality Gate*** (`.github/workflows/snyk-security-gate.yml`) se ejecuta en cada Pull Request hacia `master`: instala con `npm ci --ignore-scripts --legacy-peer-deps`, ejecuta `snyk test --severity-threshold=critical` y `snyk code test --severity-threshold=high`, con `SNYK_TOKEN` guardado como secreto del repositorio. Si un comando devuelve un código de salida distinto de cero, el merge se bloquea. **Resultado esperado, no observado:** debería fallar con este código, por las 14 críticas de SCA y el grave de SAST que quedan. `[ACTUALIZAR si se ejecuta el Pull Request: resultado y captura]`.

## Conclusiones

1. Completé el ciclo detectar, analizar, priorizar, corregir, verificar y prevenir sobre tres vulnerabilidades y una adicional, cada una con su commit y su reescaneo.
2. Las correcciones eliminaron sus hallazgos, pero los totales casi no cambiaron (136 a 134 en SCA, 36 a 34 en SAST). Contar vulnerabilidades no mide el riesgo.
3. Actualizar una librería puede romper el proyecto: ejs 3 dejó 15 plantillas sin funcionar y solo lo supe porque las compilé.
4. Corregir el código no reemplaza actualizar la librería: arreglé la inyección SQL de mi código, pero `sequelize@4.44.4` sigue con fallas del mismo tipo.
5. La automatización necesita supervisión humana: el agente trabajó sin las herramientas, inventó una lista de herramientas, reinstaló sin el *lockfile* y propuso un `.gitignore` con un error.
6. Queda riesgo aceptado considerable: 14 críticas en dependencias y la deserialización con `node-serialize`. Lo correcto sería eliminar o reemplazar esas librerías.
7. `snyk monitor` deja el análisis de dependencias vivo: ya no dependo de volver a ejecutar el escaneo para enterarme de una falla nueva. Cubre solo dependencias, no el código propio.

## Referencias

Appsecco. (s. f.). *DVNA: Damn Vulnerable Node Application* [Repositorio]. GitHub. https://github.com/appsecco/dvna

MITRE. (s. f.-a). *CWE-78: Improper neutralization of special elements used in an OS command ('OS command injection')*. https://cwe.mitre.org/data/definitions/78.html

MITRE. (s. f.-b). *CWE-89: Improper neutralization of special elements used in an SQL command ('SQL injection')*. https://cwe.mitre.org/data/definitions/89.html

MITRE. (s. f.-c). *CWE-94: Improper control of generation of code ('code injection')*. https://cwe.mitre.org/data/definitions/94.html

Ochoa Agudelo, E. A. (2026). *Implementación práctica: Snyk, CLI y MCP en Cursor* [Guía de actividad]. Diplomado en Ciberseguridad Informática y Resiliencia Digital.

OWASP Foundation. (2021). *OWASP Top 10:2021*. https://owasp.org/Top10/

Snyk. (s. f.-a). *Cursor guide*. Snyk User Docs. https://docs.snyk.io/cli-ide-and-ci-cd-integrations/snyk-cli/developer-guardrails-for-agentic-workflows/quickstart-guides-for-mcp/cursor-guide

Snyk. (s. f.-b). *Remote code execution (RCE) in ejs* (SNYK-JS-EJS-2803307) [Base de datos de vulnerabilidades]. https://security.snyk.io/vuln/SNYK-JS-EJS-2803307

The CVE Program. (s. f.). *CVE-2022-29078*. https://www.cve.org/CVERecord?id=CVE-2022-29078

`[PENDIENTE: ajustar al formato APA de la institución (sangría francesa, cursivas y fechas de consulta si las exigen)]`

## Anexo. Archivos de evidencia

Todos están en la carpeta `evidencias/` del repositorio.

**Tabla A1**

*Archivos de evidencia y lo que demuestra cada uno*

| Archivo | Qué demuestra |
|---|---|
| `evidencias/scan_sca_inicial.txt` | `snyk test` inicial: 136 problemas, 180 rutas |
| `evidencias/scan_sast_inicial.txt` | `snyk code test` inicial: 36 problemas, 4 graves |
| `evidencias/scan_sast_v1_despues.txt` | Reescaneo tras V1 |
| `evidencias/scan_sast_v3_despues.txt` | Reescaneo tras V3 |
| `evidencias/scan_sca_v2_despues.txt` | Reescaneo tras V2 |
| `evidencias/scan_sast_v4_despues.txt`, `evidencias/scan_sast_final.txt` | Reescaneo tras corregir el secreto y estado final de SAST |
| `evidencias/scan_sca_final.txt`, `evidencias/scan_sca_final_high.txt` | Estado final de SCA (total y solo graves y críticos) |
| `evidencias/diff_v1_commit.txt`, `diff_v3_commit.txt`, `diff_v2_commit.txt`, `diff_v4_commit.txt` | Cambio de cada corrección (`git show` de los commits `81a132d`, `418bbd4`, `dc9973d` y `3e5a0a7`) |
| `evidencias/diff_v1.txt`, `diff_v2.txt`, `diff_v3.txt`, `diff_v4.txt` | Diffs tomados antes de cada commit |
| `evidencias/evidencia_v2_antes_package.json.txt`, `evidencias/evidencia_v3_antes_appHandler.js.txt` | Estado ANTES de V2 y V3 (con extensión `.txt` para que Snyk no los escanee) |
| `evidencias/compat_ejs_antes.txt`, `evidencias/compat_ejs_despues.txt` | Compilación de plantillas: 15 fallan, luego 54 compilan sin fallos |
| `evidencias/SECURITY_AUDIT_REPORT_mcp.md` | Reporte que generó el agente con las herramientas MCP |

**Commits:** `81a132d` (V1), `418bbd4` (V3), `dc9973d` (V2), `dec0cc4` y `c4cf36c` (configuración MCP), `5f6b333` (lockfile versionado), `3e5a0a7` (secreto por variable de entorno), `c0cc07f` (CI y reglas de Cursor).
