---
title: "¿Qué son los Microfrontend? - 3 años con ellos"
date: 2025-03-07T12:00:00-05:00
categories:
  - Microfrontends
tags:
  - frontend
  - microfrontends
  - arquitectura frontend
keywords:
  - microfrontends
  - frontend
  - web
  - single-spa
  - module federation
description: Inicio una nueva serie sobre microfrontends basada en mis 3 años de experiencia. Exploramos qué son, sus arquitecturas, ventajas y cómo implementarlos.
header:
  teaser: /assets/images/microfrontends/post-1/teaser.svg
serie: experiencia-microfrontends
serieitemtitle: ¿Qué son los Microfrontend? - 3 años con ellos
---

¡Hola de nuevo! Hoy inicio una nueva serie de posts sobre microfrontends, basada en mi experiencia de tres años desarrollando con esta arquitectura. A lo largo de esta serie, compartiré lo que he aprendido, los desafíos que he enfrentado y las mejores prácticas que he descubierto.

## ¿Qué son los microfrontends?

Antes de entrar en detalles, revisemos qué se entiende por microfrontends.

### Lo que pensaba antes

Hace tres años, en una prueba técnica, me preguntaron qué eran los microfrontends y en qué casos se podrían aplicar. No tenía mucha idea del tema, así que busqué información y respondí lo siguiente:

> Hablando en el caso de las tecnologías web y los navegadores modernos, se podría implementar por medio de herramientas como los módulos de javascript, el API de Custom Elements. Se pueden crear multiples repositorios o proyectos por cada micro frontend a crear y cada uno debería encargarse de tareas específicas y consumir servicios específicos.  
<br>El mejor escenario que yo veo es en el caso de proyectos grandes con equipos grandes, en los cuales se puedan asignar pequeños equipos a cada microfrontend, esto sería bueno ya que todos podrían trabajar en paralelo y se conseguiría una aplicación completa de pronto en menos tiempo que un solo equipo trabajando en una sola aplicación.

### Qué pienso ahora

Después de trabajar con microfrontends, mi visión ha evolucionado. Ahora los defino así:

> Un microfrontend es una pieza de software independiente que se integra con otras para formar una aplicación completa. Se ejecuta en el frontend, interactuando directamente con los usuarios.  
<br>Su fortaleza radica en su desacoplamiento, lo que permite que crezca y evolucione sin afectar al resto del sistema.

### Cómo los definen otros

Algunas definiciones adicionales:

* **IA de Google**: Un microfrontend es una arquitectura de desarrollo web que divide una aplicación en módulos independientes. Cada módulo se puede desarrollar, probar e implementar de forma autónoma. [https://www.aplyca.com/blog/microfrontends-que-son-y-cuando-usarlos](https://www.aplyca.com/blog/microfrontends-que-son-y-cuando-usarlos)
* **Wikipedia**: Es un patrón de desarrollo web front-end en el que se puede crear una sola aplicación a partir de compilaciones separadas. Es un enfoque análogo a los microservicios, pero para aplicaciones de una sola página del lado del cliente escritas en JavaScript. [https://es.wikipedia.org/wiki/Microfrontend](https://es.wikipedia.org/wiki/Microfrontend)


## Arquitecturas de microfrontends

La arquitectura de los microfrontends define cómo conviven dentro de una aplicación. Aquí están las tres principales:

### Arquitectura Vertical

{% include img-with-caption.html alt="Arquitectura vertical" img="/assets/images/microfrontends/post-1/vertical.svg" caption="Arquitectura vertical" style="width: 100%; max-width: 480px" class="align-center" %}

Cada sección de la página web es un microfrontend independiente. Todos están activos al mismo tiempo y pueden comunicarse en tiempo real
> **Ejemplo**: Un dashboard con diferentes widgets, donde cada widget es un microfrontend.

### Arquitectura Horizontal

{% include img-with-caption.html alt="Arquitectura horizontal" img="/assets/images/microfrontends/post-1/horizontal.svg" caption="Arquitectura horizontal" style="width: 100%; max-width: 1009px" class="align-center" %}

Cada página completa es un microfrontend independiente. No conviven simultáneamente, lo que hace que la comunicación en tiempo real sea más complicada.
> **Ejemplo**: Un sistema con módulos separados como "Facturación", "Inventario" y "Reportes", donde cada uno es un microfrontend cargado en diferentes rutas.

### Arquitectura Híbrida

{% include img-with-caption.html alt="Arquitectura híbrida" img="/assets/images/microfrontends/post-1/hybrid.svg" caption="Arquitectura híbrida" style="width: 100%; max-width: 1009px" class="align-center" %}

Combinación de las anteriores. Algunos microfrontends (como el menú y el footer) están siempre activos, mientras que otros se cargan según la ruta.
> **Ejemplo**: Una SPA donde el menú de navegación es un microfrontend vertical y los diferentes módulos se cargan como microfrontends horizontales.

## ¿Cuándo usar microfrontends?

Es importante no aplicar microfrontends en todos los proyectos solo porque es una tecnología interesante. Veamos sus ventajas y desventajas para tomar decisiones informadas.

### Ventajas

* **Escalabilidad**: Se pueden agregar nuevos microfrontends fácilmente y cada uno puede desplegarse de forma independiente.
* **Agilidad**: Permite que diferentes equipos trabajen en paralelo sin afectar otras partes de la aplicación.
* **Resiliencia**: Si un microfrontend falla, los demás pueden seguir funcionando.
* **Agnosticismo** tecnológico: Cada microfrontend puede implementarse con diferentes frameworks y librerías.

### Desventajas

* **Complejidad**: Requiere una arquitectura bien planificada para manejar la comunicación y la gestión de estados.
* **Cargas lentas**: Si cada microfrontend se despliega por separado, puede haber latencia en la carga.
* **Coordinación**: Es más difícil alinear equipos cuando cada uno trabaja en su propio microfrontend.

### Casos de uso recomendados

* Empresas con múltiples productos y servicios que necesitan integrarse en una misma plataforma.
* Aplicaciones grandes con módulos independientes que evolucionan a diferentes ritmos.
* Proyectos con equipos distribuidos que trabajan en diferentes partes del frontend.

## ¿Cómo implementar microfrontends?

Existen varias herramientas y enfoques para implementar microfrontends. Aquí algunas opciones:

### Frameworks y librerías

#### Single-Spa

[Single-Spa](https://single-spa.js.org/) es un framework especializado en microfrontends. Permite combinar múltiples aplicaciones construidas con distintas tecnologías (React, Angular, Vue, etc.) en una sola aplicación.
* Usa **import maps** y carga aplicaciones a demanda.
* Permite **montar y desmontar** microfrontends dinámicamente.
* Tiene herramientas avanzadas de desarrollo y debugging.
* Sencillo manejo de dependencias e integración de multiples tecnologías y frameworks

#### Webpack Module Federation

[Module Federation](https://module-federation.io/) es una funcionalidad de Webpack que permite exponer módulos remotos y compartir código entre aplicaciones.
* **No es un framework**, sino una forma de compartir módulos entre aplicaciones.
* Puede integrarse con **cualquier tecnología** (React, Angular, Vue, etc.).
* La gestión de dependencias e integración es **más compleja** y requiere wrappers en algunos casos.

Desde mi punto de vista:
> Module Federation es a single-spa lo que react es a angular

### Otras opciones

* **Qwik**: Ofrece "containers" para microfrontends.
* **Iframes**: Útiles en algunos casos, pero pueden generar problemas de rendimiento.
* **Custom Elements**: Permiten encapsular apps completas dentro de componentes HTML personalizados.

## Conclusión

Después de tres años trabajando con microfrontends, puedo decir que esta arquitectura ofrece un enfoque potente y flexible para el desarrollo de aplicaciones web escalables. Hemos visto cómo diferentes arquitecturas (vertical, horizontal e híbrida) permiten adaptar los microfrontends a distintos escenarios, optimizando la modularidad, la agilidad y la independencia de los equipos.

Sin embargo, no es una solución mágica. La implementación de microfrontends conlleva desafíos como la complejidad en la comunicación entre módulos, la coordinación entre equipos y posibles problemas de rendimiento. Es fundamental evaluar cuidadosamente si esta arquitectura es la más adecuada para cada proyecto y equipo de desarrollo.

En próximos artículos, seguiré profundizando en mi experiencia con microfrontends, explorando herramientas, patrones de implementación y mejores prácticas. ¡Nos leemos pronto!
