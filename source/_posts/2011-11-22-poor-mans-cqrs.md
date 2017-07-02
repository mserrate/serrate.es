---
id: 281
title: Poor Man’s CQRS
date: 2011-11-22T12:45:39+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=281
permalink: /2011/11/22/poor-mans-cqrs/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
categories:
  - CQRS
tags:
  - CQRS
---
[Url del proyecto: <a href="https://github.com/mserrate/PoorMansCQRS" target="_blank">https://github.com/mserrate/PoorMansCQRS</a>]

Muchas veces cuando se habla de Command-Query Responsibility Segregation (CQRS) empiezan a surgir conceptos tales como: **Event Sourcing**, **2-Phase Commit**, **Snapshots**, **mensajería asíncrona**, **eventual consistency** y multitud más que hacen que mucha gente piense que CQRS es una arquitectura compleja y que solo se puede utilizar para desarrollar aplicaciones tipo Amazon, Twitter y Facebook (vamos, los proyectos que nos surgen cada día&#8230;).

Por el contrario, soy de la opinión de que el uso de este patrón nos permite implementar de forma exitosa una aplicación basada en **Domain-Driven Design** (DDD). Y es que **no se entiende CQRS sin DDD**.
  
Es decir, no hace falta implementar Amazon para utilizar DDD y CQRS pero tampoco vayamos a implementarlo en la web de la panadería de nuestro cuñado en Villaconejo del Monte.<!--more-->

Antes de nada, aviso para navegantes: **Para una explicación de que es CQRS** y que conceptos vienen ligados al patrón podéis mirar el siguiente webcast que un servidor grabó para SecondNug en Mayo &#8217;11:
  
<https://msevents.microsoft.com/CUI/EventDetail.aspx?culture=es-ES&EventID=1032486026&CountryCode=ES>

En él veréis teoría y un ejemplo completo del uso de Event Sourcing, Snapshots, Task Based UI, etc. Recomiendo su visionado para entender cómo desarrollar aplicaciones altamente escalables vía CQRS y, de la misma forma, leer los artículos de los pioneros en esta clase de arquitecturas: **Greg Young** y **Udi Dahan**.

La motivación de este proyecto es básicamente el contrario, **acercar CQRS** y ver las ventajas que conlleva su uso sin necesidad de incorporar conceptos más complejos y con tecnologías ya conocidas:

  * ASP.NET MVC 3
  * NHibernate y Fluent NHibernate
  * Castle Windsor
  * MSpec

Por eso mismo, la ejecución de la aplicación es totalmente **síncrona** (por lo tanto, **no escalable**), sin la existencia de eventos externos ni colas, y una sola base de datos tanto para lecturas y escrituras. Al utilizar contratos para la implementación se podría migrar sin mucha dificultad a una implementación más &#8220;estándar&#8221; de CQRS pero para ello ya existen varios ejemplos por internet. Pero bueno, por esa razón el título de Poor Man’s CQRS 😉

## Solución

La solución está compuesta por los siguientes proyectos que iré explicando en siguientes posts:

  * PoorMansCQRS.Web
  * PoorMansCQRS.ReadModel
  * PoorMansCQRS.Commands
  * PoorMansCQRS.CommandHandlers
  * PoorMansCQRS.Domain
  * PoorMansCQRS.Data
  * PoorMansCQRS.Infrastructure
  * PoorMansCQRS.Testing