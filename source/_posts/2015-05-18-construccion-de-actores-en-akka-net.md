---
id: 448
title: Construcción de actores en Akka.net
date: 2015-05-18T12:46:39+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=448
permalink: /2015/05/18/construccion-de-actores-en-akka-net/
categories:
  - Actor Model
  - Arquitectura
tags:
  - AkkaDotNet
---
El modelo de actores se caracteriza por establecer un modelo de computación basado en el <a href="http://www.reactivemanifesto.org/" target="_blank">Reactive Manifesto</a> que nos permite construir:

  * Sistemas altamente concurrentes
  * Sistemas escalables tanto horizontalmente como verticalmente
  * Sistemas fiables y tolerantes a errores

Para empezar vamos a ver como se define un actor ya que tenemos 3 formas distintas dependiendo de nuestras necesidades. Los actores en Akka.net hacen un uso extensivo de _pattern matching_ para seleccionar cómo se van a gestionar los mensajes dependiendo de su tipo o valor.

<!--more-->Para nuestro ejemplo definimos dos simples mensajes POCO:

<pre class="brush: csharp; title: ; notranslate" title="">public class DoSomething
{
    public string MyName { get; set; }
}

public class  DoSomethingElse
{
}
</pre>

### Untyped Actor

Es la forma más básica para definir un actor y nos obliga a sobreescribir el método OnReceive. Al ser de más difícil lectura se recomienda la opción TypedActor o ReceiveActor.

<pre class="brush: csharp; title: ; notranslate" title="">public class MyUntypedActor : UntypedActor
{
    protected override void OnReceive(object message)
    {
        if (message is DoSomething)
        {
            Console.WriteLine("DoSomething");
        }
        else if (message is DoSomethingElse)
        {
            Console.WriteLine("DoSomethingElse");
        }
    }
}
</pre>

### TypedActor

Con TypedActor se hace mucho más explícito el tipo de mensajes que un actor puede procesar y aumenta la legibilidad del código.

<pre class="brush: csharp; title: ; notranslate" title="">public class MyTypedActor : TypedActor,
    IHandle&lt;DoSomething&gt;,
    IHandle&lt;DoSomethingElse&gt;
{
    public void Handle(DoSomething message)
    {
        Console.WriteLine("DoSomething");
    }

    public void Handle(DoSomethingElse message)
    {
        Console.WriteLine("DoSomethingElse");
    }
}
</pre>

### ReceiveActor

ReceiveActor &#8211; que hereda de UntypedActor &#8211; nos proporciona la capacidad de seleccionar cómo gestionar los mensajes de forma más sofisticada.

En este caso, le decimos que trate de forma distinta los mensajes de tipo _DoSomething_ en que el valor de la propiedad empieza por &#8220;Barcelona&#8221;. El orden en la declaración es importante!

<pre class="brush: csharp; title: ; notranslate" title="">public class MyReceiveActor : ReceiveActor
{
    public MyReceiveActor()
    {
        Receive&lt;DoSomething&gt;(
            x =&gt; x.MyName.StartsWith("Barcelona"),
            x =&gt; Console.WriteLine("DoSomething -- Barcelona"));
        Receive&lt;DoSomething&gt;(
            x =&gt; Console.WriteLine("DoSomething"));
        Receive&lt;DoSomethingElse&gt;(
            x =&gt; Console.WriteLine("DoSomethingElse"));
    }
}
</pre>

Happy coding!