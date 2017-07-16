---
title: Poor Man's CQRS - BDD con MSpec
date: 2011-12-01T09:20:16+00:00
categories:
  - CQRS
tags:
  - BDD
  - CQRS
  - DDD
---
[El artículo forma parte de una serie de artículos: [http://www.serrate.es/2011/11/22/poor-mans-cqrs/]](http://www.serrate.es/2011/11/22/poor-mans-cqrs/)

En el proyecto de Testing se ha utilizado **<a href="https://github.com/machine/machine.specifications" target="_blank">Machine.Specifications (MSpec)</a>** para realizar **<a href="http://en.wikipedia.org/wiki/Behavior_Driven_Development" target="_blank">Behavior Driven Development (BDD)</a>**.

MSpec utiliza las cláusulas **Establish/Because/It** equivalente a **Given/When/Then** disponible en otros frameworks como SpecFlow.

La forma de uso seria la siguiente:

  * **Establish:** Se usa para establecer el estado inicial.
  * **Because:** Definimos la acción que vamos a testear.
  * **It:** Comprobamos que el resultado es el esperado.

<!--more-->Es muy importante que la cláusula Because conste de una sola línea ya que debemos testear 

**una única acción**. Si necesitamos más de una línea en Because significará seguramente que hemos establecido un contexto demasiado grande.

En el caso que nos ocupa se ha creado una clase abstracta para establecer configuraciones iniciales para todos los casos de test. Podemos ver como se ha creado un **Mock** del ServiceLocator usado por la clase **DomainEvents** y como también se ha establecido la cláusula **Cleanup** para que en cada test se vacíe la lista de callbacks de los eventos de dominio. Además, hemos creado un método Factory para instanciar el agregado **Project** con algunos datos por defecto.

{% codeblock lang:csharp %}
public abstract class ProjectSpecs
{
	protected static Project project;

	Establish basecontext = () =>
	{
		// mocking the service locator used by Domain Events
		var mock = new Moq.Mock<IServiceLocator>();
		mock.Setup(x => x.GetAllInstances(typeof(IEventHandler<>))).Returns(new List<object>());
		ServiceLocator.SetLocatorProvider(() => mock.Object);
	};

	Cleanup resources = () =>
	{
		DomainEvents.ClearCallbacks();
	};

	protected static Project CreateProject()
	{
		return new Project("p1", "Project 1", DateTime.Now.AddDays(5));
	}
}
{% endcodeblock %}

Veremos a continuación tres ejemplos diferentes de escenarios de test.

En el primero vamos a probar lo siguiente:

> Cuando cerramos un proyecto, se debe crear un evento ProjectClosedEvent

{% codeblock lang:csharp %}
[Subject(typeof(Project))]
public class when_closing_a_project
	: ProjectSpecs
{
	static IEvent @event;

	Establish context = () =>
		{
			project = CreateProject();
			DomainEvents.Register(e => @event = e);
		};

	Because of = () =>
		project.Close();

	It should_create_a_projectclosed_event = () =>
		@event.ShouldBeOfType<ProjectClosedEvent>();
}
{% endcodeblock %}

La segunda prueba será la siguiente:

> Cuando cerremos un proyecto, si el proyecto ya está cerrado no debe lanzarse el evento ProjectClosedEvent

{% codeblock lang:csharp %}
[Subject(typeof(Project))]
public class when_closing_a_closed_project
    : ProjectSpecs
{
    static IEvent @event;

    Establish context = () =>
    {
        project = CreateProject();
        project.Close();
        DomainEvents.Register(e => @event = e);
    };

    Because of = () =>
        project.Close();

    It project_closed_event_should_be_null = () =>
        @event.ShouldBeNull();
}
{% endcodeblock %}

Finalmente, la tercera prueba será la siguiente:

> Cuando desactivamos un proyecto cerrado, debe lanzarse una excepción del tipo ProjectIsClosedException

{% codeblock lang:csharp %}
[Subject(typeof(Project))]
public class when_deactivating_a_closed_project
	: ProjectSpecs
{
	static Exception exception;

	Establish context = () =>
	{
		project = CreateProject();
		project.Close();
	};

	Because of = () =>
		exception = Catch.Exception(() => project.Deactivate());

	It should_rise_a_project_is_closed_exception = () =>
		exception.ShouldBeOfType<ProjectIsClosedException>();
}
{% endcodeblock %}

La salida que obtenemos cuando ejecutamos el proyecto de test mediante línea de comandos es la siguiente:

> Project, when closing a project
  
> » should create a projectclosed event
> 
> Project, when closing a closed project
  
> » project closed event should be null
> 
> Project, when deactivating a closed project
  
> » should rise a project is closed exception