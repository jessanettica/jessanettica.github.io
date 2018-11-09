---
layout: post
title:  "Recreando Pinterest con Gatos"
date:   2017-07-19 08:20:13 -0800
tags: code
image_sliders:
  - slider10
style: |
  .post-title {
    font-family: 'Playfair Display', serif;
    font-size: 45px;
  }
  .post-content p {
    font-family: 'Pontano Sans', sans-serif;
    font-size: 20px;
  }
  pre, code {
    background-color: #ffd7d7;
  }
photos: ["/img/party_frenchie.jpg", "/img/rainbow_cake.jpg", "/img/dog_at_party.jpg"]
---

{% include slider.html selector="slider10" %}

No estoy segura si Pinterest fue el primer sitio web que popularizó el diseño de una cuadrícula de imágenes en cascada, pero ahora lo veo en todas partes. Si nunca has utilizado Pinterest, es un sitio donde puedes guardar colecciones de imágenes que se organizan en una cuadrícula.

Hay un par de librerías a tu disposición para crear un diseño de cuadrícula que se encargaran del posicionamiento para ti (como [Masonry][masonry]), pero la implementación de un diseño como el de Pinterest no es tan dificil. ¡Puedes hacerlo tu misma!

Voy a usar jQuery para explicar cómo hacerlo, pero tu puedes usar lo que quieras. Lo que importa es la matemática detrás del posicionamiento, no la biblioteca que utilices. Te daré una explicación general y luego podremos entrar en los detalles de implementación con código.

Aquí va la explicación general. Los “pins” se muestran en la página inicialmente con AJAX. Después de asignar lo ancho de la columna y el margen entre las columnas, puedes utilizar lo ancho de la ventana para calcular el número de columnas que caben en la página. Utiliza una lista para guardar la altura de cada columna y la longitud de la lista es cuántas columnas hay. Por ejemplo, `misColumnas = [300, 340, 370]` significa que hay 3 columnas en la página. La altura de la primera columna es de 300, la altura de la segunda columna es de 340 y la altura de la tercer columna es de 370. Cool, ¿no? Así que cuando cada pin se agrega a la página, se agrega a la columna más corta y la matriz se actualiza con la nueva altura de la columna. La posición izquierda y superior de cada uno de los “pins”, los cuales tienen una posición absoluta, se determina usando estos valores de altura (ve `getTopPosition ()` y `getLeftPosition ()` a continuación).

¡Eso es! Probablemente es un poco confuso leer sólo esa explicación, así que echemos un vistazo al código para ver cómo se implementa. Añadi comentarios con números para que puedas leer el archivo en el orden en que suceden las cosas que mencione en la explicación anterior. Los nombres de las variables y el resto del codigo lo voy a dejar en inglés, pero todos mis comentarios explicando lo que está pasando en el código están en espanol:

{% highlight javascript %}
function makeArray(len, value) {
  return new Array(len).fill(value);
}

function getShortestColumnIndex(colArray) {
  // Regresa qué columna es la más corta para que
  // sepamos dónde colocar el siguiente pin
  return colArray.indexOf(Math.min(...colArray));
}

function getLeftPosition(colIndex, numColumns) {
  // Calcula la posición izquierda del pin tomando
  // en cuenta su anchura y en qué columna está
  var gutter = 12;
  var pinWidth = 236;
  if (colIndex == 0){
    return gutter;
  }else{
    return gutter * (colIndex + 1) + pinWidth * colIndex;
  }
}

function getTopPosition(colIndex, colArray){
    // Obten la posición de la parte superior del pin usando
    // la altura de la columna en la que se colocará
    // usando la lista que tenemos
    return colArray[colIndex];
}

function updateColArray(colIndex, colArray, pinHeight){
  // actualiza la lista que representa las columnas agregando la altura del pin
  // más el espacio vertical que queremos entre los pins
  colArray[colIndex] += (pinHeight + 120);
  return colArray;
}
var screenWidth = $(window).width();
var gutter = 12;
var numColumns = Math.floor(screenWidth / (236 + gutter));
// 2. Haz una lista para representar las columnas y
// inicializa la altura de todas las columnas en la página a cero
var colArray = makeArray(numColumns, 0);

function renderPins(resp){
  // 3. Crea elementos de DOM de los datos de la respuesta
  Object.values(resp["data"]).map(function(v) {
      return $(`<div data-height="${v.img_height}" style="padding:15px;position:absolute;">
        <img style="border-radius:10px;" width=236 src="${v.img_url}">
        <div style="min-height:13px;width: 236px;position:relative;padding-left:8px;padding-right:8px;">
            <div style="position:relative;max-width:145px;">
                <p style="max-width: 180px;margin-top:0;padding:0;max-height:60px;overflow:hidden;text-overflow:ellipsis;">${v.description}
                </p>
            </div>
            <div style="position:absolute;right:0;top:1px;z-index:3;">
                <p style="color:#a7a7a7">${v.repin_count} repins
                </p>
            </div>
        </div>
        <div style="display:flex;position:relative;-webkit-box-align:center;padding-left:8px;padding-right:8px;margin-top:4px;"
        >
            <div style="padding:0;display:flex;">
                <a href="pinterest.com/${v.pinner.username}" page style="-webkit-box-align: center;text-decoration:none">
                    <div style="height:24px;width:24px;margin-right:8px;">
                        <img src="${v.pinner.img_url}" style="height:24px;width:24px;border-radius:50%;position:static;">
                    </div>
                    <div>
                        <div style="display:block;overflow:hidden; text-overflow:ellipsis;">
                    ${v.pinner.username}</div>
                        <div style="display:block;overflow:hidden; text-overflow:ellipsis;">
                    ${v.boards.name}</div>
                    </div>
                </a>
             </div>
        </div>
      </div>`)
  }).forEach(function(v) {
      // Obten la altura de cada pin
      var imageHeight = v.data("height");
      var pinHeight = imageHeight + 24 + 24;
      // 4. Obten el índice de la columna más corta. La columna más corta
      // es donde se colocará el siguiente pin.
      var columnIdx = getShortestColumnIndex(colArray);
      var left = getLeftPosition(columnIdx, numColumns);
      var top = getTopPosition(columnIdx, colArray);
      // 5. Actualiza la lista que representa nuestras columnas para que
      // podemos continuar poniendo los siguientes "pins" en
      // la columna más corta
      colArray = updateColArray(columnIdx, colArray, pinHeight);
      // 6. Añade la posición al pin
      v.css("top", top.toString() + "px");
      v.css("left", left.toString() + "px");
      v.css("width", "236px");
      v.css("height", "100%");
      // 7. Añade el pin a la página
      $('#pin-container').append(v);
  });
}
function fetchAllInitialPins(){
  $.ajax({
      url: '/pins/get_all_pins/',
      type: 'POST',
      data: {},
      success: function(resp) {
          if (resp.status == 'ok'){
              renderPins(resp);
          }
      }
   });
}

window.onload = function () {
  // 1. Llama a las funciones al cargarse la página
  fetchAllInitialPins();
};

{% endhighlight %}

Si quieres ver la implementación del proyecto entero esta [aquí][recreando-pinterest].

Jessica 👋

[masonry]: https://masonry.desandro.com/
[recreando-pinterest]: https://github.com/jessanettica/Recreate-Pinterest-with-cats
