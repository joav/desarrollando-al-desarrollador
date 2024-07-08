---
title: "Cómo compilar mensajes .proto con protobufjs"
date: 2024-07-02T17:00:00-05:00
categories:
  - Desarrollo fullstack
tags:
  - typescript
  - IDX
  - protobuff
keywords:
  - Protocol Buffers
  - Project IDX
  - node
  - expressjs
  - express
  - bash
description: Acompaña a un desarrollador web fullstack en su viaje de aprendizaje con Protocol Buffers y Project IDX, el IDE en la nube de Google. Descubre cómo crear una API con Node, Express y TypeScript, resolver problemas y explorar nuevas tecnologías en su primer post.
---

¡Bienvenidos al segundo post de la serie "Prueba Protocol Buffers con Node desde IDX"! En esta ocasión, hablaré sobre los avances en la construcción de nuestra API con Express, específicamente sobre la compilación de mensajes .proto usando protobufjs.

## Creación de scripts iniciales

Para empezar, me centré en los scripts necesarios para compilar los mensajes. Utilicé la documentación del CLI de protobufjs como guía.

### Script proto:to-json

El primer script transforma archivos .proto en un módulo de Node.js que compila los mensajes utilizando protobufjs. Este módulo genera una clase para cada mensaje definido, con funciones utilitarias como encode, decode, toObject, etc.

El código generado contiene dos partes:

1. **Definición de mensajes:** Protobufjs recibe un objeto con la definición de los mensajes parseados de los archivos .proto.

2. **Generación de constructor:** El resultado es un constructor con funciones utilitarias, que luego es exportado por el módulo.

Descubrí que el CLI solo permite pasar una lista de archivos .proto, pero no una carpeta o un set de archivos con un glob pattern. Afortunadamente, esto no fue un impedimento ya que solo tenemos un mensaje.

### Script proto:types

Este script genera un archivo TypeScript con los tipos necesarios para que el código entienda el módulo generado anteriormente.

### Script proto:build

Este script combina los dos pasos anteriores: primero genera el módulo y luego crea los tipos en TypeScript.

## Mejorando el script proto:to-json

Decidí mejorar el script proto:to-json para crear el módulo a partir de la ruta de la carpeta de los mensajes, pensando en la escalabilidad futura.

1. Escribí un script de Node.js que lista los archivos de la carpeta de mensajes e imprime por consola las rutas encontradas.

2. Este script se ejecuta desde la terminal con el binario de Node.js. Luego, mejoré el script proto:to-json utilizando bash scripts para ejecutar comandos y pasar su resultado de texto como parte de otro.

## Usando el código generado

Con todos los scripts listos, pude utilizar el módulo generado en el punto de entrada de la API. Modifiqué el script build de la aplicación para incluir un paso que ejecuta el build de proto.Además, añadí un fragmento de código probando un método del módulo generado.

### Preview cargando…

Al relanzar el preview, este se quedó cargando. Descubrí que faltaba copiar el módulo de proto a la carpeta dist. Después de hacerlo, el preview cargó correctamente, lo cual fue una prueba exitosa.

## Probando la API de protobuf

Para finalizar, hice un cambio en la API para que, en cada request, aleatoriamente lanzara un error o una respuesta exitosa. Utilicé varios métodos del código generado por protobufjs.

La idea era que respondiera con un error cuando el mensaje proto no pudiera validar un objeto, o con éxito enviando un objeto correcto que se serializa y se deserializa en la respuesta.

De esta manera, entendí mejor los métodos generados y cómo usarlos para los fines de esta serie.

## Resultado del día 2

Al final del día, los scripts para la compilación de mensajes .proto estaban listos y la API pasó una prueba exitosa. Esto facilitará el trabajo con los mensajes desde nuestra API Express.

## Bonus: Repositorio de GitHub

He creado un repositorio en GitHub donde se publicarán los avances commit a commit. Aquí podrán ver en detalle las decisiones a nivel de código.

creación de scripts iniciales
proto:to-json
 no permite usar glob pattern o una carpeta, se usa nombre del archivo
proto:types
proto:build

(Extra)permitir la creación de multiples mensajes
 script para listar archivos
  node
  bash $() - https://stackoverflow.com/questions/27472540/difference-between-and-in-bash   `it means run command and put its output here`

Usando el código generado
 Modificación de script build e importación y uso de los mensajes
 Se queda cargando el preview, faltaba copiar el build de proto a la carpeta dist

Probando la API de protobuff

conclusiones
