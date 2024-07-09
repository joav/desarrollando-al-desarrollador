---
title: "Cómo compilar mensajes .proto con protobufjs"
date: 2024-07-08T10:30:00-05:00
categories:
  - Desarrollo fullstack
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
  - bash
description: Aprende a compilar mensajes .proto con protobufjs en Node.js y Express. Descubre scripts esenciales, mejoras para el API. Sigue los avances en nuestro repositorio de GitHub.
seriename: Prueba Protocol Buffers con Node desde IDX
serie: protobuf-node
serieitemtitle: Día 2 - Cómo compilar mensajes .proto con protobufjs
---

¡Bienvenidos al segundo post de la serie "Prueba Protocol Buffers con Node desde IDX"! En esta ocasión, hablaré sobre los avances en la construcción de nuestra API con Express, específicamente sobre la compilación de mensajes .proto usando protobufjs.

## Creación de scripts iniciales

Para empezar, me centré en los scripts necesarios para compilar los mensajes. Utilicé la documentación del CLI de protobufjs como guía.

### Script proto:to-json

```diff
# package.json

  "scripts": {
    "dev": "nodemon --watch --ext 'ts,json' --exec 'npm run build && npm run start'",
    "build": "tsc",
-    "start": "node dist/index.js"
+    "start": "node dist/index.js",
+    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js ./proto-messages/hello.proto"
  },
  "keywords": [],
  "author": "",
```

El primer script transforma archivos .proto en un módulo de Node.js que compila los mensajes utilizando protobufjs. Este módulo genera una clase para cada mensaje definido, con funciones utilitarias como encode, decode, toObject, etc.

El código generado contiene dos partes:

1. **Definición de mensajes:** Protobufjs recibe un objeto con la definición de los mensajes parseados de los archivos .proto.

2. **Generación de constructor:** El resultado es un constructor con funciones utilitarias, que luego es exportado por el módulo.

Descubrí que el CLI solo permite pasar una lista de archivos .proto, pero no una carpeta o un set de archivos con un glob pattern. Afortunadamente, esto no fue un impedimento ya que solo tenemos un mensaje.

```bash
# Comando
# "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js proto-message/*.proto"

# Ouput

> pbjs -t json-module -w commonjs -o proto-bundle.js proto-message/*.proto

/home/user/poc-node-protobuf/node_modules/protobufjs-cli/pbjs.js:254
            throw err;
            ^

TypeError: Cannot read properties of undefined (reading 'trim')
    at json_module (/home/user/poc-node-protobuf/node_modules/protobufjs-cli/targets/json-module.js:28:69)
    at callTarget (/home/user/poc-node-protobuf/node_modules/protobufjs-cli/pbjs.js:328:9)
    at Object.main (/home/user/poc-node-protobuf/node_modules/protobufjs-cli/pbjs.js:248:13)
    at Object.<anonymous> (/home/user/poc-node-protobuf/node_modules/protobufjs-cli/bin/pbjs:4:16)
    at Module._compile (node:internal/modules/cjs/loader:1376:14)
    at Module._extensions..js (node:internal/modules/cjs/loader:1435:10)
    at Module.load (node:internal/modules/cjs/loader:1207:32)
    at Module._load (node:internal/modules/cjs/loader:1023:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:135:12)
    at node:internal/main/run_main_module:28:49

Node.js v20.11.1

```

### Script proto:types

```diff
# package.json

    "dev": "nodemon --watch --ext 'ts,json' --exec 'npm run build && npm run start'",
    "build": "tsc",
    "start": "node dist/index.js",
-    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js ./proto-messages/hello.proto"
+    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js ./proto-messages/hello.proto",
+    "proto:types": "pbjs -t static-module ./proto-messages/hello.proto | pbts -o proto-bundle.d.ts -"
  },
  "keywords": [],
  "author": "",
```

Este script genera un archivo TypeScript con los tipos necesarios para que el código entienda el módulo generado anteriormente.

### Script proto:build

```diff
# package.json

    "build": "tsc",
    "start": "node dist/index.js",
    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js ./proto-messages/hello.proto",
-    "proto:types": "pbjs -t static-module ./proto-messages/hello.proto | pbts -o proto-bundle.d.ts -"
+    "proto:types": "pbjs -t static-module ./proto-messages/hello.proto | pbts -o proto-bundle.d.ts -",
+    "proto:build": "npm run proto:to-json && npm run proto:types"
  },
  "keywords": [],
  "author": "",
```

Este script combina los dos pasos anteriores: primero genera el módulo y luego crea los tipos en TypeScript.

## Mejorando el script proto:to-json

Decidí mejorar el script proto:to-json para crear el módulo a partir de la ruta de la carpeta de los mensajes, pensando en la escalabilidad futura.

1. Escribí un script de Node.js que lista los archivos de la carpeta de mensajes e imprime por consola las rutas encontradas.

2. Este script se ejecuta desde la terminal con el binario de Node.js. Luego, mejoré el script proto:to-json utilizando bash scripts para ejecutar comandos y pasar su resultado de texto como parte de otro.

```js
// scripts/get-proto-files.js

const fs = require('fs');
const path = require('path');

const result = fs.readdirSync(path.join(__dirname, '..', 'proto-messages'));

console.log("./proto-messages/" + result.join(" ./proto-messages"));
```
```diff
# package.json

    "dev": "nodemon --watch --ext 'ts,json' --exec 'npm run build && npm run start'",
    "build": "tsc",
    "start": "node dist/index.js",
-    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js ./proto-messages/hello.proto",
-    "proto:types": "pbjs -t static-module ./proto-messages/hello.proto | pbts -o proto-bundle.d.ts -",
+    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js $(node scripts/get-proto-files.js)",
+    "proto:types": "pbjs -t static-module $(node scripts/get-proto-files.js) | pbts -o proto-bundle.d.ts -",
    "proto:build": "npm run proto:to-json && npm run proto:types"
  },
  "keywords": [],
```

## Usando el código generado

Con todos los scripts listos, pude utilizar el módulo generado en el punto de entrada de la API. Modifiqué el script build de la aplicación para incluir un paso que ejecuta el build de proto. Además, añadí un fragmento de código probando un método del módulo generado.

```diff
# package.json

  "main": "index.js",
  "scripts": {
    "dev": "nodemon --watch --ext 'ts,json' --exec 'npm run build && npm run start'",
-    "build": "tsc",
+    "build": "npm run proto:build && tsc",
    "start": "node dist/index.js",
    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js $(node scripts/get-proto-files.js)",
    "proto:types": "pbjs -t static-module $(node scripts/get-proto-files.js) | pbts -o proto-bundle.d.ts -",
```

```diff
# index.ts

import * as express from "express";
import * as path from "path";
+import { HelloRequest } from "./proto-bundle";

// More code...

+const helloReq = HelloRequest.create({ name: 'World' });

// More code...

app.get('/api', (req, res) => {
-  res.json({"msg": "Hello world"});
+  res.json({"msg": "Hello" + helloReq.name});
});
```

### Preview cargando…

Al relanzar el preview, este se quedó cargando. Descubrí que faltaba copiar el módulo de proto a la carpeta dist. Después de hacerlo, el preview cargó correctamente, lo cual fue una prueba exitosa.

```diff
# package.json

  "main": "index.js",
  "scripts": {
    "dev": "nodemon --watch --ext 'ts,json' --exec 'npm run build && npm run start'",
-    "build": "npm run proto:build && tsc",
+    "build": "npm run proto:build && tsc && cp proto-bundle.js dist/proto-bundle.js",
    "start": "node dist/index.js",
    "proto:to-json": "pbjs -t json-module -w commonjs -o proto-bundle.js $(node scripts/get-proto-files.js)",
    "proto:types": "pbjs -t static-module $(node scripts/get-proto-files.js) | pbts -o proto-bundle.d.ts -",
```

```json
# Resultado del api
{
  "msg": "Hello World"
}
```

## Probando la API de protobuf

Para finalizar, hice un cambio en la API para que, en cada request, aleatoriamente lanzara un error o una respuesta exitosa. Utilicé varios métodos del código generado por protobufjs.

```diff
# index.ts

-const helloReq = HelloRequest.create({ name: 'World' });
+const failObj = "";
+const okObject = {name: 'World'};

// More code...

app.get('/api', (req, res) => {
-  res.json({"msg": "Hello" + helloReq.name});
+  const plainObj: any = Math.random() > 0.5 ? failObj : okObject;
+
+  const errMsg = HelloRequest.verify(plainObj);
+  if (errMsg)
+    return res.status(400).send('API_ERROR: ' + errMsg);
+
+  const messageFromPlain = HelloRequest.create(plainObj);
+  const buffer = HelloRequest.encode(messageFromPlain).finish();
+  const finalMessage = HelloRequest.decode(buffer);
+
+  res.json(HelloRequest.toObject(finalMessage));
});
```

La idea era que respondiera con un error cuando el mensaje proto no pudiera validar un objeto, o con éxito enviando un objeto correcto que se serializa y se deserializa en la respuesta.

De esta manera, entendí mejor los métodos generados y cómo usarlos para los fines de esta serie.

## Resultado del día 2

Al final del día, los scripts para la compilación de mensajes .proto estaban listos y la API pasó una prueba exitosa. Esto facilitará el trabajo con los mensajes desde nuestra API Express.

## Bonus: Repositorio de GitHub

He creado un [repositorio en GitHub](https://github.com/joav/poc-node-protobuf/) donde se publicarán los avances commit a commit. Aquí podrán ver en detalle las decisiones a nivel de código.
