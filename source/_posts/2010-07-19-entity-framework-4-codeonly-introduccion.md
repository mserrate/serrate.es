---
title: 'Entity Framework 4 CodeOnly - Introducción'
date: 2010-07-19T18:07:45+00:00
comments: true
categories:
  - Code Only
  - Entity Framework 4
tags:
  - CodeOnly
  - EF4
---
Entity Framework 4.0 nos ofrece varias formas de generar nuestro modelo, la primera, y que ya existía en la primera versión del producto, era generar el modelo **a partir de la base de datos**. Esto nos obligaba a empezar diseñando la base de datos para luego generar nuestras clases, lo cual no es lo deseable.

Una segunda opción es el llamado **model-first**, en que primero diseñamos nuestras clases del dominio y a partir de plantillas T4 y WF &#8211; ya sea las plantillas que trae por defecto o unas creadas por nosotros mismos – nos permite generar el script para crear la base de datos. Esta opción, aun siendo mejor que la anterior no da lugar a mucho juego.

Si realmente lo que queremos es total flexibilidad y no usar ni el diseñador ni pelearnos con el xml del fichero *.**edmx**, podemos obtener la funcionalidad que necesitamos mediante el uso de la librería **Microsoft.Data.Entity.CTP** y disponible <a href="http://www.microsoft.com/downloads/details.aspx?FamilyID=4e094902-aeff-4ee2-a12d-5881d4b0dd3e&displayLang=en" target="_blank">aquí</a>. Como podéis deducir por el nombre aun se encuentra en fase CTP pero ya podemos realizar bastantes cosas interesantes. Para los que hayan trabajado con **nHibernate,** CodeOnly es la alternativa a **Fluent nHibernate**.

<!--more-->A continuación vemos un ejemplo sencillo de cómo se usaría CodeOnly para mapear una clase Student y su relación con otra clase Group:

{% codeblock lang:csharp %}
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int GroupId { get; set; }
    public Group Group { get; set; }
}

public class Group
{
    public int Id { get; set; }
    public ICollection<Student> Students { get; set; }
}

public class CodeOnlyConfiguration : EntityConfiguration<Student>
{
    public CodeOnlyConfiguration()
    {
        // indicamos la clave primária
        HasKey(s => s.Id);
        Property(s => s.Id).IsIdentity();
        // indicamos que el nombre es requerido y long. max.
        Property(s => s.Name).HasMaxLength(125).IsRequired();
        // la relación con su constraint
        Relationship(s => s.Group)
            .FromProperty(g => g.Students)
            .HasConstraint((s, g) => s.GroupId == g.Id);
        // mapeo con la tabla de la BBDD
        MapSingleType(s =>
            new
            {
                ID = s.Id,
                StudentName = s.Name,
                Group_ID = s.GroupId
            })
            .ToTable("dbo.Students");
    }
}
{% endcodeblock %}

### Ejemplos con la herencia

Vamos a mostrar varios ejemplos en que usamos los diferentes tipos de herencia soportados por EF4: Table per Hierarchy (TPH),Table per Type (TPT) y Table per Concrete Class (TPC).

Nuestras clases del dominio serán las mismas para los tres ejemplos y representan una herencia muy simple – no creo que haga falta explicarlas 🙂 -:

{% img  /uploads/2010/07/Diagrama-de-clases.png 450 281 '"Diagrama de clases"' '"Diagrama de clases"' %}

Por comodidad, añado los enlaces a los posts relacionados:

  1. <a href="/2010/07/19/entity-framework-4-codeonly-table-per-hierarchy-tph/" target="_self">Table per Hierarchy (TPH)</a>
  2. <a href="/2010/07/21/entity-framework-4-codeonly-table-per-type-tpt/" target="_self">Table per Type (TPT)</a>
  3. Table per concrete Class (TPC)