---
title: Entity Framework 4 CodeOnly – Table per Type (TPT)
date: 2010-07-21T00:17:03+00:00
categories:
  - Code Only
  - Entity Framework 4
tags:
  - CodeOnly
  - EF4
  - TPT
---
En el <a href="/2010/07/19/entity-framework-4-codeonly-table-per-hierarchy-tph/" target="_self">post anterior</a> vimos el tipo de herencia Table per Hierarchy (TPH).  

Ahora vamos a ver el tipo de herencia en que el tipo base se guarda en una tabla compartida, es lo que llamamos **Table per Type** (TPT).  

En el ejemplo que estamos siguiendo (<a href="/2010/07/19/entity-framework-4-codeonly-introduccion/" target="_self">intro</a>), nuestra clase base Product se guardará en la tabla ProductsTpT que tiene las propiedades de la propia clase base y luego tendremos una tabla por cada subtipo: ShirtsTpT, BooksTpT y TelevisionsTpT como podemos ver a continuación:  

<p style="text-align: center;">
  {% img /uploads/2010/07/TablaTpT.png 450 177 '"TablaTpT"' '"TablaTpT"' %}
</p>

Ahora vamos a definir los mapeos de la base de datos con nuestras clases.  

Para la **clase base** Product:  

{% codeblock lang:csharp %}
public class ProductsTpTConfiguration : EntityConfiguration<Product>
{
    public ProductsTpTConfiguration()
    {
        this

            .HasKey(p => p.Id)
            .MapHierarchy(
                p => new
                {
                    ProductId = p.Id,
                    ProductName = p.Name,
                    ProductPrice = p.Price
                }
            )
            .ToTable("ProductsTpT");
    }
}
{% endcodeblock %}

<!--more-->

Para los **subtipos**: 

{% codeblock lang:csharp %}
public class BooksTpTConfiguration : EntityConfiguration<Book>
{
    public BooksTpTConfiguration()
    {
        this
            .MapHierarchy(
                b => new
                {
                    Id = b.Id,
                    Pages = b.PageNumber,
                    Author = b.Author
                }
            )
            .ToTable("BooksTpT");
    }
}

public class TelevisionsTpTConfiguration : EntityConfiguration<Television>
{
    public TelevisionsTpTConfiguration()
    {
        this
            .MapHierarchy(
                t => new
                {
                    Id = t.Id,
                    Size = t.Size,
                    Brand = t.Brand
                }
            )
            .ToTable("TelevisionsTpT");
    }
}

public class ShirtsTpTConfiguration : EntityConfiguration<Shirt>
{
    public ShirtsTpTConfiguration()
    {
        this
            .MapHierarchy(
                s => new
                {
                    Id = s.Id,
                    Colour = s.Colour,
                    Size = s.Size
                }
            )
            .ToTable("ShirtsTpT");
    }
}
{% endcodeblock %}

Como podemos observar, en cada expresión **MapHierarchy** solo mapeamos las propiedades declaradas por el propio tipo, y las que se refieran a su clave primaria. 

Para crear las instancias a las que añadimos nuestros objetos utilizaremos una clase que herede de **ObjectContext**.  Podeis observar que es la misma que la del ejemplo de Table per Hierarchy: 

{% codeblock lang:csharp %}
public class ModelContext : ObjectContext
{
    public ModelContext(EntityConnection connection)
        : base(connection)
    {
        DefaultContainerName = "ModelContext";
    } 

    public IObjectSet<Product> Products
    {
        get { return base.CreateObjectSet<Product>(); }
    }
}
{% endcodeblock %}

Y finalmente, para terminar nuestro ejemplo y persistir los 3 tipos a las diferentes tablas lo haremos de la siguiente forma:

{% codeblock lang:csharp %}
public void AddProducts()
{
    // crear la conexión a la BBDD y crear el contexto de EF
    SqlConnection conn = new SqlConnection("Data Source=(local);Initial Catalog=Serrate.CodeOnly;Integrated Security=True;MultipleActiveResultSets=True;");
    var builder = new ContextBuilder<ModelContext>();
    // debemos añadir las 4 configuraciones que equivalen a cada tabla
    builder.Configurations.Add(new ProductsTpTConfiguration());
    builder.Configurations.Add(new BooksTpTConfiguration());
    builder.Configurations.Add(new TelevisionsTpTConfiguration());
    builder.Configurations.Add(new ShirtsTpTConfiguration());
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
{% endcodeblock %}