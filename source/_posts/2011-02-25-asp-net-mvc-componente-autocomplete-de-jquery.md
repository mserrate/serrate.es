---
id: 213
title: 'ASP.NET MVC: Componente Autocomplete de jQuery'
date: 2011-02-25T14:21:17+00:00
author: Marçal
layout: post
guid: http://www.serrate.es/?p=213
permalink: /2011/02/25/asp-net-mvc-componente-autocomplete-de-jquery/
categories:
  - ASP.NET MVC
tags:
  - ASP.NET MVC
  - jQuery
---
En el artículo de hoy vamos a ver qué fácil es alimentar mediante Ajax y objetos **JSON** componentes jQuery desde **ASP.NET MVC**.

Para empezar, vamos a usar la CDN de google para importar las librerías que necesitamos: jQuery y jQuery UI más los ficheros de estilos para jQuery UI:

<pre class="brush: xml; light: true; title: ; notranslate" title="">http://ajax.googleapis.com/ajax/libs/jquery/1.5.0/jquery.min.js
http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.10/jquery-ui.min.js
http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.9/themes/base/jquery-ui.css
</pre>

Seguidamente definimos en nuestra vista un label y una caja de texto:

<pre class="brush: xml; light: true; title: ; notranslate" title="">&lt;label for="txtTitulos"&gt;Titulos: &lt;/label&gt;&lt;input type="text" id="txtTitulos" /&gt;
</pre>

El código cliente para enlazar la extensión de **jQuery** a la caja de texto es la siguiente:

<pre class="brush: jscript; title: ; notranslate" title="">&lt;script type="text/javascript"&gt;
    $(document).ready(function () {
        $("#txtTitulos").autocomplete({
            minLength: 3,
            source: getData
        });
    });

    function getData(request, response) {
        $.ajax({
            url: '/Home/GetData',
            type: 'GET',
            dataType: 'json',
            data: { words: request.term },
            success: function (data) {
                response(data);
            }
        });
    }
&lt;/script&gt;
</pre>

Como podemos ver, inicializamos la opción source con nuestro propio callback getData. En que realizamos la consulta a la action GetData del controller Home. Para más opciones y eventos del componente autocomplete mirad la documentación en la página de <a href="http://jqueryui.com/demos/autocomplete/" target="_blank">jQuery UI</a>.

Finalmente nuestra action GetData queda de la siguiente manera:

<pre class="brush: csharp; title: ; notranslate" title="">public ActionResult GetData(string words)
{
    var mySource = GetData();

    var result = from s in mySource
                    where s.Title.Text.Contains(words)
                    select s.Title.Text;

    return Json(result, JsonRequestBehavior.AllowGet);
}
</pre>

Si os fijáis en el segundo parámetro de Json, indicamos que admitimos peticiones de tipo GET. Esto se debe indicar para evitar problemas de seguridad. Para más información consultad este post de <a href="http://haacked.com/archive/2009/06/25/json-hijacking.aspx" target="_blank">Phil Haack</a>.

Los datos de ejemplo los he sacado del rss de la página de ASP.NET. Os dejo el código para que no es creáis que miento:

<pre class="brush: csharp; title: ; notranslate" title="">private IEnumerable&lt;SyndicationItem&gt; GetData()
{
    string feed = "http://www.asp.net/rss/news";

    var req = WebRequest.Create(feed);

    using (var reader = XmlReader.Create(req.GetResponse().GetResponseStream()))
    {
        var rssData = SyndicationFeed.Load(reader);
        return rssData.Items;
    }
}
</pre>