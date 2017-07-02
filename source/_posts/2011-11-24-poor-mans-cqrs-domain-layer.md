---
id: 290
title: 'Poor Man&#8217;s CQRS &#8211; Domain Layer'
date: 2011-11-24T17:22:10+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=290
permalink: /2011/11/24/poor-mans-cqrs-domain-layer/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
categories:
  - CQRS
tags:
  - CQRS
  - DDD
---
[El artículo forma parte de una serie de artículos: [http://www.serrate.es/2011/11/22/poor-mans-cqrs/]](http://www.serrate.es/2011/11/22/poor-mans-cqrs/)

Empezaremos detallando la parte principal de la aplicación: la capa de **dominio**. Aunque el ejemplo que nos ocupa consta simplemente de dos míseras clases, es importante que interioricemos algunos conceptos.

Diseñar un modelo de dominio es una tarea altamente compleja: definir las entidades que formaran parte de los **agregados**, el **límite** de éstos, qué entidad será el **agregado raíz**, distinguir **value objects**, etc. y, curiosamente, es la parte que menor relación tiene con CQRS.

Como siempre, la recomendación principal que puedo dar es la de leerse y empaparse del libro <a href="http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215" target="_blank">Domain-Driven Design</a> de **Eric Evans**.

<!--more-->Un pequeño listado a la hora de definir correctamente los agregados:

  * Los objetos externos únicamente podrán referenciar al agregado raíz. Si se necesita acceder a una entidad interna será siempre a través del agregado raíz.
  * Los agregados deben ser **límites consistentes** para transacciones, distribuciones y concurrencia.
  * En general, la regla de _eliminar en cascada_ puede sernos útil para saber si un grupo de entidades/VO formaran parte de un agregado. Pero nunca tomarlo como verdad absoluta.

Por el contrario hay varias formas de modelar los agregados de forma <span style="text-decoration: underline;">incorrecta</span>:

  * Caer en la trampa de realizar el diseño por conveniencia composicional y que los agregados sean demasiado grandes.
  * Por el contrario, diseñar los agregados demasiado granulares perdiendo de esta manera la protección de los invariantes. Ningún extremo es bueno 🙂

Además, es fundamental tener en cuenta la **encapsulación** y no crear **acoplamiento innecesario** en el diseño de entidades en DDD, esto es, **no tener un constructor público por defecto** y <span style="text-decoration: underline;"><strong>no escribir getters y setters</strong></span>. Nunca debemos publicar el estado de nuestras entidades al exterior!!

Vamos a ver el ejemplo con el agregado raíz Project (versión reducida). Como veis, la clase no tiene propiedades públicas ni constructor público por defecto. Además las reglas de negocio están encapsuladas en la clase por lo que no necesitamos que accedan al estado de la clase desde el exterior. En el método para desactivar el proyecto, en caso de que el proyecto esté cerrado (_IsProjectClosed_) no se podrá desactivar por lo que lanzamos una excepción de negocio (_ProjectIsClosedException_).

<pre class="brush: csharp; title: ; notranslate" title="">public class Project : AggregateRoot
{
	private Guid _id;
	private string _code;
	private string _name;
	private DateTime _deliveryDate;
	private ProjectStatus _status;
	private IList _tasks;

	private Project()
	{ }

	public Project(string code, string name, DateTime deliveryDate)
	{
		_id = Guid.NewGuid();
		_code = code;
		_name = name;
		_status = ProjectStatus.Active;
		IsValidDeliveryDate(deliveryDate);
		_deliveryDate = deliveryDate;
		_tasks = new List();

		ApplyEvent(new ProjectAddedEvent(_id, _code, _name, _deliveryDate));
	}

	public void Deactivate()
	{
		IsProjectClosed();

		if (_status != ProjectStatus.Inactive)
		{
			_status = ProjectStatus.Inactive;
			ApplyEvent(new ProjectDeactivatedEvent(_id));
		}
	}

	private void IsProjectClosed()
	{
		if (_status == ProjectStatus.Closed)
			throw new ProjectIsClosedException("The action could not be processed because the project is closed");
	}
}
</pre>

<a href="https://github.com/mserrate/PoorMansCQRS/blob/master/PoorMansCQRS.Domain/Project.cs" target="_blank">(Ver la clase entera Project</a>)

A continuación presentamos la clase base para los agregados raíz: AggregateRoot que es muy sencilla y la única particularidad es la del método ApplyEvent que nos sirve para lanzar un evento cada vez que ocurre una acción en el agregado:

<pre class="brush: csharp; title: ; notranslate" title="">public abstract class AggregateRoot : IEntity
{
	public abstract Guid Id { get; }

	internal void ApplyEvent(T args)
		where T : IEvent
	{
		DomainEvents.DomainEvents.Raise(args);
	}
}

public interface IEntity
{
	Guid Id { get; }
}
</pre>

Para lanzar los eventos utilizamos el concepto de Domain Events de Udi Dahan: <a href="http://www.udidahan.com/2009/06/14/domain-events-salvation/" target="_blank">http://www.udidahan.com/2009/06/14/domain-events-salvation/</a>