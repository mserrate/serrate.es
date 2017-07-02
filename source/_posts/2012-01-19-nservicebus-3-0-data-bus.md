---
id: 368
title: 'NServiceBus 3.0 &#8211; Data Bus'
date: 2012-01-19T18:02:23+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=368
permalink: /2012/01/19/nservicebus-3-0-data-bus/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
categories:
  - SOA
tags:
  - NServiceBus
  - SOA
---
Siempre que usemos algún sistema de mensajería tenemos que adaptarnos al límite de tamaño del mensaje que el sistema es capaz de enviar. Este límite varía dependiendo de la tecnología usada.

En la mayoría de sistemas _on-premise_ no hay mucho problema ya que el tamaño acostumbra a ser suficiente para la mayoría de casos. Por ejemplo, en MSMQ &#8211; el transporte por defecto en NServiceBus – el límite es de **4MB**.

En sistemas en la nube este límite es notablemente inferior. Algunos ejemplos:

  * Azure Queues: 8KB
  * Azure Service Bus Queues: 64KB
  * Amazon SQS: 64KB

En aquellos casos en los que se requiera un límite mayor ya sea porqué usamos **NServiceBus en Azure** o bien porqué queremos enviar algún dato de gran tamaño como una imagen o video, podemos hacer uso de la propiedad **DataBusProperty<T>**.<!--more-->

Al usar esta propiedad haremos saber a NServiceBus que en el momento de enviar el mensaje éste sea interceptado por un <a title="NServiceBus 3.0 – Message Mutators" href="http://www.serrate.es/2012/01/11/nservicebus-3-0-message-mutators/" target="_blank"><strong>MessageMutator</strong> – ver en el post anterior –</a> y la propiedad sea serializada y persistida en una carpeta compartida para que sea deserializada en el momento de ser obtenida por el servidor. Una imagen vale más que mil palabras:

[<img class="aligncenter size-full wp-image-379" title="WF DataBus" src="http://www.serrate.es/wp-content/uploads/2012/01/DataBus_min.png" alt="WF DataBus" width="561" height="135" srcset="http://www.serrate.es/wp-content/uploads/2012/01/DataBus_min.png 561w, http://www.serrate.es/wp-content/uploads/2012/01/DataBus_min-300x72.png 300w" sizes="(max-width: 561px) 100vw, 561px" />](http://www.serrate.es/wp-content/uploads/2012/01/DataBus_min.png)

Hacer uso del DataBus es muy simple, básicamente tenemos un mensaje con una propiedad de tipo **DataBusProperty<Image>**:

<pre class="brush: csharp; title: ; notranslate" title="">public class MessageWithImage : IMessage
{
    public string MyProperty { get; set; }
    public DataBusProperty Image { get; set; }
}
</pre>

Posteriormente definimos tanto en el cliente como en el servidor para que NServiceBus use la configuración **FileShareDataBus**:

<pre class="brush: csharp; title: ; notranslate" title="">public class Initialization : IWantCustomInitialization
{
    public void Init()
    {
        string basePath = ConfigurationManager.AppSettings["SharedFolder"];

        Configure.Instance
            .FileShareDataBus(basePath)
            .UnicastBus().DoNotAutoSubscribe();
    }
}
</pre>

Enviamos el mensaje _et voilà_:

<pre class="brush: csharp; title: ; notranslate" title="">public class Main : IWantToRunAtStartup
{
    public IBus Bus { get; set; }

    public void Run()
    {
        var img = Image.FromFile("..\\..\\img\\Tulips.png");

        Bus.Send(m =&gt;
        {
            m.MyProperty = "Message with image file ";
            m.Image = new DataBusProperty(img);
        });
    }

    public void Stop()
    {
    }
}
</pre>

Tenemos el mensaje en la cola con la referencia al fichero de la carpeta compartida:

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version="1.0" ?&gt;
&lt;Messages xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://tempuri.net/Messages"&gt;
&lt;MessageWithImage&gt;
&lt;MyProperty&gt;Message with image file &lt;/MyProperty&gt;
&lt;Image&gt;
&lt;Key&gt;2012-01-19_04\cf611fb6-8e31-4543-9db4-08bf51869710&lt;/Key&gt;
&lt;HasValue&gt;true&lt;/HasValue&gt;
&lt;/Image&gt;
&lt;/MessageWithImage&gt;
&lt;/Messages&gt;
</pre>

[<img class="aligncenter size-full wp-image-384" title="SharedFolder DataBus" src="http://www.serrate.es/wp-content/uploads/2012/01/DataBus_image.png" alt="SharedFolder DataBus" width="588" height="145" srcset="http://www.serrate.es/wp-content/uploads/2012/01/DataBus_image.png 588w, http://www.serrate.es/wp-content/uploads/2012/01/DataBus_image-300x73.png 300w" sizes="(max-width: 588px) 100vw, 588px" />](http://www.serrate.es/wp-content/uploads/2012/01/DataBus_image.png)

El ejemplo puede descargarse de: <a href="https://github.com/mserrate/NServiceBus.Samples" target="_blank">https://github.com/mserrate/NServiceBus.Samples</a>

**<span style="text-decoration: underline;">Nota</span>: Quiero insistir en que dado el tamaño del que disponemos en sistemas de mensajería _on-premise_ la necesidad de usar el DataBus fuera de casos extraordinarios puede significar que no estamos diseñando correctamente los mensajes.**