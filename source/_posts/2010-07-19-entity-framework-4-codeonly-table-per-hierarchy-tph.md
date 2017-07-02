---
id: 60
title: 'Entity Framework 4 CodeOnly &#8211; Table per Hierarchy (TPH)'
date: 2010-07-19T18:21:07+00:00
author: Marçal
layout: post
guid: http://66.147.240.162/~serratee/?p=60
permalink: /2010/07/19/entity-framework-4-codeonly-table-per-hierarchy-tph/
categories:
  - Code Only
  - Entity Framework 4
tags:
  - CodeOnly
  - EF4
  - TPH
---
En mi <a href="/2010/07/19/entity-framework-4-codeonly-introduccion/" target="_self">post anterior</a> hice una introducción a EF4 CodeOnly. A continuación entraremos en detalle con el tipo de herencia de TPH:

En TPH, utilizamos una tabla para guardar todos los datos de la jerarquía y mediante una columna que actúa como discriminador determinamos el tipo concreto de la fila y que columnas debemos leer.

La tabla de la base de datos es la siguiente:

<p style="text-align: center;">
  <a href="http://www.serrate.es/wp-content/uploads/2010/07/TablaTpH.png"><img class="size-full wp-image-116 aligncenter" title="TablaTpH" src="http://www.serrate.es/wp-content/uploads/2010/07/TablaTpH.png" alt="TablaTpH" width="310" height="270" srcset="http://www.serrate.es/wp-content/uploads/2010/07/TablaTpH.png 310w, http://www.serrate.es/wp-content/uploads/2010/07/TablaTpH-300x261.png 300w" sizes="(max-width: 310px) 100vw, 310px" /></a>
</p>

<p style="text-align: left;">
  La columna que actúa como <strong>discriminador</strong> es ProductType y será ‘S’ para el tipo Shirt, ‘T’ para Television y ‘B’ para Book.
</p>

<!--more-->La magia de los mapeos la realizamos en una clase que hereda de 

**EntityConfiguration** y poniendo la lógica de configuración en el constructor:

<pre class="brush: csharp; title: ; notranslate" title="">public class ProductsTpHConfiguration : EntityConfiguration&lt;Product&gt;
{
    public ProductsTpHConfiguration()
    {
        this
            // Indicamos clave primaria
            .HasKey(p =&gt; p.Id)
            .MapHierarchy()
            // La clase base y sus propiedades
            .Case&lt;Product&gt;(
                p =&gt; new
                {
                    ProductId = p.Id,
                    ProductName = p.Name,
                    ProductPrice = p.Price
                }
            )
            // A continuación los tipos concretos y los discriminadores
            .Case&lt;Book&gt;(
                b =&gt; new
                {
                    BookPages = b.PageNumber,
                    BookAuthor = b.Author,
                    ProductType = "B"
                })
            .Case&lt;Television&gt;(
                t =&gt; new
                {
                    TvSize = t.Size,
                    TvBrand = t.Brand,
                    ProductType = "T"
                })
            .Case&lt;Shirt&gt;(
                s =&gt; new
                {
                    ShirtColour = s.Colour,
                    ShirtSize = s.Size,
                    ProductType = "S"
                })
            .ToTable("ProductsTpH");
    }
}
</pre>

Como podéis ver, en cada **expresión Case** se mapean únicamente las propiedades declaradas por el tipo concreto y su discriminador, las propiedades comunes ya se mapean en la clase base.
  
Por otro lado, los tipos anónimos que se devuelven son los se que mapearán a las columnas de la base de datos, se ve claramente al comparar con los nombres de la imagen de la tabla de más arriba.

Independientemente de si usáramos el diseñador, necesitamos primero la clase que hereda de **ObjectContext** y que nos crea la instancia a la que añadimos los objetos de ese tipo:

<pre class="brush: csharp; title: ; notranslate" title="">public class ModelContext : ObjectContext
{
    public ModelContext(EntityConnection connection)
        : base(connection)
    {
        DefaultContainerName = "ModelContext";
    }

    public IObjectSet&lt;Product&gt; Products
    {
        get { return base.CreateObjectSet&lt;Product&gt;(); }
    }
}

</pre>

A continuación mostramos un ejemplo en que añadimos los 3 tipos en el mismo **ObjectSet**:

<pre class="brush: csharp; title: ; notranslate" title="">public void AddProducts()
{
    // crear la conexión a la BBDD y crear el contexto de EF
    SqlConnection conn = new SqlConnection("Data Source=(local);Initial Catalog=Serrate.CodeOnly;Integrated Security=True;MultipleActiveResultSets=True;");
    var builder = new ContextBuilder&lt;ModelContext&gt;();
    // la clase de config de los mapeos
    builder.Configurations.Add(new ProductsTpHConfiguration());

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
    context.Products.AddObject(book);

    var tv = new Television()
    {
        Id = Guid.NewGuid(),
        Name = "Led TV",
        Price = 999.99M,
        Brand = "Samsung",
        Size = 40
    };
    context.Products.AddObject(tv);

    var shirt = new Shirt()
    {
        Id = Guid.NewGuid(),
        Name = "Kamtxatka Shirt",
        Price = 20.50M,
        Colour = 3,
        Size = "XL"
    };
    context.Products.AddObject(shirt);
    // guardamos los cambios
    context.SaveChanges();
}

</pre>