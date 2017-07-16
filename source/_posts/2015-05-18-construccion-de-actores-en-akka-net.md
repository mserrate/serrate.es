---
title: Construcción de actores en Akka.net
date: 2015-05-18T12:46:39+00:00
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

{% codeblock lang:csharp %}
public class DoSomething
{
    public string MyName { get; set; }
}

public class  DoSomethingElse
{
}
{% endcodeblock %}

### Untyped Actor

Es la forma más básica para definir un actor y nos obliga a sobreescribir el método OnReceive. Al ser de más difícil lectura se recomienda la opción TypedActor o ReceiveActor.

{% codeblock lang:csharp %}
public class MyUntypedActor : UntypedActor
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
{% endcodeblock %}

### TypedActor

Con TypedActor se hace mucho más explícito el tipo de mensajes que un actor puede procesar y aumenta la legibilidad del código.

{% codeblock lang:csharp %}
public class MyTypedActor : TypedActor,
    IHandle<DoSomething>,
    IHandle<DoSomethingElse>
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
{% endcodeblock %}

### ReceiveActor

ReceiveActor &#8211; que hereda de UntypedActor &#8211; nos proporciona la capacidad de seleccionar cómo gestionar los mensajes de forma más sofisticada.

En este caso, le decimos que trate de forma distinta los mensajes de tipo _DoSomething_ en que el valor de la propiedad empieza por &#8220;Barcelona&#8221;. El orden en la declaración es importante!

{% codeblock lang:csharp %}
public class MyReceiveActor : ReceiveActor
{
    public MyReceiveActor()
    {
        Receive<DoSomething>(
            x => x.MyName.StartsWith("Barcelona"),
            x => Console.WriteLine("DoSomething -- Barcelona"));
        Receive<DoSomething>(
            x => Console.WriteLine("DoSomething"));
        Receive<DoSomethingElse>(
            x => Console.WriteLine("DoSomethingElse"));
    }
}
{% endcodeblock %}

Happy coding!