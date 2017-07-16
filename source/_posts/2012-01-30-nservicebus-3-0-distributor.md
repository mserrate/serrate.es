---
title: 'NServiceBus 3.0 - Distributor'
date: 2012-01-30T22:15:44+00:00
categories:
  - SOA
tags:
  - NServiceBus
  - SOA
---
En las últimas entradas estamos repasando las novedades de NServiceBus 3.0 que ya está en la <a href="https://github.com/downloads/NServiceBus/NServiceBus/NServiceBus.3.0.0-rc2.zip" target="_blank">Release Candidate 2</a>. En esta entrada veremos una funcionalidad que ya existía en versiones anteriores pero que se ha rehecho completamente.

Distributor nos permite balancear la carga entre varios nodos a partir de un nodo maestro. La configuración del nodo maestro es muy sencilla a través de los profiles de NServiceBus, simplemente debemos añadir el argumento **NServiceBus.Master** en el Host. Para asegurar la alta disponibilidad del nodo maestro es recomendable que se ejecute en un cluster.

Para configurar los nodos worker, añadiremos a su vez el argumento **NServiceBus.Worker** para establecer este comportamiento.

El funcionamiento o flujo de trabajo es el siguiente:

  1. Los nodos worker envían un mensaje a **[endpoint].distributor.control** cada vez que están libres.
  2. El nodo Master procesa la cola guardando todos los nodos workers disponibles en la cola **[endpoint].distributor.storage**.
  3. Cuando el nodo Master recibe un mensaje utiliza el primer worker de la lista de **[endpoint].distributor.storage** para enviarle el mensaje y que lo procese.<!--more-->

Imaginemos el siguiente escenario, tenemos configurado un nodo como maestro y tres nodos como workers uno de los cuales no está disponible, los dos envían un mensaje al nodo maestro diciendo que están disponibles para procesar mensajes:

{% img /uploads/2012/01/distributor1.png 547 352 '"Distributor 1"' '"Distributor 1"' %}

Cuando se reciben mensajes, el nodo maestro reparte en los nodos disponibles existentes:

{% img /uploads/2012/01/distributor2.png 547 377 '"Distributor 2"' '"Distributor 2"' %}

Gracias al distributor podemos ofrecer de forma muy sencilla alta disponibilidad en los nodos de nuestra aplicación.