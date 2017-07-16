---
title: 'Entity Framework 4 CodeOnly - Table per concrete Class (TPC)'
date: 2010-08-04T18:40:12+00:00
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

  {% img /uploads/2010/08/TablaTpC.png 450 259 '"TablaTpC"' '"TablaTpC"' %}

Ahora vamos a definir los mapeos de la base de datos con nuestras clases.  

Para el subtipo **Book**:

{% codeblock lang:csharp %}
public class BooksTpCConfiguration : EntityConfiguration<Book>
{
    public BooksTpCConfiguration()
    {
        this
            .HasKey(b => b.Id)
            .MapSingleType(
                b => new
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
{% endcodeblock %}

<!--more-->Para el subtipo 

**Television**:

{% codeblock lang:csharp %}
public class TelevisionsTpCConfiguration : EntityConfiguration<Television>
{
    public TelevisionsTpCConfiguration()
    {
        this
            .HasKey(t => t.Id)
            .MapSingleType(
                t => new
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
{% endcodeblock %}

 Para el subtipo **Shirt**:

{% codeblock lang:csharp %}
public class ShirtsTpCConfiguration : EntityConfiguration<Shirt>
{
    public ShirtsTpCConfiguration()
    {
        this
            .HasKey(s => s.Id)
            .MapSingleType(
                s => new
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
{% endcodeblock %}

Vemos que mediante la expresión **MapSingleType** cada clase mapea las **propiedades propias del subtipo y de la clase base.**

Para crear las instancias a las que añadimos nuestros objetos utilizaremos una clase que herede de **ObjectContext.** En ella definimos sólo los subtipos:

{% codeblock lang:csharp %}
public class ModelTpCContext : ObjectContext
{
    public ModelTpCContext(EntityConnection connection)
        : base(connection)
    {
        DefaultContainerName = "ModelTpCContext";
    }

    public IObjectSet<Book> Books
    {
        get { return base.CreateObjectSet<Book>(); }
    }

    public IObjectSet<Television> Televisions
    {
        get { return base.CreateObjectSet<Television>(); }
    }

    public IObjectSet<Shirt> Shirts
    {
        get { return base.CreateObjectSet<Shirt>(); }
    }
}
{% endcodeblock %}

 Y para terminar con el ejemplo, vemos el método que configura y persiste los 3 subtipos tratados:

{% codeblock lang:csharp %}
public void AddProducts()
{
    // crear la conexión a la BBDD y crear el contexto de EF
    SqlConnection conn = new SqlConnection("Data Source=(local);Initial Catalog=Serrate.CodeOnly;Integrated Security=True;MultipleActiveResultSets=True;");
    var builder = new ContextBuilder<ModelTpCContext>();
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
{% endcodeblock %}