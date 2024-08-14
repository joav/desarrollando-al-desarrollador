---
title: "Cómo enviar y recibir protobuf con Express"
date: 2024-08-13T20:00:00-05:00
last_modified_at: 2024-08-14T08:30:00-05:00
categories:
  - Desarrollo-full-stack
tags:
  - typescript
  - IDX
  - protobuf
description: Configura una API para enviar y recibir mensajes protobuf con Node.js y Express. Aprende sobre Content-Type, headers, y pruebas con CURL.
serie: protobuf-node
serieitemtitle: Día 3 - Cómo enviar y recibir protobuf con Express
---

¡Bienvenidos al nuevo post de la serie "Prueba Protocol Buffers con Node desde IDX"! Hoy hablaré sobre cómo manejar las solicitudes y respuestas utilizando Protocol Buffers en nuestra API, completando así el [primer objetivo](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/prueba-protobuf-node-dia-1/#objetivos) de esta serie. Ya tenemos lista una API sencilla.

## Content-Type y Formato de un Protobuf

```diff
# index.ts

app.get('/api', (req, res) => {
-  const plainObj: any = Math.random() > 0.5 ? failObj : okObject;

-  const errMsg = HelloRequest.verify(plainObj);
-  if (errMsg)
-    return res.status(400).send('API_ERROR: ' + errMsg);
+  const messageFromPlain = HelloResponse.create({
+    message: "Custom Message"
+  });
+  const buffer = HelloResponse.encode(messageFromPlain).finish();

-  const messageFromPlain = HelloRequest.create(plainObj);
-  const buffer = HelloRequest.encode(messageFromPlain).finish();
-  const finalMessage = HelloRequest.decode(buffer);

-  res.json(HelloRequest.toObject(finalMessage));
+  res.header('Content-Type', 'application/octet-stream');
+  res.send(buffer);
});
```

Lo más común en una API es enviar y recibir texto en formato "application/json", pero en esta ocasión, trabajaremos con datos binarios, que es el formato en el que se producen los Protocol Buffers.

La API de los mensajes .proto compilados en el [post anterior](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/compilar-mensajes-proto/) utiliza el método `encode` para producir un Buffer nativo de Node. Con el método `send` de Express, podemos enviar estos buffers.

Lo siguiente es enviar un encabezado informando el `Content-Type`. Primero, usé `application/octet-stream`, que, según la documentación de [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types), se utiliza para tipos de archivos desconocidos o archivos binarios.

### Content-Type: application/protobuf

```diff
# intex.ts

app.get('/api', (req, res) => {
  const messageFromPlain = HelloResponse.create({
    message: "Custom Message"
  });
  const buffer = HelloResponse.encode(messageFromPlain).finish();

-  res.header('Content-Type', 'application/octet-stream');
+  res.header('Content-Type', 'application/protobuf; proto=com.joav.HelloResponse');
  res.send(buffer);
});
```

Al investigar un poco más, encontré que no hay una definición única para el tipo de contenido de los Protocol Buffers. Identifiqué las siguientes opciones:

- `application/protobuf; proto=org.some.Message`
- `application/vnd.google.protobuf`
- `application/x-protobuf; messageType=x.y.Z`

Para el propósito de este post, seleccioné `application/protobuf` y reemplacé "org.some.Message" con mi usuario de GitHub y el nombre del mensaje de respuesta "HelloResponse".

## Recibiendo Mensajes Protobuf Binarios

### Preparando el Binario del Request

```diff
app.get('/api', (req, res) => {
-  const messageFromPlain = HelloResponse.create({
-    message: "Custom Message"
+  const messageFromPlain = HelloRequest.create({
+    name: 'John Doe'
  });
-  const buffer = HelloResponse.encode(messageFromPlain).finish();
+  const buffer = HelloRequest.encode(messageFromPlain).finish();

-  res.header('Content-Type', 'application/protobuf; proto=com.joav.HelloResponse');
+  res.header('Content-Type', 'application/protobuf; proto=com.joav.HelloRequest');
  res.send(buffer);
});
```

Para empezar a recibir mensajes, creé un ejemplo de solicitud. Cambié el mensaje `HelloResponse` por `HelloRequest` y guardé el resultado binario en un archivo sin extensión. Con este ejemplo, procedí a modificar el código para recibirlo.

{% include img-with-caption.html alt="Ejemplo request" img="/assets/images/post3-request-example.jpg" caption="Ejemplo de request visto desde GitHub" style="width: 161px" class="align-center" %}

### Recibiendo el Mensaje: Leer los Datos del Body

```diff
# index.ts

import * as express from "express";
import * as path from "path";
-import { HelloRequest, HelloResponse } from "./proto-bundle";
+import { HelloRequest, HelloResponse } from "./proto-bundle";
+import * as $protobuf from "protobufjs";

-app.get('/api', (req, res) => {
-  const messageFromPlain = HelloRequest.create({
-    name: 'John Doe'
-  });
-  const buffer = HelloRequest.encode(messageFromPlain).finish();
-
-  res.header('Content-Type', 'application/protobuf; proto=com.joav.HelloRequest');
-  res.send(buffer);
+app.post('/api', (req, res) => {
+  try {
+    console.log( req.body )
+    const helloRequest = HelloRequest.decode(req.body);
+    const messageFromPlain = HelloResponse.create({
+      message: `Hello ${helloRequest.name}`
+    });
+    const buffer = HelloResponse.encode(messageFromPlain).finish();
+    res.header('Content-Type', 'application/protobuf; proto=com.joav.HelloResponse');
+    res.send(buffer);
+  } catch (error) {
+    if (error instanceof $protobuf.util.ProtocolError) {
+      res.status(400)
+        .send('API_ERROR: ' + error.message+typeof req.body);
+      return;
+    }
+    res.status(500)
+      .send('API_ERROR: ' + error.message);
+  }
});
```

Inicié adaptando el código para recibir un mensaje `HelloRequest`, validarlo y construir un mensaje `HelloResponse` utilizando los datos del mensaje de entrada.

En mi experiencia, nunca había leído datos binarios en una solicitud de Express. En el primer intento, intenté obtener los datos directamente del body, pero esto arrojó un error 400 con el mensaje `API_ERROR: illegal buffer`, indicando que algo estaba haciendo mal.

### Recibiendo el Mensaje: Utilizando el Body Parser Raw

```diff
# index.ts

+app.use(express.raw({type: 'application/protobuf; proto=com.joav.HelloRequest'}));
```

Recordé que para datos en formato JSON, se debe usar el body parser de JSON. Añadí un middleware de Express para leer los datos en formato raw, que se supone guarda los datos en un Buffer en el body, pero esto también arrojó el mismo error.

### Recibiendo el Mensaje: Leer los Datos por Medio del Evento Data

```diff
# index.ts

-app.use(express.raw({type: 'application/protobuf; proto=com.joav.HelloRequest'}));
+app.use((req, _, next) => {
+  let buffer = [];
+  req.on('data', (chunk) => {
+    buffer.push(chunk);
+  });
+  req.on('end', () => {
+    req.body = Buffer.concat(buffer);
+    next();
+  });
+  req.on('error', (err) => {
+    next(err);
+  });
+});
```

Como última medida, escribí un middleware que leyera los datos del request por medio del evento `data`, ya que este extiende de un `Stream.readable`. Los datos se acumulan en un array de Buffer para finalmente concatenarlos en un solo Buffer y guardarlo en el body.

De esta manera, la API ya está en capacidad de leer y responder los mensajes definidos.

{% include img-with-caption.html alt="Ejemplo de response exitoso" img="/assets/images/post3-success-response-example.jpg" caption="Ejemplo de response exitoso visto desde GitHub" style="width: 309px" class="align-center" %}

### Cómo Hice las Pruebas desde IDX: CURL

Dado que el objetivo de esta serie es explorar IDX y los Protocol Buffers, decidí probar la API directamente desde la terminal de IDX usando CURL, que ya está preinstalado en la instancia.

```bash
curl --request POST -H "Content-Type:application/protobuf; proto=com.joav.HelloRequest" --data-binary @examples/request  http://localhost:9002/api > response
```

El comando envía como datos binarios [el request que creamos antes](#preparando-el-binario-del-request) mediante el método POST, y el resultado se imprime en un archivo llamado `response`.

## Resultado del Día 3

En conclusión, la API está preparada para recibir y responder con los mensajes .proto, completando así el primer objetivo. Podemos avanzar al siguiente paso: construir el frontend que consuma esta API enviando y recibiendo protobufs.

## Bonus: Middlewares

Como bonus, he creado dos middlewares y refactorizado el código, separando el parser y creando un puente para llevar la lógica de validación, codificación y decodificación de mensajes proto fuera de las rutas.

* [proto.parser.ts](https://github.com/joav/poc-node-protobuf/blob/main/proto.parser.ts)
* [proto.middleware.ts](https://github.com/joav/poc-node-protobuf/blob/main/proto.middleware.ts)
