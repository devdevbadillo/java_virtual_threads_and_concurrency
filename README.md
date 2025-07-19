# Java Virtual Threads & Concurrency

## Tabla de contenido

- [Profundizando los hilos virtuales](#deep-dive-into-virtual-threads)
  - [Introducción a Virtual Threads](#introduccion-a-virtual-threads)
    - [¿Qué problema resuelven?](#problema-virtual-threads)
    - [Concepto de User-Mode vs. Kernel-Mode Threads](#user-mode-vs-kernel-mode)
    - [Modelo de Carrier Threads (Platform Threads)](#carrier-threads)
      - [Ventajas clave](#ventajas-virtual-threads)
      - [Diferencias con otros modelos de concurrencia](#diferencias-otros-modelos)
  - [Creación y Gestión de Virtual Threads](#creacion-gestion-virtual-threads)
    - [`Thread.ofVirtual()` y `Thread.Builder`](#thread-ofvirtual-thread-builder)
    - [`Executors.newVirtualThreadPerTaskExecutor()`](#executors-newvirtualthreadpertaskexecutor)
    - [Lanzamiento y espera (`join()`)](#lanzamiento-espera)
    - [Estados de las Virtual Threads](#estados-virtual-threads)
    - [Interrupción de Virtual Threads](#interrupcion-virtual-threads)
  - [Virtual Threads y I/O Bloqueante](#virtual-threads-io-bloqueante)
    - [El impacto revolucionario](#impacto-io-bloqueante)
    - [Desmontaje (unmounting) y Montaje (mounting)](#unmounting-mounting)
    - [Limitaciones y escenarios no optimizados](#limitaciones-virtual-threads)
  - [Concurrencia Estructurada (Introducción)](#concurrencia-estructurada-introduccion)
    - [Necesidad](#necesidad-concurrencia-estructurada)
    - [Concepto](#concepto-concurrencia-estructurada)
  - [Herramientas y Debugging con Virtual Threads](#herramientas-debugging-virtual-threads)
    - [Monitorización](#monitorizacion-virtual-threads)
    - [Debugging](#debugging-virtual-threads)
  - [Impacto en APIs existentes](#impacto-apis-existentes)
    - [Interacción con APIs de concurrencia](#interaccion-apis-concurrencia)
    - [`ThreadLocal` y su uso con Virtual Threads](#threadlocal-virtual-threads)

- [Executor Service](#executor-service)
  - [Revisión de Concurrencia Clásica](#revision-concurrencia-clasica)
    - [Problemas del uso directo de `Thread`](#problemas-directo-thread)
    - [Concepto de Thread Pool](#concepto-thread-pool)
  - [`Executor` y `ExecutorService` Interfaces](#executor-executorservice-interfaces)
    - [Propósito y diferenciación](#proposito-diferenciacion-executor)
    - [Métodos clave](#metodos-clave-executor)
  - [Tipos de `ExecutorService` (Fábricas en `Executors`)](#tipos-executorservice)
    - [`newFixedThreadPool()`](#newfixedthreadpool)
    - [`newCachedThreadPool()`](#newcachedthreadpool)
    - [`newSingleThreadExecutor()`](#newsinglethreadexecutor)
    - [`newScheduledThreadPool()`](#newscheduledthreadpool)
    - [`newWorkStealingPool()` (ForkJoinPool)](#newworkstealingpool)
    - [`newVirtualThreadPerTaskExecutor()` (Conexión VT)](#newvirtualthreadpertaskexecutor-connection)
  - [`Callable` y `Future`](#callable-future)
    - [`Runnable` vs. `Callable`](#runnable-vs-callable)
    - [`Future` y sus métodos](#future-y-sus-metodos)
  - [Gestión del Ciclo de Vida del ExecutorService](#gestion-ciclo-vida-executorservice)
    - [`shutdown()` vs. `shutdownNow()`](#shutdown-vs-shutdownnow)
    - [`awaitTermination()`](#awaittermination)
  - [Manejo de Excepciones en Tareas Asíncronas](#manejo-excepciones-tareas-asincronas)
    - [Excepciones en `Runnable` y `Callable`](#excepciones-runnable-callable)
    - [`UncaughtExceptionHandler`](#uncaughtexceptionhandler)

- [CompletableFuture](#completablefuture)
  - [Limitaciones de `Future`](#limitaciones-future)
  - [Concepto de `CompletableFuture`](#concepto-completablefuture)
    - [Programación Asíncrona No Bloqueante](#programacion-asincrona-no-bloqueante)
    - [Composición de Tareas Asíncronas](#composicion-tareas-asincronas)
    - [Manejo de Errores Asíncronos](#manejo-errores-asincronos)
  - [Creación de `CompletableFuture`](#creacion-completablefuture)
    - [`runAsync()`](#runasync)
    - [`supplyAsync()`](#supplyasync)
    - [`completedFuture()`](#completedfuture)
  - [Encadenamiento y Composición](#encadenamiento-composicion-completablefuture)
    - [Transformación (`thenApply`, `thenApplyAsync`)](#transformacion-completablefuture)
    - [Consumo (`thenAccept`, `thenRun`)](#consumo-completablefuture)
    - [Combinación de Futuros (`thenCompose`, `thenCombine`, `allOf`, `anyOf`)](#combinacion-futuros)
  - [Manejo de Excepciones](#manejo-excepciones-completablefuture)
    - [`exceptionally()`](#exceptionally)
    - [`handle()`](#handle)
    - [`whenComplete()`](#whencomplete)
  - [Control del Hilo de Ejecución (`Async` métodos)](#control-hilo-ejecucion)
  - [Programación Funcional y `CompletableFuture`](#programacion-funcional-completablefuture)

- [Scoped Values & Structured Concurrency](#scoped-values-structured-concurrency)
  - [Scoped Values](#scoped-values)
    - [Problemas de `ThreadLocal` con Virtual Threads](#problemas-threadlocal-virtual-threads)
    - [Concepto de Scoped Values](#concepto-scoped-values)
    - [Uso (`where()`, `bind()`, `get()`)](#uso-scoped-values)
    - [Ventajas](#ventajas-scoped-values)
    - [Casos de Uso](#casos-uso-scoped-values)
  - [Structured Concurrency](#structured-concurrency)
    - [Problema de la concurrencia "go-to"](#problema-concurrencia-go-to)
    - [Concepto de Concurrencia Estructurada](#concepto-structured-concurrency)
    - [`StructuredTaskScope` API](#structuredtaskscope-api)
      - [`fork()`](#fork-structuredtaskscope)
      - [`join()`](#join-structuredtaskscope)
      - [Manejo de errores y cancelación](#manejo-errores-cancelacion-structuredtaskscope)
    - [Estrategias de `StructuredTaskScope`](#estrategias-structuredtaskscope)
      - [`ShutdownOnFailure`](#shutdownonfailure)
      - [`ShutdownOnSuccess`](#shutdownonsuccess)
    - [Ventajas](#ventajas-structured-concurrency)
    - [Relación con Virtual Threads](#relacion-virtual-threads-structured-concurrency)

- [Desarrollo de aplicaciones con Spring & hilos virtuales](#application-development-spring-virtual-threads)
  - [Configuración de Spring Boot para Virtual Threads](#configuracion-spring-boot-virtual-threads)
    - [Propiedades de configuración](#propiedades-configuracion-spring-boot)
    - [Configuración de `TaskExecutor`](#configuracion-taskexecutor)
  - [Impacto de Virtual Threads en Spring Web](#impacto-virtual-threads-spring-web)
    - [Servlets y Tomcat/Jetty](#servlets-tomcat-jetty)
    - [Simplificación de código bloqueante](#simplificacion-codigo-bloqueante)
    - [Integración con APIs de persistencia](#integracion-apis-persistencia)
  - [Integración con otros módulos de Spring](#integracion-otros-modulos-spring)
  - [Diseño de APIs y Servicios](#diseno-apis-servicios)
    - [Reactive Programming vs. Virtual Threads](#reactive-vs-virtual-threads)
  - [Pruebas de Rendimiento y Escalamiento](#pruebas-rendimiento-escalamiento)

- [Pruebas de rendimiento con JMeter](#performance-testing-jmeter)
  - [Fundamentos de Performance Testing](#fundamentos-performance-testing)
    - [¿Por qué es importante?](#importancia-performance-testing)
    - [Tipos de Pruebas de Rendimiento](#tipos-pruebas-rendimiento)
    - [Métricas clave](#metricas-clave-performance)
  - [Introducción a Apache JMeter](#introduccion-jmeter)
    - [Arquitectura y Conceptos](#arquitectura-conceptos-jmeter)
    - [Instalación y Configuración](#instalacion-configuracion-jmeter)
  - [Creación de un Plan de Pruebas Básico](#creacion-plan-pruebas-jmeter)
    - [Thread Group](#thread-group-jmeter)
    - [HTTP Request Sampler](#http-request-sampler-jmeter)
    - [Listeners](#listeners-jmeter)
  - [Configuración Avanzada de JMeter](#configuracion-avanzada-jmeter)
    - [Config Elements](#config-elements-jmeter)
    - [Assertions](#assertions-jmeter)
    - [Timers](#timers-jmeter)
    - [Controllers](#controllers-jmeter)
  - [Diseño de Escenarios de Pruebas](#diseno-escenarios-pruebas-jmeter)
  - [Análisis de Resultados de JMeter](#analisis-resultados-jmeter)



<a id="deep-dive-into-virtual-threads"></a>
## Profundizando los hilos virtuales

<a id="introduccion-a-virtual-threads"></a>
### Introducción a Virtual Threads

<a id="problema-virtual-threads"></a>
#### ¿Qué problema resuelven?

<a id="user-mode-vs-kernel-mode"></a>
#### Concepto de User-Mode vs. Kernel-Mode Threads

<a id="carrier-threads"></a>
#### Modelo de Carrier Threads (Platform Threads)

<a id="ventajas-virtual-threads"></a>
#### Ventajas clave

<a id="diferencias-otros-modelos"></a>
#### Diferencias con otros modelos de concurrencia

<a id="creacion-gestion-virtual-threads"></a>
### Creación y Gestión de Virtual Threads

<a id="thread-ofvirtual-thread-builder"></a>
#### `Thread.ofVirtual()` y `Thread.Builder`

<a id="executors-newvirtualthreadpertaskexecutor"></a>
#### `Executors.newVirtualThreadPerTaskExecutor()`

<a id="lanzamiento-espera"></a>
#### Lanzamiento y espera (`join()`)

<a id="estados-virtual-threads"></a>
#### Estados de las Virtual Threads

<a id="interrupcion-virtual-threads"></a>
#### Interrupción de Virtual Threads

<a id="virtual-threads-io-bloqueante"></a>
### Virtual Threads y I/O Bloqueante

<a id="impacto-io-bloqueante"></a>
#### El impacto revolucionario

<a id="unmounting-mounting"></a>
#### Desmontaje (unmounting) y Montaje (mounting)

<a id="limitaciones-virtual-threads"></a>
#### Limitaciones y escenarios no optimizados

<a id="concurrencia-estructurada-introduccion"></a>
### Concurrencia Estructurada (Introducción)

<a id="necesidad-concurrencia-estructurada"></a>
#### Necesidad

<a id="concepto-concurrencia-estructurada"></a>
#### Concepto

<a id="herramientas-debugging-virtual-threads"></a>
### Herramientas y Debugging con Virtual Threads

<a id="monitorizacion-virtual-threads"></a>
#### Monitorización

<a id="debugging-virtual-threads"></a>
#### Debugging

<a id="impacto-apis-existentes"></a>
### Impacto en APIs existentes

<a id="interaccion-apis-concurrencia"></a>
#### Interacción con APIs de concurrencia

<a id="threadlocal-virtual-threads"></a>
#### `ThreadLocal` y su uso con Virtual Threads

<a id="executor-service"></a>
## Executor Service

<a id="revision-concurrencia-clasica"></a>
### Revisión de Concurrencia Clásica

<a id="problemas-directo-thread"></a>
#### Problemas del uso directo de `Thread`

<a id="concepto-thread-pool"></a>
#### Concepto de Thread Pool

<a id="executor-executorservice-interfaces"></a>
### `Executor` y `ExecutorService` Interfaces

<a id="proposito-diferenciacion-executor"></a>
#### Propósito y diferenciación

<a id="metodos-clave-executor"></a>
#### Métodos clave

<a id="tipos-executorservice"></a>
### Tipos de `ExecutorService` (Fábricas en `Executors`)

<a id="newfixedthreadpool"></a>
#### `newFixedThreadPool()`

<a id="newcachedthreadpool"></a>
#### `newCachedThreadPool()`

<a id="newsinglethreadexecutor"></a>
#### `newSingleThreadExecutor()`

<a id="newscheduledthreadpool"></a>
#### `newScheduledThreadPool()`

<a id="newworkstealingpool"></a>
#### `newWorkStealingPool()` (ForkJoinPool)

<a id="newvirtualthreadpertaskexecutor-connection"></a>
#### `newVirtualThreadPerTaskExecutor()` (Conexión VT)

<a id="callable-future"></a>
### `Callable` y `Future`

<a id="runnable-vs-callable"></a>
#### `Runnable` vs. `Callable`

<a id="future-y-sus-metodos"></a>
#### `Future` y sus métodos

<a id="gestion-ciclo-vida-executorservice"> </a>
### Gestión del Ciclo de Vida del ExecutorService

<a id="shutdown-vs-shutdownnow"> </a>
#### `shutdown()` vs. `shutdownNow()`

<a id="awaittermination"> </a>
#### `awaitTermination()`

<a id="manejo-excepciones-tareas-asincronas"> </a>
### Manejo de Excepciones en Tareas Asíncronas

<a id="excepciones-runnable-callable"> </a>
#### Excepciones en `Runnable` y `Callable`

<a id="uncaughtexceptionhandler"> </a>
#### `UncaughtExceptionHandler`

<a id="completablefuture"> </a>
## CompletableFuture

<a id="limitaciones-future"> </a>
### Limitaciones de `Future`

<a id="concepto-completablefuture"> </a>
### Concepto de `CompletableFuture`
  
<a id="programacion-asincrona-no-bloqueante"> </a>
#### Programación Asíncrona No Bloqueante

<a id="composicion-tareas-asincronas"> </a>
#### Composición de Tareas Asíncronas

<a id="manejo-errores-asincronos"> </a>
#### Manejo de Errores Asíncronos

<a id="creacion-completablefuture"> </a>
### Creación de `CompletableFuture`

<a id="runasync"> </a>
#### `runAsync()`

<a id="supplyasync"> </a>
#### `supplyAsync()`

<a id="completedfuture"> </a>
#### `completedFuture()`

<a id="encadenamiento-composicion-completablefuture"> </a>
### Encadenamiento y Composición

<a id="transformacion-completablefuture"> </a>
#### Transformación (`thenApply`, `thenApplyAsync`)

<a id="consumo-completablefuture"> </a>
#### Consumo (`thenAccept`, `thenRun`)

<a id="combinacion-futuros"> </a>
#### Combinación de Futuros (`thenCompose`, `thenCombine`, `allOf`, `anyOf`)

<a id="manejo-excepciones-completablefuture"> </a>
### Manejo de Excepciones

<a id="exceptionally"> </a>
#### `exceptionally()`

<a id="handle"> </a>
#### `handle()`

<a id="whencomplete"> </a>
#### `whenComplete()`

<a id="control-hilo-ejecucion"> </a>
### Control del Hilo de Ejecución (`Async` métodos)

<a id="programacion-funcional-completablefuture"> </a>
### Programación Funcional y `CompletableFuture`

<a id="scoped-values-structured-concurrency"> </a>
## Scoped Values & Structured Concurrency

<a id="scoped-values"> </a>
### Scoped Values

<a id="problemas-threadlocal-virtual-threads"> </a>
#### Problemas de `ThreadLocal` con Virtual Threads

<a id="concepto-scoped-values"> </a>
#### Concepto de Scoped Values

<a id="uso-scoped-values"> </a>
#### Uso (`where()`, `bind()`, `get()`)

<a id="ventajas-scoped-values"> </a>
#### Ventajas

<a id="casos-uso-scoped-values"> </a>
#### Casos de Uso

<a id="structured-concurrency"> </a>
### Structured Concurrency

<a id="problema-concurrencia-go-to"> </a>
#### Problema de la concurrencia "go-to"

<a id="concepto-structured-concurrency"> </a>
#### Concepto de Concurrencia Estructurada

<a id="structuredtaskscope-api"> </a>
#### `StructuredTaskScope` API

<a id="fork-structuredtaskscope"> </a>
##### `fork()`

<a id="join-structuredtaskscope"> </a>
##### `join()`
  
<a id="manejo-errores-cancelacion-structuredtaskscope"> </a>
##### Manejo de errores y cancelación

<a id="estrategias-structuredtaskscope"> </a>
#### Estrategias de `StructuredTaskScope`

<a id="shutdownonfailure"></a>
##### `ShutdownOnFailure`

<a id="shutdownonsuccess"></a>
##### `ShutdownOnSuccess`

<a id="ventajas-structured-concurrency"></a>
#### Ventajas

<a id="relacion-virtual-threads-structured-concurrency"></a>
#### Relación con Virtual Threads

<a id="application-development-spring-virtual-threads"></a>
## Application Development With Spring & Virtual Threads

<a id="configuracion-spring-boot-virtual-threads"></a>
### Configuración de Spring Boot para Virtual Threads

<a id="propiedades-configuracion-spring-boot"></a>
#### Propiedades de configuración

<a id="configuracion-taskexecutor"></a>
#### Configuración de `TaskExecutor`

<a id="impacto-virtual-threads-spring-web"></a>
### Impacto de Virtual Threads en Spring Web

<a id="servlets-tomcat-jetty"></a>
#### Servlets y Tomcat/Jetty

<a id="simplificacion-codigo-bloqueante"></a>
#### Simplificación de código bloqueante

<a id="integracion-apis-persistencia"></a>
#### Integración con APIs de persistencia

<a id="integracion-otros-modulos-spring"></a>
### Integración con otros módulos de Spring

<a id="diseno-apis-servicios"></a>
### Diseño de APIs y Servicios

<a id="reactive-vs-virtual-threads"></a>
#### Reactive Programming vs. Virtual Threads

<a id="pruebas-rendimiento-escalamiento"></a>
### Pruebas de Rendimiento y Escalamiento

<a id="performance-testing-jmeter"></a>
## Pruebas de rendimiento con JMeter

<a id="fundamentos-performance-testing"></a>
### Fundamentos de Performance Testing

<a id="importancia-performance-testing"></a>
#### ¿Por qué es importante?

<a id="tipos-pruebas-rendimiento"></a>
#### Tipos de Pruebas de Rendimiento

<a id="metricas-clave-performance"></a>
#### Métricas clave

<a id="introduccion-jmeter"></a>
### Introducción a Apache JMeter

<a id="arquitectura-conceptos-jmeter"></a>
#### Arquitectura y Conceptos

<a id="instalacion-configuracion-jmeter"></a>
#### Instalación y Configuración

<a id="creacion-plan-pruebas-jmeter"></a>
### Creación de un Plan de Pruebas Básico

<a id="thread-group-jmeter"></a>
#### Thread Group

<a id="http-request-sampler-jmeter"></a>
#### HTTP Request Sampler

<a id="listeners-jmeter"></a>
#### Listeners

<a id="configuracion-avanzada-jmeter"></a>
### Configuración Avanzada de JMeter

<a id="config-elements-jmeter"></a>
#### Config Elements

<a id="assertions-jmeter"></a>
#### Assertions

<a id="timers-jmeter"></a>
#### Timers

<a id="controllers-jmeter"></a>
#### Controllers

<a id="diseno-escenarios-pruebas-jmeter"></a>
### Diseño de Escenarios de Pruebas

<a id="analisis-resultados-jmeter"></a>
### Análisis de Resultados de JMeter
