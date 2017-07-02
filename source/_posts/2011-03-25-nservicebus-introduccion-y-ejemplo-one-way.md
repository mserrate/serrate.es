---
id: 235
title: 'NServiceBus: Introducción y ejemplo One-Way'
date: 2011-03-25T13:06:13+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=235
permalink: /2011/03/25/nservicebus-introduccion-y-ejemplo-one-way/
categories:
  - SOA
tags:
  - NServiceBus
  - SOA
---
[](http://www.serrate.es/wp-content/uploads/2011/03/consola.png)A raíz de algunos proyectos e inmersión en los mundos de <a href="http://en.wikipedia.org/wiki/Service-oriented_architecture" target="_blank">Service Oriented Architecture </a>(SOA) y <a href="http://en.wikipedia.org/wiki/Event-driven_architecture" target="_blank">Event-Driven Architecture</a> (EDA) he conocido este producto creado por <a href="http://www.udidahan.com/" target="_blank">Udi Dahan </a>y que podemos encontrar en: <a href="http://www.nservicebus.com/" target="_blank">http://www.nservicebus.com/</a>

**NServiceBus** es una implementación de Enterprise Service Bus (ESB) para .NET y que se apoya en Microsoft Message Queue Server (MSMQ) como transporte y persistencia de colas.

Enterprise Service Bus es una arquitectura que nos permite el intercambio de mensajes entre aplicaciones y servicios de manera desacoplada entre los mismos proporcionando la fiabilidad y manejo de errores que éste tipo de aplicaciones requiere.

Las diferencias entre Enterprise Service Bus y Enterprise Application Integration (EAI) como BizTalk y cuando usar uno u otro daría lugar a muchas discusiones y está fuera del alcance de éste humilde artículo.

NServiceBus proporciona varios patrones de mensajería OOB como **One-Way**, **Request/Response** y **Publish/Subscribe** aunque nos permite extenderlo según nuestras necesidades.

## Ejemplo con mensajería One-Way

En este ejemplo veremos la comunicación asíncrona punto a punto. Aunque parezca a priori que podríamos conseguir los mismos resultados con netMsmqBinding y WCF, y en cierta manera así es, la verdad es que NServiceBus nos abstrae de muchas configuraciones que deberíamos realizar a mano: creación de las colas en MSMQ, habilitar permisos necesarios, etc.

Primero vamos a crear el proyecto **Messages** (Class library) con una referencia a la librería NServiceBus.dll. A continuación definiremos nuestro mensaje:

<pre class="brush: csharp; title: ; notranslate" title="">namespace Messages
{
    /// &lt;summary&gt;
    /// Esta clase será enviada por NServiceBus
    /// &lt;/summary&gt;
    public class MyMessage : IMessage
    {
        public string SomeString { get; set; }
    }
}
</pre>

La interface **IMessage** no obliga a implementar ningún método, simplemente se usa como marcador para NServiceBus.

<!--more-->

A continuación crearemos el proyecto **Client** (Class library) con referencias a NServiceBus.dll, NServiceBus.Core.dll, NServiceBus.Host.exe, log4net.dll y a nuestro proyecto **Messages**.

En éste proyecto crearemos una clase de nombre **EndpointConfig**:

<pre class="brush: csharp; title: ; notranslate" title="">namespace Client
{
    /// &lt;summary&gt;
    /// Configuramos el endpoint como cliente
    /// &lt;/summary&gt;
    public class EndpointConfig : IConfigureThisEndpoint, AsA_Client
    {
    }
}
</pre>

Tenemos dos interfaces más que actúan como marcadores:

  * **IConfigureThisEndpoint** El proceso NServiceBus.Host.exe buscará en el directorio de ejecución la clase que implementa ésta interface para saber qué tipo de endpoint es.
  * **AsA_Client** indica al mismo proceso que el endpoint será de tipo cliente.

Creamos ahora una clase llamada **ClientEndpoint**:

<pre class="brush: csharp; title: ; notranslate" title="">namespace Client
{
    /// &lt;summary&gt;
    /// Clase que se ejecuta al inicio
    /// &lt;/summary&gt;
    public class ClientEndpoint : IWantToRunAtStartup
    {
        /// &lt;summary&gt;
        /// Inyectado por defecto mediante Spring
        /// &lt;/summary&gt;
        public IBus Bus { get; set; }

        public void Run()
        {
            Console.WriteLine("--- Escribe \"exit\" para salir de la aplicación. ---");
            Console.WriteLine("--- Pulsa ENTER para enviar el mensaje. ---");

            var mensaje = Console.ReadLine();

            while (mensaje != "exit")
            {
                // enviamos el mensaje
                Bus.Send&lt;MyMessage&gt;(m =&gt; { m.SomeString = mensaje; });

                mensaje = Console.ReadLine();
            }
        }

        public void Stop()
        {
        }
    }
}
</pre>

Con la interfaz **IWantToRunAtStartup** indicamos que la clase se ejecuta al inicio, y en el método Run() realizamos el envío del mensaje mediante Bus.Send<IMessage>(Action<IMessage>)

Podemos ver que la propiedad IBus se inyecta mediante el contenedor Spring que utiliza por defecto NServiceBus, aunque también podemos configurar y usar el framework que más nos guste (Unity, Castle, etc).

Finalmente, creamos en nuestro fichero **App.config** las configuraciones siguientes:

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;configuration&gt;
  &lt;configSections&gt;
    &lt;section name="MsmqTransportConfig" type="NServiceBus.Config.MsmqTransportConfig, NServiceBus.Core" /&gt;
    &lt;section name="UnicastBusConfig" type="NServiceBus.Config.UnicastBusConfig, NServiceBus.Core" /&gt;
  &lt;/configSections&gt;

  &lt;MsmqTransportConfig
    InputQueue="SerrateSendingQueue"
    ErrorQueue="SerrateError"
    NumberOfWorkerThreads="1"
    MaxRetries="5"
  /&gt;

  &lt;UnicastBusConfig&gt;
    &lt;MessageEndpointMappings&gt;
      &lt;add Messages="Messages" Endpoint="SerrateReceivingQueue" /&gt;
    &lt;/MessageEndpointMappings&gt;
  &lt;/UnicastBusConfig&gt;

&lt;/configuration&gt;
</pre>

En las **configSections** definimos las dos secciones que utilizamos más abajo.

En **MsmqTransportConfig** configuramos las colas de entrada y error respectivamente. También configuramos el número máximo de reintentos cuando exista cualquier excepción en el envío antes de enviar el mensaje a la cola de error.

En **UnicastBusConfig** configuramos el ensamblado donde se encuentra las clases que implementan IMessage y el endpoint del servidor.

Para la parte de servidor, creamos el proyecto **Server** (Class library) con referencias a NServiceBus.dll, NServiceBus.Core.dll, NServiceBus.Host.exe, log4net.dll y a nuestro proyecto **Messages**.

De la misma manera, crearemos una clase de nombre **EndpointConfig**:

<pre class="brush: csharp; title: ; notranslate" title="">namespace Server
{
    /// &lt;summary&gt;
    /// Configuramos el endpoint como servidor
    /// &lt;/summary&gt;
    public class EndpointConfig : IConfigureThisEndpoint, AsA_Server
    {
    }
}
</pre>

Esta vez, utilizamos la interface **AsA_Server** para indicar que el endpoint es de tipo servidor.

Posteriormente, creamos una clase **MyMessageHandler** para procesar los mensajes recibidos:

<pre class="brush: csharp; title: ; notranslate" title="">namespace Server
{
    /// &lt;summary&gt;
    /// Procesa los mensajes recibidos
    /// &lt;/summary&gt;
    public class MyMessageHandler : IHandleMessages&lt;MyMessage&gt;
    {
        public void Handle(MyMessage message)
        {
            if (message.SomeString == "error")
            {
                throw new Exception("error grave");
            }

            Console.WriteLine(string.Format("Se ha recibido el siguiente mensaje: {0}", message.SomeString));
        }
    }
}
</pre>

Finalmente, la configuración será parecida a la parte cliente exceptuando la parte de UnicastBusConfig:

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;configuration&gt;
  &lt;configSections&gt;
    &lt;section name="MsmqTransportConfig" type="NServiceBus.Config.MsmqTransportConfig, NServiceBus.Core" /&gt;
    &lt;section name="UnicastBusConfig" type="NServiceBus.Config.UnicastBusConfig, NServiceBus.Core" /&gt;
  &lt;/configSections&gt;

  &lt;MsmqTransportConfig
    InputQueue="SerrateReceivingQueue"
    ErrorQueue="SerrateError"
    NumberOfWorkerThreads="1"
    MaxRetries="5"
  /&gt;

  &lt;UnicastBusConfig&gt;
    &lt;MessageEndpointMappings&gt;
    &lt;/MessageEndpointMappings&gt;
  &lt;/UnicastBusConfig&gt;
&lt;/configuration&gt;
</pre>

## Ejecución del ejemplo

Como los proyectos Client y Server son de tipo class library para ejecutarlos deberemos seleccionar la pestaña Debug de la configuración del proyecto y en “Start External Program” seleccionar NServiceBus.Host.exe.

[<img class="aligncenter size-full wp-image-245" title="projectConfig" src="http://www.serrate.es/wp-content/uploads/2011/03/projectConfig.png" alt="" width="547" height="129" />](http://www.serrate.es/wp-content/uploads/2011/03/projectConfig.png)

A continuación ejecutamos el servidor y el cliente y al enviar un mensaje veremos como el servidor lo recibe sin problema:

[<img class="aligncenter size-full wp-image-243" title="consola" src="http://www.serrate.es/wp-content/uploads/2011/03/consola.png" alt="" width="502" height="401" />](http://www.serrate.es/wp-content/uploads/2011/03/consola.png)

Posteriormente podemos hacer la prueba de parar el servidor y al enviar un mensaje, ésta quedará encolada en MSMQ de la siguiente forma:

[<img class="aligncenter size-full wp-image-244" title="no available" src="http://www.serrate.es/wp-content/uploads/2011/03/no-available.png" alt="" width="363" height="84" srcset="http://www.serrate.es/wp-content/uploads/2011/03/no-available.png 363w, http://www.serrate.es/wp-content/uploads/2011/03/no-available-300x69.png 300w" sizes="(max-width: 363px) 100vw, 363px" />](http://www.serrate.es/wp-content/uploads/2011/03/no-available.png)

Posteriormente, si abrimos el servidor, el mensaje será entregado sin problema alguno.

Si en el servidor se produce una excepción, después de los 5 intentos establecidos, el mensaje pasará a la cola de error. Ahí se deberá de hacer la acción requerida para establecer el servicio y desencolar el mensaje de la cola de error.