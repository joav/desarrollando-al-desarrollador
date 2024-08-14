---
title: "Cómo enviar y recibir protobuf con Express"
date: 2024-08-13T20:00:00-05:00
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

¡Bienvenidos al nuevo post de la serie "Prueba Protocol Buffers con Node desde IDX"! Hoy hablaré sobre los request y response con protobuf, y daremos fin al [primer objetivo](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/prueba-protobuf-node-dia-1/#objetivos) de esta seríe, ya tenemos lista la API sencilla.

## Content-Type y formato de un protobuf

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

Lo más común en una API es enviar y recibir texto en formato "application/json", pero esta vez este no es el caso, esta vez nos interesa que la API trabaje con datos binarios, pues es el formato en el que se producen los protobuf.

El API de los mensajes .proto compilados en el [post anterior](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/compilar-mensajes-proto/) por medio de su metodo `encode` produce un Buffer nativo de Node y por medio del método `send` del response de Express se puede enviar los buffers.

Lo que nos queda es enviar un header informando el `Content-Type`, primero use `application/octet-stream` que según la documentación de [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types) se usa para tipos de archivos desconocidos o archivos binarios.

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

Buscando un poco en internet encontré que no hay una definición única para el content type de los protobuf, identifiqué los siguientes:
* application/protobuf; proto=org.some.Message
* application/vnd.google.protobuf
* application/x-protobuf; messageType=x.y.Z

Para el próposito de este post seleccione `application/protobuf` y reemplace "org.some.Message", por mi usuario de github y el nombre del mensaje response "HelloResponse".

## Recibiendo mensajes protobuf binarios

### Preparando el binario del request

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

Para iniciar a recibir los mensajes, primero cree un ejemplo de request, para esto cambie el mensaje `HelloResponse` por `HelloRequest` y guarde el resultado binario en un archivo sin extensión. Con este ejemplo me dispuse a modificar el código para recibirlo.

{% include img-with-caption.html alt="Ejemplo request" img="/assets/images/post3-request-example.jpg" caption="Ejemplo de request visto desde GitHub" style="width: 161px" class="align-center" %}

### Recibiendo el mensaje: Leer los datos del body

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

Inicie adaptando el código para recibir un mensaje HelloRequest, validarlo y finalmente construir un mensaje HelloResponse utilizando los datos del mensaje de entrada.

En mi experiencia nunca he leido datos binarios en un request de Express, en el primer intento por leer los datos intento obtenerlos directo del body, pero esta prueba me arroja un error 400 y de mensaje de error `API_ERROR: illegal buffer`. Esto me dice que algo estoy haciendo mal.

### Recibiendo el mensaje: Utilizando el body parser raw

```diff
# index.ts

+app.use(express.raw({type: 'application/protobuf; proto=com.joav.HelloRequest'}));
```

Lo primero que recuerdo es que para el caso de los datos en formato JSON se debe utilizar el body parser de json, así que añado un middleware directo de Express que lee los datos en formato raw que se supone guarda en el body los datos en un Buffer, pero esto me arrojo el mismo error.

### Recibiendo el mensaje: Leer los datos por medio del evento data

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

Como última medida escribi un middleware que leyera los datos del request por medio del evento `data` gracias a que este extiende de un `Stream.readable`, los datos se acumulan en un array de Buffer para finalmente concatenarlos en un solo Buffer y guardarlo en el body.

De esta manera el API ya está en capacidad de leer y responder los mensajes definidos.

{% include img-with-caption.html alt="Ejemplo de response exitoso" img="/assets/images/post3-success-response-example.jpg" caption="Ejemplo de response exitoso visto desde GitHub" style="width: 309px" class="align-center" %}

### Como hice las pruebas desde IDX: CURL

Ya que la finalidad de esta serie es explorar IDX y los Protocol Buffers, decidí que para probar el API podía hacerlo directo desde la terminal de IDX y para esto me apolle en CURL, el cual ya esta preinstalado en la instancia.

```bash
curl --request POST -H "Content-Type:application/protobuf; proto=com.joav.HelloRequest" --data-binary @examples/request  http://localhost:9002/api > response
```

El comando que use envía como datos binarios [el request que creamos antes](#preparando-el-binario-del-request) por medio del método POST y por último el resultado lo imprime en un archivo llamado `response`.

## Resultado del día 3

Como conclusión, ya esta preparada la API para recibir y responder con los mensajes .proto, esto quiere decir que ya esta listo el primer objetivo y podemos avanzar al siguiente: Construir el front que consuma esta API enviando y recibiendo protobuf.

## Bonus: Middlewares

Como bonus del post, he creado dos middlewares y refactorizado el código, separando el parser y creando un puente para llevar la lógica de validación, encode y decode de mensajes proto por fuera de las rutas.

* [proto.parser.ts](https://github.com/joav/poc-node-protobuf/blob/main/proto.parser.ts)
* [proto.middleware.ts](https://github.com/joav/poc-node-protobuf/blob/main/proto.middleware.ts)