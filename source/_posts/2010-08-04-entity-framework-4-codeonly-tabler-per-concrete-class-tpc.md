---
id: 133
title: 'Entity Framework 4 CodeOnly &#8211; Table per concrete Class (TPC)'
date: 2010-08-04T18:40:12+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=133
permalink: /2010/08/04/entity-framework-4-codeonly-tabler-per-concrete-class-tpc/
categories:
  - Code Only
  - Entity Framework 4
tags:
  - CodeOnly
  - EF4
  - TPC
---
En el último post acerca de le herencia en Entity Framework 4 y Code Only, vamos a ver la estrategia de **Table per concrete Class** (TPC).

La diferencia con las estrategias de los ejemplos anteriores: <a href="http://www.serrate.es/2010/07/19/entity-framework-4-codeonly-table-per-hierarchy-tph/" target="_self">Table per Hierarchy</a> y <a href="/2010/07/21/entity-framework-4-codeonly-table-per-type-tpt/" target="_self">Table per Type</a> es que, en este caso, la clase base no tiene ningún tipo de representación en las base de datos y sólo tiene sentido a nivel de clases del modelo.

A continuación vemos como tenemos definido una tabla por subtipo:

[<img class="aligncenter size-full wp-image-139" title="TablaTpC" src="http://www.serrate.es/wp-content/uploads/2010/08/TablaTpC.png" alt="TablaTpC" width="450" height="259" srcset="http://www.serrate.es/wp-content/uploads/2010/08/TablaTpC.png 450w, http://www.serrate.es/wp-content/uploads/2010/08/TablaTpC-300x172.png 300w" sizes="(max-width: 450px) 100vw, 450px" />](http://www.serrate.es/wp-content/uploads/2010/08/TablaTpC.png)

Ahora vamos a definir los mapeos de la base de datos con nuestras clases.  

Para el subtipo **Book**:

<pre class="brush: csharp; title: ; notranslate" title="">public class BooksTpCConfiguration : EntityConfiguration&lt;Book&gt;
{
    public BooksTpCConfiguration()
    {
        this
            .HasKey(b =&gt; b.Id)
            .MapSingleType(
                b =&gt; new
                {
                    Id = b.Id,
                    Name = b.Name,
                    Price = b.Price,
                    Pages = b.PageNumber,
                    Author = b.Author
                }
            )
            .ToTable("BooksTpC");
    }
}
</pre>

<!--more-->Para el subtipo 

**Television**:

<pre class="brush: csharp; title: ; notranslate" title="">public class TelevisionsTpCConfiguration : EntityConfiguration&lt;Television&gt;
{
    public TelevisionsTpCConfiguration()
    {
        this
            .HasKey(t =&gt; t.Id)
            .MapSingleType(
                t =&gt; new
                {
                    Id = t.Id,
                    Name = t.Name,
                    Price = t.Price,
                    Size = t.Size,
                    Brand = t.Brand,
                }
            )
            .ToTable("TelevisionsTpC");
    }
}
</pre>

 Para el subtipo **Shirt**:

<pre class="brush: csharp; title: ; notranslate" title="">public class ShirtsTpCConfiguration : EntityConfiguration&lt;Shirt&gt;
{
    public ShirtsTpCConfiguration()
    {
        this
            .HasKey(s =&gt; s.Id)
            .MapSingleType(
                s =&gt; new
                {
                    Id = s.Id,
                    Name = s.Name,
                    Price = s.Price,
                    Colour = s.Colour,
                    Size = s.Size,
                }
            )
            .ToTable("ShirtsTpC");
    }
}
</pre>

Vemos que mediante la expresión **MapSingleType** cada clase mapea las **propiedades propias del subtipo y de la clase base.**

Para crear las instancias a las que añadimos nuestros objetos utilizaremos una clase que herede de **ObjectContext.** En ella definimos sólo los subtipos:

<pre class="brush: csharp; title: ; notranslate" title="">public class ModelTpCContext : ObjectContext
{
    public ModelTpCContext(EntityConnection connection)
        : base(connection)
    {
        DefaultContainerName = "ModelTpCContext";
    }

    public IObjectSet&lt;Book&gt; Books
    {
        get { return base.CreateObjectSet&lt;Book&gt;(); }
    }

    public IObjectSet&lt;Television&gt; Televisions
    {
        get { return base.CreateObjectSet&lt;Television&gt;(); }
    }

    public IObjectSet&lt;Shirt&gt; Shirts
    {
        get { return base.CreateObjectSet&lt;Shirt&gt;(); }
    }
}
</pre>

 Y para terminar con el ejemplo, vemos el método que configura y persiste los 3 subtipos tratados:

<pre class="brush: csharp; title: ; notranslate" title="">public void AddProducts()
{
    // crear la conexión a la BBDD y crear el contexto de EF
    SqlConnection conn = new SqlConnection("Data Source=(local);Initial Catalog=Serrate.CodeOnly;Integrated Security=True;MultipleActiveResultSets=True;");
    var builder = new ContextBuilder&lt;ModelTpCContext&gt;();
    // debemos añadir las 3 configuraciones que equivalen a cada tabla
    builder.Configurations.Add(new BooksTpCConfiguration());
    builder.Configurations.Add(new TelevisionsTpCConfiguration());
    builder.Configurations.Add(new ShirtsTpCConfiguration());
    var context = builder.Create(conn);

    // añadimos todas las entidades a Products
    var book = new Book()
    {
        Id = Guid.NewGuid(),
        Name = "Clash of Kings",
        Price = 60.30M,
        PageNumber = 489,
        Author = "George R. R. Martin"
    };
    context.Books.AddObject(book);

    var tv = new Television()
    {
        Id = Guid.NewGuid(),
        Name = "Led TV",
        Price = 999.99M,
        Brand = "Samsung",
        Size = 40
    };
    context.Televisions.AddObject(tv);

    var shirt = new Shirt()
    {
        Id = Guid.NewGuid(),
        Name = "Kamtxatka Shirt",
        Price = 20.50M,
        Colour = 3,
        Size = "XL"
    };
    context.Shirts.AddObject(shirt);
    // guardamos los cambios
    context.SaveChanges();
}

</pre>