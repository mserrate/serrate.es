---
id: 196
title: 'ASP.NET MVC: Múltiples botones en el mismo formulario'
date: 2011-01-21T20:01:35+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=196
permalink: /2011/01/21/asp-net-mvc-multiples-botones-en-el-mismo-formulario/
categories:
  - ASP.NET MVC
tags:
  - ASP.NET MVC
---
Seguramente os habréis encontrando en ocasiones en que, dentro de un mismo formulario, necesitamos alojar varios botones de acción sobre el mismo.

Existen varias opciones de conseguir este objetivo, pero mí preferida, como amante de _client scripting_, es con **jQuery**.

A continuación mostramos la vista con un formulario definido y dos botones de _submit_:

<pre class="brush: xml; title: ; notranslate" title="">&lt;% using (Html.BeginForm()) { %&gt;
    &lt;div&gt;
        &lt;%= Html.LabelFor(m=&gt; m.Name) %&gt;
        &lt;%= Html.TextBoxFor(m =&gt; m.Name) %&gt;
    &lt;/div&gt;
    &lt;div&gt;
        &lt;%= Html.LabelFor(m =&gt; m.Description)%&gt;
        &lt;%= Html.TextBoxFor(m =&gt; m.Description) %&gt;
    &lt;/div&gt;

    &lt;input id="btnSave" type="submit" value="Guardar" /&gt;
    &lt;input id="btnGetNextRow" type="submit" value="Siguiente registro" /&gt;
&lt;% } %&gt;
</pre>

Por el momento, nada nuevo. A continuación vemos como enlazamos el evento **click** a los botones:

<pre class="brush: jscript; title: ; notranslate" title="">&lt;script type="text/javascript"&gt;
    $(document).ready(function () {
        $("#btnSave").click(function () {
            var form = $(this).parent("form");
            form.attr('action', '&lt;%= Url.RouteUrl(new { Controller = "Home", Action = "Save" }) %&gt;');
            form.attr('method', 'post');
        });

        $("#btnGetNextRow").click(function () {
            var form = $(this).parent("form");
            form.attr('action', '&lt;%= Url.RouteUrl(new { Controller = "Home", Action = "GetNextRow" }) %&gt;');
            form.attr('method', 'get');
        });
    });
&lt;/script&gt;
</pre>

Obtenemos una referencia al formulario con la instrucción: $(this).parent(“form”) y a continuación cambiamos los atributos _action_ y _method_ del formulario. De esta forma, hacemos submit a la acción que nos interesa en cada momento utilizando GET o POST según convenga.