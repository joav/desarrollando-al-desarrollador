---
title: "Cómo hacer requests con protobuf desde el frontend"
date: 2024-10-29T17:00:00-05:00
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
description: Finalizamos la serie implementando un frontend interactivo que envía y recibe mensajes Protobuf desde la API. ¡Descubre el resultado!
header:
  teaser: /assets/images/post5-result.jpg
serie: protobuf-node
serieitemtitle: Día 5 - Cómo hacer requests con protobuf desde el frontend
---

¡Bienvenidos de nuevo a la serie "Prueba Protocol Buffers con Node desde IDX"! En este último post, completamos los objetivos planteados en el [primer post](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/prueba-protobuf-node-dia-1/), con una aplicación frontend que envía requests con mensajes Protobuf y muestra en pantalla las respuestas de la API.

## Diseño de la Aplicación Frontend

Para la interacción del usuario, decidí crear una interfaz estilo consola o terminal, que además de mostrar instrucciones, permite al usuario responder a las solicitudes del sistema.

Dividí la aplicación en estos componentes principales:

* **App**: Orquesta toda la funcionalidad.
* **Renderer**: Define los textos que se mostrarán en pantalla.
* **Console**: Renderiza los textos en la interfaz estilo consola.
* **User**: Recibe la entrada del usuario.
* **Events**: Administra los eventos de la aplicación.
* **Fetcher**: Maneja los requests a la API.

### Paso 1: Estructura HTML y estilos

{% include img-with-caption.html alt="Resultado del paso 1" img="/assets/images/post5-step-1.jpg" caption="Resultado del paso 1" style="width: 100%; max-width: 180px" class="align-center" %}

```html
<!-- Ejemplo de código -->
<div class="line system">Hello BLOG! Hello BLOG!</div>
<div class="line system system--success">Hello BLOG!</div>
<div class="line line--input">some input</div>
```

```css
:root {
    --base-bright-white: #F2F2F2;
    --base-pure-white: #FFFFFF;
    --base-bright-green: #16C60C;

    --base-fg: var(--base-white);/* Mensajes del sistema */
    --input-color: var(--base-pure-white);/* Input del usuario */
    --success-color: var(--base-bright-green);/* Mensajes del API */
}
```

Definí los estilos de la consola para diferenciar los tipos de mensajes:

* **Sistema**: Mensajes de la aplicación, identificados con `[S]` y en gris.
* **Usuario**: Entrada del usuario, identificada con `[U]` y en blanco.
* **Respuesta Exitosa**: Respuestas de la API, en verde y también con `[S]`.

### Paso 2: Mostrar mensajes en la Consola

{% include img-with-caption.html alt="Resultado del paso 2" img="/assets/images/post5-step-2.gif" caption="Resultado del paso 2" style="width: 100%; max-width: 1568px" class="align-center" %}

{% include img-with-caption.html alt="Diagrama de flujo" img="/assets/images/post5-step-2-diagram.png" caption="Diagrama de flujo del paso 2" style="width: 100%; max-width: 1312px" class="align-center" %}

A continuación, trabajé en la funcionalidad de la consola, encargada de renderizar secuencialmente una lista de mensajes en pantalla. Para simular la escritura en tiempo real, utilicé recursividad para imprimir cada mensaje uno después del otro.

### Paso 3: Entrada de Usuario y Eventos

{% include img-with-caption.html alt="Resultado del paso 3" img="/assets/images/post5-step-3.png" caption="Resultado del paso 3" style="width: 100%; max-width: 737px" class="align-center" %}

Una vez lista la renderización, añadí la funcionalidad para que el usuario pueda comunicarse con el sistema. Para ello, configuré la última línea de la consola como editable mediante la propiedad `contentEditable` en HTML, y programé un evento para capturar cuando el usuario presiona "Enter". Este evento desencadena una acción en el componente "Events".

## Comunicación con la API

### Creación del Request

```js
// impure.mjs

export const doFetch = curry((url, options) => fetch(url, options))
export const toArrayBuffer = (res) => res.arrayBuffer()
```

```js
// proto.mjs
const helloRequestRaw = (name) => ({name})
export const helloRequest = compose(helloRequestConstructor, helloRequestRaw)
export const encode = (msg) => HelloRequest.encode(msg).finish()
export const decode = (msg) => HelloResponse.decode(msg)
```

```js
// fetcher.mjs

import compose from "ramda/compose.js"
import prop from "ramda/prop.js"
import {doFetch, trace} from "impure"
import {helloRequest, encode} from "proto"

const api = doFetch('api')
const options = (data) => ({method: 'POST', body: data, headers: {'Content-Type': 'application/protobuf; proto=com.joav.HelloRequest'}})
const apiOptions = compose(
    trace('options'),
    options,
    trace('encode result'),
    encode,
    helloRequest
)
export const fetcher = compose(
    api,
    apiOptions,
    trace('name for message'),
    prop('detail')
)
```

Para enviar un request desde el frontend hacia la API, utilizamos el método `encode` definido en [el post anterior](https://joav.github.io/desarrollando-al-desarrollador/categorias/desarrollo-full-stack/importar-protobuf-desde-navegador/). La API espera recibir el contenido con el tipo `application/protobuf; proto=com.joav.HelloRequest`.

### Recibir y parsear la Respuesta

```js
const responseParse = compose(
    andThen(prop('message')),
    andThen(decode),
    andThen(toUint8Arr),
    andThen(toArrayBuffer)
)
```

Para procesar la respuesta de la API, realizamos varias transformaciones:

1. Obtener la respuesta como `ArrayBuffer`.
2. Convertir el buffer en `Uint8Array`.
3. Decodificar el buffer para obtener un objeto `HelloResponse`.

### Mostrando la respuesta en la consola

{% include img-with-caption.html alt="Resultado final" img="/assets/images/post5-result.gif" caption="Resultado final" style="width: 100%; max-width: 582px" class="align-center" %}

Para finalizar, al recibir la respuesta de la API, se crea una solicitud de renderizado con el mensaje retornado, acompañado de mensajes motivando al usuario a seguir interactuando. ¡Y listo! Tenemos una aplicación full-stack que se comunica mediante mensajes Protobuf.

## Conclusiones

Con este último post, completamos el desarrollo de una aplicación frontend que interactúa con una API usando Protocol Buffers. A lo largo de la serie, hemos cubierto cómo configurar Protobuf en Node.js, compilar mensajes proto, y establecer una comunicación eficaz entre el cliente y el servidor. La interfaz tipo consola no solo facilita la interacción del usuario, sino que también demuestra la eficiencia y velocidad de Protobuf en la transmisión de datos.

Este recorrido muestra cómo Protocol Buffers optimiza la comunicación en aplicaciones modernas, mejorando el rendimiento y la experiencia del usuario. Con esta base sólida, estamos listos para explorar nuevas funcionalidades y seguir innovando en nuestras aplicaciones. ¡Gracias por acompañarme en esta serie y hasta la próxima!

***Nota:*** No olviden pasarse por [el repositorio](https://github.com/joav/poc-node-protobuf/) para ver el resultado completo.
