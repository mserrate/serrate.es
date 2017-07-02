---
id: 392
title: 'NServiceBus 3.0 &#8211; Distributor'
date: 2012-01-30T22:15:44+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=392
permalink: /2012/01/30/nservicebus-3-0-distributor/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
dsq_thread_id:
  - "573367122"
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

[<img class="aligncenter size-full wp-image-396" title="Distributor1" src="http://www.serrate.es/wp-content/uploads/2012/01/distributor1.png" alt="Distributor 1" width="547" height="352" srcset="http://www.serrate.es/wp-content/uploads/2012/01/distributor1.png 547w, http://www.serrate.es/wp-content/uploads/2012/01/distributor1-300x193.png 300w" sizes="(max-width: 547px) 100vw, 547px" />](http://www.serrate.es/wp-content/uploads/2012/01/distributor1.png)

Cuando se reciben mensajes, el nodo maestro reparte en los nodos disponibles existentes:

[<img class="aligncenter size-full wp-image-400" title="Distributor2" src="http://www.serrate.es/wp-content/uploads/2012/01/distributor2.png" alt="Distributor 2" width="547" height="377" srcset="http://www.serrate.es/wp-content/uploads/2012/01/distributor2.png 547w, http://www.serrate.es/wp-content/uploads/2012/01/distributor2-300x206.png 300w" sizes="(max-width: 547px) 100vw, 547px" />](http://www.serrate.es/wp-content/uploads/2012/01/distributor2.png)

Gracias al distributor podemos ofrecer de forma muy sencilla alta disponibilidad en los nodos de nuestra aplicación.