---
title: "Prueba Protocol Buffers con Node desde IDX - Día 1"
date: 2024-06-30T10:00:00-05:00
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
description: Acompaña a un desarrollador web fullstack en su viaje de aprendizaje con Protocol Buffers y Project IDX, el IDE en la nube de Google. Descubre cómo crear una API con Node, Express y TypeScript, resolver problemas y explorar nuevas tecnologías en su primer post.
seriename: Prueba Protocol Buffers con Node desde IDX
serie: protobuf-node
serieitemtitle: Día 1 - Pasos iniciales
---

## Introducción

Bienvenidos al primer post de mi blog. Como menciono en la sección de '[about](https://joav.github.io/desarrollando-al-desarrollador/about/)', aquí exploro nuevas herramientas, pruebo cosas y aprendo en el proceso.


## Motivación e Inspiración

Hace ya varios meses encontré menciones a una tecnología llamada [Protocol Buffers](https://protobuf.dev/), creada por Google, que es una nueva manera de serializar datos, mejorando su tamaño al convertirlos en binarios.

Algunas semanas atrás, me encontré con publicaciones al respecto en varias redes sociales: videos, reels, shorts, entre otros. Gracias a estas publicaciones, me volvió a picar el bichito de la curiosidad y decidí aprender más sobre el tema.


## Aprendiendo Protocol Buffers con Project IDX

En esta ocasión he decidido 'matar dos pájaros de un solo tiro'. Quiero aprovechar la oportunidad de aprender sobre [Protocol Buffers](https://protobuf.dev/) utilizando [Project IDX](https://idx.dev/), el nuevo IDE en la nube también de Google. De esta manera, aprendo sobre una nueva tecnología y me familiarizo con una herramienta que podría utilizar en el futuro.


## Creación de la cuenta de IDX

Para crear una cuenta en IDX, visita su [página web](https://idx.dev/), inicia sesión con una cuenta de Google y sigue los pasos indicados. En mi caso, utilicé mi cuenta personal y en pocos clics ya tenía acceso a IDX.


## Creación del primer proyecto en IDX - Problemas en el paraíso

Lo primero que encuentras en IDX es un listado de posibles proyectos que puedes crear, con variedad en lenguajes de programación y frameworks. Incluso puedes escoger crear un espacio en blanco para configurar a tu gusto.

### Objetivos

Esta vez escogí un proyecto de Node con Express y TypeScript, con los siguientes objetivos:
* Crear una API sencilla con un solo endpoint que consuma y produzca Protocol Buffers.
* Crear una página web que consuma la API y muestre el resultado.

### Creación del espacio de trabajo

La creación del proyecto base y el despliegue del ambiente puede tardar algunos minutos. Cuando todo estuvo listo, se mostró una previsualización del resultado, gracias a la configuración de la plantilla creada por IDX.

### Primeros errores

Aquí encontré un primer inconveniente: la previsualización arrojaba un error no controlado sobre el sistema de Views de la plantilla EJS. La configuración inicial hace un build y lo deja en una carpeta llamada 'dist', pero no se tuvo en cuenta que el apuntamiento a la carpeta de las vistas cambiaba con el build.

```js
// /tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "sourceMap": true,
    "outDir": "dist"
  }
}
```

```ts
// /index.ts
const app = express();
const port = parseInt(process.env.PORT) || process.argv[3] || 8080;

app.use(express.static(path.join(__dirname, 'public')))
  .set('views', path.join(__dirname, 'views'))
  .set('view engine', 'ejs');
```

#### Solución

Modifiqué el código, pero no sucedió nada en la previsualización. Entiendo que la plantilla inicial no tiene hot reloading. Mi intuición y experiencia previa me llevaron a abrir el panel de comandos con 'shift + CMD + P' (o 'shift + ctrl + P' en otros casos). Allí encontré un comando que recarga por completo la previsualización, lo cual funcionó y cargó la página inicial.

```diff
const app = express();
const port = parseInt(process.env.PORT) || process.argv[3] || 8080;
+const basePath = path.join(__dirname, '..');
   
-app.use(express.static(path.join(__dirname, 'public')))
+app.use(express.static(path.join(basePath, 'public')))
-  .set('views', path.join(__dirname, 'views'))
+  .set('views', path.join(basePath, 'views'))
  .set('view engine', 'ejs');
```


## Probufjs - Instalación y primeros mensajes

Con el entorno listo, instalé las librerías necesarias para manejar Protocol Buffers en la API. Usé [protobufjs](https://www.npmjs.com/package/protobufjs) y su [CLI](https://www.npmjs.com/package/protobufjs-cli) para convertir los mensajes .proto a JSON. Luego, creé el primer archivo .proto definiendo los mensajes de entrada y salida de la API. Este fue el progreso del primer día.

```proto
// hello.proto
syntax = "proto3";

message HelloRequest {
  string name = 1;
}

message HelloResponse {
  string message = 1;
}
```


## Conclusión

En esta primera entrada del día 1, presenté rápidamente las intenciones del blog y di inicio a un pequeño proyecto para aprender sobre Protocol Buffers y Project IDX. Espero continuar pronto con el proyecto y tener más entradas, cumpliendo los objetivos planteados.

Gracias por leer, ¡hasta la próxima!
