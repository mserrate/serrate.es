---
id: 177
title: 'NHibernate: Paginando con DetachedCriteria'
date: 2011-01-16T14:52:50+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=177
permalink: /2011/01/16/nhibernate-paginando-con-detachedcriteria/
categories:
  - NHibernate
tags:
  - NHibernate
  - ORM
---
El propósito de este post es ver cómo podemos paginar de forma óptima en servidor mediante el objeto **DetachedCriteria**. El objetivo es evitar que nuestros controles de presentación hagan el trabajo sucio paginando en memoria los resultados al obtener todos los elementos de la base de datos en cada petición.

Por trivial que parezca, para poder mostrar los resultados de forma amigable al usuario en, por ejemplo un grid, la consulta debe devolver los elementos de la página en cuestión y el número total de elementos. Para ello se requieren dos consultas, una para los elementos y otra para el total, pero no suena muy práctico realizar dos consultas separadamente no? La gracia aquí está en agrupar las dos consultas en una y así optimizar el rendimiento. 

**Nota**: En caso de utilizar directamente un objeto de tipo ICriteria podríamos hacer uso directamente de las clausulas Future<T>  y FutureValue<T> que hacen internamente hacen uso de Multi Queries como haremos aquí. Más información en éste <a href="http://ayende.com/Blog/archive/2009/04/27/nhibernate-futures.aspx" target="_blank">post</a> de Ayende. 

Antes de nada… Que es DetachedCriteria? Pues no deja de ser un objeto criteria que definimos sin tener acceso a ISession. Esto nos permite definir nuestras consultas y poderlas ejecutar más tarde en cualquier otra parte dentro de un contexto de ISession. 

A continuación vemos una declaración típica para DetachedCriteria: 

<pre class="brush: csharp; title: ; notranslate" title="">// Creamos el objeto DetachedCriteria
var detachedCriteria = DetachedCriteria.For&lt;Customer&gt;("c");

// Agregamos filtros a la consulta
if (!string.IsNullOrEmpty(criteria.Name)
    && !string.IsNullOrWhiteSpace(criteria.Name))
{
    detachedCriteria.Add(Expression.Like("Name", criteria.Name, MatchMode.Anywhere));
}
</pre>

Más adelante, en un contexto de ISession, tenemos el código para la paginación. Vemos los comentarios en cada línea: 

<pre class="brush: csharp; title: ; notranslate" title="">public PagedList&lt;T&gt; FindPaged(DetachedCriteria criteria, int startIndex, int pageSize)
{
    using (var session = GetSession())
    {
        // Clonamos nuestro DetachedCriteria
        DetachedCriteria itemsCriteria = CriteriaTransformer.Clone(criteria);
        // Especificamos los parámetros de paginación:
        // 1) El índice del primer elemento a obtener
        itemsCriteria.SetFirstResult(startIndex * pageSize);
        // 2) El número máximo de resultados
        itemsCriteria.SetMaxResults(pageSize);

        // Transformamos nuestra consulta original para que nos devuelva el número de elementos
        DetachedCriteria countCriteria = CriteriaTransformer.TransformToRowCount(criteria);

        // Creamos un objeto MultiCriteria para agrupar las dos consultas
        IMultiCriteria multiCriteria = session.CreateMultiCriteria();
        multiCriteria.Add(itemsCriteria);
        multiCriteria.Add(countCriteria);
        IList multiResult = multiCriteria.List();

        // En posición 0 de la lista tenemos los elementos paginados
        IList pagedElementsUntyped = multiResult[0] as IList;
        // Con la extensión Cast&lt;T&gt; obtenemos la lista genérica de resultados
        IEnumerable&lt;T&gt; pagedElements = pagedElementsUntyped.Cast&lt;T&gt;();

        // En posición 1 de la lista tenemos el total de elementos
        int totalCount = Convert.ToInt32(((IList)multiResult[1])[0]);

        // Finalmente devolvemos la clase PagedList que encapsula los dos resultados
        return new PagedList&lt;T&gt;(pagedElements, totalCount);
    }
}
</pre>

La clase **PagedList<T>** es simplemente un contenedor de los resultados: 

<pre class="brush: csharp; title: ; notranslate" title="">public class PagedList&lt;T&gt;
{
    public IEnumerable&lt;T&gt; Items { get; protected set; }

    public int TotalItems { get; protected set; }

    public PagedList(IEnumerable&lt;T&gt; items, int totalItems)
    {
        this.Items = items;
        this.TotalItems = totalItems;
    }
}
</pre>

  

A continuación mostramos como se termina haciendo la consulta a la base de datos meditante SQL Profiler: 

<pre class="brush: sql; title: ; notranslate" title="">exec sp_executesql N'SELECT top 5 this_.Id as CU1_0_0_, this_.Name as CU2_0_0_ FROM Customers this_ WHERE this_.Name like @p0;
SELECT count(*) as y0_ FROM Customers this_ WHERE this_.Name like @p1;
',N'@p0 nvarchar(6),@p1 nvarchar(6)',@p0=N'%name%',@p1=N'%name%' 
</pre>

En este pequeño post hemos visto como con IMultiCriteria y NHibernate podemos hacer paginaciones optimizadas en servidor para que nuestros controles de presentación no tengan que hacer los cálculos en memoria obteniendo todos los datos de la base de datos en cada petición.