---
title: 'ASP.NET MVC: Múltiples botones en el mismo formulario'
date: 2011-01-21T20:01:35+00:00
categories:
  - ASP.NET MVC
tags:
  - ASP.NET MVC
---
Seguramente os habréis encontrando en ocasiones en que, dentro de un mismo formulario, necesitamos alojar varios botones de acción sobre el mismo.

Existen varias opciones de conseguir este objetivo, pero mí preferida, como amante de _client scripting_, es con **jQuery**.

A continuación mostramos la vista con un formulario definido y dos botones de _submit_:

{% codeblock lang:csharp %}
<% using (Html.BeginForm()) { %>
    <div>
        <%= Html.LabelFor(m=> m.Name) %>
        <%= Html.TextBoxFor(m => m.Name) %>
    </div>
    <div>
        <%= Html.LabelFor(m => m.Description)%>
        <%= Html.TextBoxFor(m => m.Description) %>
    </div>

    <input id="btnSave" type="submit" value="Guardar" />
    <input id="btnGetNextRow" type="submit" value="Siguiente registro" />
<% } %>
{% endcodeblock %}

Por el momento, nada nuevo. A continuación vemos como enlazamos el evento **click** a los botones:

{% codeblock %}
<script type="text/javascript">
    $(document).ready(function () {
        $("#btnSave").click(function () {
            var form = $(this).parent("form");
            form.attr('action', '<%= Url.RouteUrl(new { Controller = "Home", Action = "Save" }) %>');
            form.attr('method', 'post');
        });

        $("#btnGetNextRow").click(function () {
            var form = $(this).parent("form");
            form.attr('action', '<%= Url.RouteUrl(new { Controller = "Home", Action = "GetNextRow" }) %>');
            form.attr('method', 'get');
        });
    });
</script>
{% endcodeblock %}

Obtenemos una referencia al formulario con la instrucción: $(this).parent(“form”) y a continuación cambiamos los atributos _action_ y _method_ del formulario. De esta forma, hacemos submit a la acción que nos interesa en cada momento utilizando GET o POST según convenga.