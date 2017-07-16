---
title: Introducción a Managed Extensibility Framework (MEF)
date: 2010-09-19T21:10:17+00:00
categories:
  - Managed Extensibility Framework
tags:
  - IoC
  - Managed Extensibility Framework
---
## Qué es MEF y que te aporta

MEF es una tecnología que permite desarrollar aplicaciones extensibles. La gran ventaja que nos ofrece MEF es que no necesitamos diseñar la aplicación conociendo qué extensiones formaran parte de ella ni la implementación interna de las propias extensiones.

Por extensible nos referimos a que nuestra aplicación puede estar en producción y de forma dinámica añadir, reemplazar, eliminar las extensiones que tenemos sin necesidad ni de recompilar ni reiniciar la aplicación.

MEF viene con .NET Framework 4 en la librería: **System.ComponentModel.Composition.**
  
## MEF vs IoC Container

Aunque pueda parecer que MEF e IoC  ofrecen la misma funcionalidad la diferencia consiste en que tratan de resolver problemas distintos.  

El objetivo básico de un framework de IoC es el de ofrecer **desacoplamiento** entre componentes y nos resuelve dependencias que <span style="text-decoration: underline;">conocemos</span>. Esto nos permite que nuestra aplicación sea modular y testeable.

En cambio, en MEF, el principal objetivo es la **extensibilidad**: el desacoplamiento es una consecuencia. Sabemos que las extensiones que se vayan a implementar cumplirán un contrato pero no sabremos si sólo habrá una extensión, muchas o ninguna.

## Ejemplo: Extensión IHelloWorld

<!--more-->Vamos a ver un ejemplo de cómo consumir nuestras extensiones con MEF. El ejemplo es muy sencillo y únicamente tenemos una aplicación de consola que consumirá extensiones que nos devuelven el mensaje Hola Mundo en el idioma definido en la extensión.

Primero declaramos en el ensamblado _Serrate.MEFDemo.HelloWorld.Interface_ el contrato que deben cumplir:

{% codeblock lang:csharp %}
namespace Serrate.MEFDemo.HelloWord.Interface
{
    /// <summary>
    /// Contrato para extensiones
    /// </summary>
    public interface IHelloWorld
    {
        string GetMessage();
    }
}
{% endcodeblock %}

Posteriormente en nuestra aplicación de consola _Serrate.MEFDemo.HelloWorldApp_ tenemos la clase Consumer encargada de consumir, como no, las extensiones de terceros.

{% codeblock lang:csharp %}
namespace Serrate.MEFDemo.HelloWorldApp
{
    /// <summary>
    /// Clase encargada de consumir nuestras extensiones
    /// </summary>
    public class Consumer
    {
        /// <summary>
        /// Esta lista importará las extensiones que cumplan
        /// nuestros requisitos
        /// </summary>
        [ImportMany]
        public List<IHelloWorld> HelloWorldPlugins { get; set; }

        /// <summary>
        /// Lectura de nuestras extensiones
        /// </summary>
        public void Read()
        {
            // Configuramos nuestro contenedor
            this.Setup();

            // Recorremos la lista de extensiones
            foreach (var plugin in HelloWorldPlugins)
            {
                Console.WriteLine(plugin.GetMessage());
            }

            Console.ReadLine();
        }

        private void Setup()
        {
            // a partir del directorio de la aplicación buscamos
            // en una ruta relativa nuestro directorio de extensiones
            string appDirectory = AppDomain.CurrentDomain.BaseDirectory;
            DirectoryInfo extensionsDirectory =
                new DirectoryInfo(appDirectory + "..\\..\\..\\extensions");

            // creamos el catálogo a partir del directorio
            var catalog = new DirectoryCatalog(extensionsDirectory.FullName);

            // componemos las extensiones para poderlas usar
            var compositionContainer = new CompositionContainer(catalog);
            compositionContainer.ComposeParts(this);
        }
    }
}
{% endcodeblock %}

Como podemos tener varias extensiones cumpliendo el mismo contrato, en nuestra aplicación declaramos una lista de este contrato, y la decoramos con el atributo **[ImportMany]**:

{% codeblock lang:csharp %}
    /// <summary>
    /// Esta lista importará las extensiones que cumplan
    /// nuestros requisitos
    /// </summary>
    [ImportMany]
    public List<IHelloWorld> HelloWorldPlugins { get; set; }
{% endcodeblock %}

Seguidamente podemos realizar la lectura de nuestras extensiones:

{% codeblock lang:csharp %}
    /// <summary>
    /// Lectura de nuestras extensiones
    /// </summary>
    public void Read()
    {
        // Configuramos nuestro contenedor
        this.Setup();

        // Recorremos la lista de extensiones
        foreach (var plugin in HelloWorldPlugins)
        {
            Console.WriteLine(plugin.GetMessage());
        }

        Console.ReadLine();
    }
{% endcodeblock %}

En el método Setup indicamos el directorio de nuestras extensiones y realizamos la carga de éstas:

{% codeblock lang:csharp %}
    private void Setup()
    {
        // a partir del directorio de la aplicación buscamos
        // en una ruta relativa nuestro directorio de extensiones
        string appDirectory = AppDomain.CurrentDomain.BaseDirectory;
        DirectoryInfo extensionsDirectory =
            new DirectoryInfo(appDirectory + "..\\..\\..\\extensions");

        // creamos el catálogo a partir del directorio
        var catalog = new DirectoryCatalog(extensionsDirectory.FullName);

        // componemos las extensiones para poderlas usar
        var compositionContainer = new CompositionContainer(catalog);
        compositionContainer.ComposeParts(this);
    }
{% endcodeblock %}

Finalmente, ya sólo nos hace falta crear nuestras extensiones que realizarán el trabajo duro. Indicaremos que las extensiones son exportables mediante el atributo **[Export]**

Crearemos el ensamblado _Serrate.MEFDemo.Plugin.ES_ y allí tendremos nuestra clase que implementará el contrato:

{% codeblock lang:csharp %}
namespace Serrate.MEFDemo.Plugin.ES
{
    [Export(typeof(IHelloWorld))]
    public class HolaMundo : IHelloWorld
    {
        public string GetMessage()
        {
            return "Hola Mundo";
        }
    }
}
{% endcodeblock %}

En otro ensamblado _Serrate.MEFDemo.Plugin.CA_ tendremos definida otra extensión:

{% codeblock lang:csharp %}
namespace Serrate.MEFDemo.Plugin.CA
{
    [Export(typeof(IHelloWorld))]
    public class HolaMon : IHelloWorld
    {
        public string GetMessage()
        {
            return "Hola Món";
        }
    }
}
{% endcodeblock %}

Si colocamos las librerías de nuestras extensiones en la carpeta que espera nuestra aplicación, al ejecutarla ya nos saldrá por pantalla el resultado esperado:

<pre>> Hola Mundo
> Hola Món</pre>

En esta introducción hemos visto como utilizar MEF para que nuestra aplicación pueda ser extendida por nosotros mismos o por terceros de forma realmente sencilla.