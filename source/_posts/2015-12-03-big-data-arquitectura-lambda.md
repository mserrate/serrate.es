---
title: 'Big Data: arquitectura lambda'
date: 2015-12-03T23:51:16+00:00
categories:
  - Big Data
tags:
  - Apache Storm
  - Big Data
  - Hadoop
---
Actualmente llevo varios años trabajando con sistemas distribuidos y arquitecturas orientadas a eventos, desde el malentendido <a href="http://www.serrate.es/tag/soa/" target="_blank">SOA</a> (o su versión refurbished como son los <a href="http://udidahan.com/2014/03/31/on-that-microservices-thing/" target="_blank">Microservices</a>), hasta <a href="http://www.serrate.es/2015/02/18/sesion-en-la-dotnetspain-conference/" target="_blank">Event Sourcing</a> (indirectamente enlazado con <a href="http://www.serrate.es/category/arquitectura/cqrs/" target="_blank">CQRS</a>).

Muchos de los conceptos vinculados con eventos como **inmutabilidad**, **perpetuidad**, **idempotencia**… son igualmente validos para realizar _stream processing_ o flujo continuo de eventos. Si a esto le sumamos el proceso por lotes o _batch processing_, entramos en el terreno de lo que muchos llaman Big Data.

### Big Data

Cuando pensamos en big data lo primero que nos viene a la cabeza es **Hadoop** para ejecutar procesos por lotes. Tenemos que pensar que Hadoop ofrece una gran capacidad para procesar una cantidad indecente de datos, pero a costa de una alta latencia en la respuesta. Aunque esta latencia puede ser de sobras aceptable para muchos casos de uso, no lo es para otros que tienen la necesidad de procesar una gran cantidad de datos en tiempo (casi) real.

Ahí es donde entra en juego lo que muchos llaman <a href="http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html" target="_blank">lambda architecture</a> (Nathan Marz, ex-twitter).

<!--more-->Lo que nos ofrece esta arquitectura es el concepto de procesar la mayoría de nuestros datos en lotes (batch) y que, mientras dura este proceso, podamos procesar los datos que continúan entrando (stream) en el sistema:

{% img /uploads/2015/12/Lambda-3.png 483 223 '"Arquitectura lambda"' '"Arquitectura lambda"' %}

Podemos decir que:

> Situación Actual = Consulta(Batch Process) + Consulta(Stream Process)

#### Batch Processing

Como su nombre indica, vamos a procesar eventos pero de forma agrupada. Normalmente hablamos aquí de **Hadoop** (o para ser mas exactos el sistema de ficheros **HDFS** y una herramienta para procesar como **MapReduce**, **Pig** …)

El resultado del proceso se persistirá en una base de datos que soporte escrituras en lotes (ElephantDB, HBase), lo bueno de ésta parte es que la base de datos no tiene que soportar escrituras aleatorias: una de las principales causas de complejidad de las bases de datos por tener que ofrecer compactación en línea y concurrencia.

#### Stream Processing

El objetivo es procesar uno por uno los eventos que recibimos: dependiendo del numero de eventos y del rendimiento que necesitemos vamos a usar una tecnología u otra: **Spark Streaming** (aunque sea micro-batching, el rendimiento es suficiente para la mayoría de casos), **Storm**, **Samza**, **Flink**.

El resultado del proceso se persistirá en una base de datos que sí tiene que soportar escrituras aleatorias por lo que una de las opciones puede ser **Cassandra**.

En siguientes publicaciones voy a presentar ejemplos concretos con las imágenes _docker_ de soporte con algunas de las tecnologías que utilizo como **Kafka**, **Storm**, **Cassandra** y **Druid**.