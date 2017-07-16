---
title: 'NServiceBus 3.0 - Message Mutators'
date: 2012-01-11T18:55:57+00:00
categories:
  - SOA
tags:
  - NServiceBus
  - SOA
---
Con la entrada del nuevo año ya falta menos para tener la versión definitiva de NServiceBus 3.0. Esta versión traerá un gran número de nuevas funcionalidades. Entre ellas:

  * Integración de **RavenDB** para la persistencia de Sagas y almacenamiento de la información de subscripciones.
  * Soporte para **Azure**.
  * Transportar grandes cantidades de datos gracias al **DataBus**.
  * Una configuración mucho más sencilla y **no intrusiva**.

Podéis leer un resumen de estas funcionalidades y descargar la beta desde aquí: <a href="http://www.nservicebus.com/NServiceBusV3NewFeatures.aspx" target="_blank">http://www.nservicebus.com/NServiceBusV3NewFeatures.aspx</a>

En este post hablaremos de los **MessageMutators**, una nueva funcionalidad que actúa como interceptores de los mensajes al llegar y/o al salir de nuestro endpoint. Así, podemos ejecutar transformaciones a nuestros mensajes de forma dinámica.<!--more-->

Tenemos dos tipos de MessageMutators:

  * **IMessageMutator:** Esta interfaz nos permite ejecutar cualquier acción sobre un mensaje en concreto.
  * **IMutateTransportMessages:** Esta interfaz se diferencia de la anterior en que aplicamos las acciones sobre un TransportMessage, este puede contener múltiples mensajes. Realmente, estos conceptos son más difíciles de explicar que de implementar. Por eso, vamos a amenizar el artículo mediante un ejemplo.

Tenemos el siguiente mensaje, muy simple:

{% codeblock lang:csharp %}
public class SampleMessage : IMessage
{
    public string MyProperty { get; set; }
    public int MyNumber { get; set; }
}
{% endcodeblock %}

Utilizaremos un interceptor para encriptar los mensajes antes de salir del endpoint y desencriptarlos al entrar:

{% codeblock lang:csharp %}
public class TransportEncryptionMutator : IMutateTransportMessages
{
    public void MutateIncoming(TransportMessage transportMessage)
    {
        var decrypt = EncryptionHelper.Decrypt(transportMessage.Body);

        transportMessage.Body = decrypt;
    }

    public void MutateOutgoing(object[] messages, TransportMessage transportMessage)
    {
        var encrypt = EncryptionHelper.Encrypt(transportMessage.Body);

        transportMessage.Body = encrypt;
    }
}
{% endcodeblock %}

Y finalmente lo registramos mediante el contenedor de IoC de NServiceBus.

{% codeblock lang:csharp %}
public class Initialization : IWantCustomInitialization
{
    public void Init()
    {
        Configure.Instance.Configurer
         .ConfigureComponent<TransportEncryptionMutator>(
            DependencyLifecycle.InstancePerCall);
    }
}
{% endcodeblock %}

Ahora solo falta referenciarlo desde los proyectos cliente y servidor y cuando enviemos un mensaje, estos serán encriptados al enviar y desencriptados al recibir.

{% img /uploads/2012/01/messagemutators.png 597 342 '"Message Mutators"' '"Message Mutators"' %}

Tal y como podemos ver en la imagen: sin aplicar el MessageMutator y aplicándolo.

El ejemplo puede descargarse de: <a href="https://github.com/mserrate/NServiceBus.Samples" target="_blank">https://github.com/mserrate/NServiceBus.Samples</a>