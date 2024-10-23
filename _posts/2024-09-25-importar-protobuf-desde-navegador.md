---
title: "Cómo Importar Mensajes Protobuf desde el Navegador"
date: 2024-09-25T10:00:00-05:00
categories:
  - Desarrollo-full-stack
tags:
  - typescript
  - IDX
  - protobuf
keywords:
  - Protocol Buffers
  - Project IDX
  - node
  - expressjs
  - express
description: Aprende a importar y usar mensajes protobuf en el navegador con ES Modules. Descubre cómo integrar Protocol Buffers en el frontend de tu app.
header:
  teaser: /assets/images/post4-result.jpg
serie: protobuf-node
serieitemtitle: Día 4 - Cómo Importar Mensajes Protobuf desde el Navegador
---

Después de un tiempo, regreso con un nuevo post de la serie "Prueba Protocol Buffers con Node desde IDX". En esta ocasión, damos inicio al desarrollo del frontend de nuestra aplicación, comenzando por importar y utilizar un mensaje proto.

## Decisiones para la Aplicación Frontend

Para la construcción de la aplicación, he decidido adoptar la [programación funcional](https://es.wikipedia.org/wiki/Programaci%C3%B3n_funcional) como paradigma principal. Además, utilizaré [ES Modules](https://developer.mozilla.org/es/docs/Web/JavaScript/Guide/Modules) para la gestión de módulos en el navegador. Esto permitirá un desarrollo más modular y organizado, aprovechando las funcionalidades nativas de JavaScript moderno.

## Manejo de Errores en la Decodificación

```diff
# proto.middleware.ts

const proto: RequestHandler = (req: InProtoRequest, res: ProtoResponse, next) => {
-  const helloRequest = HelloRequest.decode(req.body as Buffer);
-  req.body = helloRequest;
+  try {
+      const helloRequest = HelloRequest.decode(req.body as Buffer);
+      req.body = helloRequest;
+  } catch (error) {
+      if (error instanceof $protobuf.util.ProtocolError) {
+          return res.status(400)
+              .send('API_ERROR(decode): ' + error.message);
+      }
+      return res.status(500)
+      .send('API_ERROR(decode): ' + error.message);
+    }
```

Antes de comenzar con el frontend, me di cuenta de que había dejado pendiente el manejo de errores al decodificar los mensajes protobuf en la API. Para asegurarnos de que los datos pasen correctamente del middleware al controlador sin errores, implementé una validación adicional en el proceso de decodificación.

## Script `proto:es`

```diff
# package.json

    "build": "npm run proto:build && tsc  && cp proto-bundle.js dist/proto-bundle.js",
    "start": "node dist/index.js",
    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js $(node scripts/get-proto-files.js)",
+    "proto:es": "pbjs -t json-module -w es6 --es6 -o public/proto-es-bundle.js $(node scripts/get-proto-files.js)",
    "proto:types": "pbjs -t static-module $(node scripts/get-proto-files.js) | pbts -o proto-bundle.d.ts -",
-    "proto:build": "npm run proto:to-json && npm run proto:types"
+    "proto:build": "npm run proto:to-json && npm run proto:types && npm run proto:es"
  },
  "keywords": [],
  "author": "",
```

Dado que vamos a utilizar ES Modules, agregué un nuevo script que compila los mensajes protobuf para un entorno compatible con módulos. El CLI de protobufjs permite especificar parámetros para generar código compatible con ES Modules, lo que fue esencial para usar los mensajes protobuf creados en el [día 1](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/prueba-protobuf-node-dia-1/).

## Probando los Módulos

### HTML y Punto de Entrada

```diff
# views/index.ejs

    <link rel="stylesheet" type="text/css" href="/styles.css" />
</head>
<body>
-    <h1>Hello world</h1>
+    <main>
+        <div id="app"></div>
+    </main>
+    <script type="module" src="/index.mjs"></script>
</body>
</html>
```

Lo primero fue crear el punto de entrada de la aplicación en el HTML, donde se declarará el módulo principal. Además, añadí un espacio en el DOM donde se podrá renderizar contenido dinámico, preparando el escenario para futuras interacciones.

### Funciones Impuras

```diff
# public/impure.mjs

+import curry from "https://unpkg.com/ramda@0.30.1/es/curry.js"
+export const getDOMEl = (sel) => document.querySelector(sel)
+export const setHTML = curry((el, html) => el.innerHTML = html)
```

A continuación, construí un módulo inicial con algunas funciones impuras (side effects). Por el momento, añadí dos funciones básicas: una para seleccionar elementos del DOM y otra para cambiar el texto de un elemento utilizando su propiedad `innerHTML`. Para facilitar la programación funcional, opté por usar la librería [RamdaJS](https://ramdajs.com/), que ofrece utilidades potentes para trabajar con funciones puras.

### Aplicación Principal

```diff
# public/app.mjs

+import compose from "https://unpkg.com/ramda@0.30.1/es/compose.js"
+import flip from "https://unpkg.com/ramda@0.30.1/es/flip.js"
+import { getDOMEl, setHTML } from "./impure.mjs"
+const renderMsg = flip(setHTML)("Hello World")
+const idSelector = (id) => `#${id}`
+const app = compose(renderMsg, getDOMEl, idSelector)
+export default app
```

```diff
# public/index.mjs

+import app from './app.mjs'
+app('app')
```

Luego, creé un módulo que representa la aplicación en sí. Por ahora, su función es simple: mostrar un "Hello World" en el espacio definido en el HTML. Esta estructura modular nos permitirá crecer la aplicación de manera eficiente a medida que avancemos.

### Importmap

```diff
# views/index.ejs

    <main>
        <div id="app"></div>
    </main>
+    <script type="importmap">
+        {
+            "imports": {
+                "ramda/": "https://unpkg.com/ramda@0.30.1/es/",
+                "impure": "./impure.mjs"
+            }
+        }
+    </script>
    <script type="module" src="/index.mjs"></script>
</body>
</html>
```

```diff
# public/app.mjs

-import compose from "https://unpkg.com/ramda@0.30.1/es/compose.js"
-import flip from "https://unpkg.com/ramda@0.30.1/es/flip.js"
-import { getDOMEl, setHTML } from "./impure.mjs"
+import compose from "ramda/compose.js"
+import flip from "ramda/flip.js"
+import { getDOMEl, setHTML } from "impure"

const renderMsg = flip(setHTML)("Hello World")
const idSelector = (id) => `#${id}`
```

```diff
# public/impure.mjs

-import curry from "https://unpkg.com/ramda@0.30.1/es/curry.js"
+import curry from "ramda/curry.js"

export const getDOMEl = (sel) => document.querySelector(sel)
export const setHTML = curry((el, html) => el.innerHTML = html)
```

Para simplificar la gestión de dependencias, configuré un `importmap` en el HTML, lo que me permitió reemplazar las importaciones manuales antiguas. Tras este cambio, la aplicación seguía funcionando correctamente, lo que fue un buen indicio de que la configuración era sólida.

## Usando el Mensaje `HelloRequest`

### Importación del Módulo Generado por `protobufjs-cli`

```diff
# views/index.ejs

        {
            "imports": {
                "ramda/": "https://unpkg.com/ramda@0.30.1/es/",
-                "impure": "./impure.mjs"
+                "impure": "./impure.mjs",
+                "proto": "./proto.mjs",
+                "proto/com.joav": "./proto-es-bundle.js"
            }
        }
    </script>
```

```diff
# public/proto.mjs

import proto from "proto/com.joav"
console.log({proto});
```

Realicé una prueba inicial creando un nuevo módulo que importa el código generado por protobufjs y hace un log de lo exportado, para asegurarme de que el módulo `HelloRequest` se importa correctamente.

### Problemas con `protobuf/light`

{% include img-with-caption.html alt="Error de importación protobuf/light" img="/assets/images/post4-error.png" caption="Error de importación en consola" style="width: 100%; max-width: 1068px" class="align-center" %}

{% include img-with-caption.html alt="Importación protobuf/light" img="/assets/images/post4-import.png" caption="Importación protobuf/light" style="width: 100%; max-width: 784px" class="align-center" %}

La prueba inicial no tuvo éxito. El módulo generado intentaba importar la librería `protobuf/light`, pero en el entorno del navegador no existe tal módulo. Intenté configurar el `importmap` para incluir la librería, pero no funcionó correctamente, ya que `protobuf/light` no exporta un ES Module, sino que crea una variable global.

### Creación de un Wrapper para `protobuf/light`

```diff
# package.json

    "build": "npm run proto:build && tsc  && cp proto-bundle.js dist/proto-bundle.js",
    "start": "node dist/index.js",
    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js $(node scripts/get-proto-files.js)",
-    "proto:es": "pbjs -t json-module -w es6 --es6 -o public/proto-es-bundle.js $(node scripts/get-proto-files.js)",
+    "proto:es": "pbjs -t json-module -w es6 --es6 $(node scripts/get-proto-files.js) | sed 's/\\*\\sas\\s//g' > public/proto-es-bundle.js",
    "proto:types": "pbjs -t static-module $(node scripts/get-proto-files.js) | pbts -o proto-bundle.d.ts -",
    "proto:build": "npm run proto:to-json && npm run proto:types && npm run proto:es"
  },
```

```diff
# index.mjs

                "ramda/": "https://unpkg.com/ramda@0.30.1/es/",
                "impure": "./impure.mjs",
                "proto": "./proto.mjs",
-                "proto/com.joav": "./proto-es-bundle.js"
+                "proto/com.joav": "./proto-es-bundle.js",
+                "protobufjs/light": "./protobufjs-light-wrapper.mjs"
            }
        }
    </script>
```

```diff
# public/protobufjs-light-wrapper.mjs

+ import "https://cdn.jsdelivr.net/npm/protobufjs@7.4.0/dist/light/protobuf.min.js"

+ export default window.protobuf
```

Para resolver este problema, creé un wrapper que importa la librería y exporta la variable global como un módulo. Además, modifiqué la creación del módulo generado para que no utilizara la notación `* as`. Una vez hecho esto, el log funcionó correctamente, confirmando que el mensaje `HelloRequest` se importaba sin problemas.

### Creación y Uso de un Mensaje Proto

```diff
# public/proto.mjs

import proto from "proto/com.joav"
+import compose from "ramda/compose.js"

const {HelloRequest} = proto

-console.log({proto, HelloRequest});
+const helloRequestConstructor = (raw) => HelloRequest.create(raw)
+const helloRequestRaw = (name) => ({name});
+export const helloRequest = compose(helloRequestConstructor, helloRequestRaw);
```

```diff
# public/app.mjs

import compose from "ramda/compose.js"
import flip from "ramda/flip.js"
+import prop from "ramda/prop.js"
import { getDOMEl, setHTML } from "impure"
-import "proto"
+import {helloRequest} from "proto"

-const renderMsg = flip(setHTML)("Hello World")
+const renderMsg = flip(setHTML)
+const reqMsg = (name) => `Hello ${name}!`
+const renderReq = compose(renderMsg, reqMsg, prop("name"), helloRequest)('MyName')
const idSelector = (id) => `#${id}`

-const app = compose(renderMsg, getDOMEl, idSelector)
+const app = compose(renderReq, getDOMEl, idSelector)

export default app
```

Para finalizar esta primera etapa del desarrollo frontend, creé algunas funciones utilitarias para trabajar con el mensaje `HelloRequest`. Luego, modifiqué la aplicación para que pudiera imprimir el nombre guardado en un mensaje proto, confirmando así la creación correcta de mensajes desde el navegador.

## Resultado del Día 4

{% include img-with-caption.html alt="Resultado" img="/assets/images/post4-result.jpg" caption="Resultado del front" style="width: 100%; max-width: 577px" class="align-center" %}

Este post introduce con éxito el uso de Protocol Buffers en el frontend utilizando ES Modules y soluciones funcionales. En la próxima etapa, profundizaremos más en cómo integrar estos mensajes en la interacción usuario-servidor. ¡Nos vemos en el próximo post!
