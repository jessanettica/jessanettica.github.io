---
layout: post
title:  "Elevación de las variables"
date:   2017-04-15 07:22:28 -0800
categories: javascript
image_sliders:
  - slider6
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
---

{% include slider.html selector="slider6" %}

Es super tentador ver código escrito en JavaScript y asumir que ejecuta de arriba a abajo. Eso es por la mayor parte correcto, pero no lo es siempre. Ese "por la mayor parte" puede resultar en confusos e inesperados resultados en tu programa. Es fácil ignorar estas inconsistencias en código en JavaScript y pensar que se deben a la flexibilidad del lenguaje, pero en realidad están pasando algunas cosas interesantes. Una orden de ejecución inesperada en tu programa probablemente se debe a algo llamado "hoisting" o elevación de las variables.

{% highlight javascript %}
cosa = 4;
var cosa;
console.log( cosa );
{% endhighlight %}

Que piensas que eso va a imprimir?

La mayoría de las personas diría `undefined` pero el programa va a imprimir `4`. Porque?

Porque JavaScript separa `var case = 4` en `var cosa` y `cosa=4`. Así que piensa en el en orden en que se lee el programa así:
{% highlight javascript %}
var cosa;
cosa = 4;
console.log(cosa);
{% endhighlight %}

Que tal aqui:

{% highlight javascript %}
console.log(cosa);
var cosa = 4;
{% endhighlight %}

`4`? No. Va a imprimir `undefined`. La ejecución ocurre en este orden:

{% highlight javascript %}
var cosa;
console.log(cosa);
cosa = 4;
{% endhighlight %}

Las declaraciones de variables y funciones son elevadas desde donde están hasta lo más alto de su contexto. Esto es importante. La elevación de variables es por contexto. Entonces, si el código en los ejemplos que acabamos de ver estuviera dentro de una función, las variables serían elevadas hasta lo más alto de el cuerpo de la function, pero no serían elevadas hasta lo más alto del archivo donde escribiste tu programa. También es importante tener en cuenta que solo las declaraciones de variables son elevadas, pero la demas lógica se queda donde está. Si la elevación de variables cambiará el orden de todo sería un caos absoluto y todos nos estuviéramos tirando el cabello.

Bueno, ahora que ya hablamos de las variables hablemos de las funciones. Las declaraciones de funciones son elevadas, pero las expresiones de funciones no.

{% highlight javascript %}
diHola();
function diHola() {
    console.log( saludo ); // undefined
    var saludo = "hi";
}
{% endhighlight %}

Ese codigo ya a ejecutar en esta orden:

{% highlight javascript %}
function diHola() {
    var saludo;
    console.log( saludo ); // undefined porque todavía no le hemos dado un valor
    saludo = "hi";
}
diHola();
{% endhighlight %}

Pero mira lo que pasa cuando es una expresión de función y no una declaración:

{% highlight javascript %}
diHola(); // resulta en un TypeError
var diHola = function printSaludo() {
    // ...
};
{% endhighlight %}

`diHola` es elevada y atada a el contexto exterior(probablemente aquí se trata de el contexto global), entonces `diHola` no causa un `ReferenceError`. Pero `diHola` todavia no tiene ningún valor, por eso cuando llamas la funcion `diHola()`, esta llamando un valor indefinido y resulta en un `TypeError`.

Que tal si tienes funciones y variables? Cual se eleva primero? Funciones son elevadas primero, y variables después.

{% highlight javascript %}
diHola();
var diHola;
function diHola() {
    console.log( "hola" );
}
diHola = function() {
    console.log( "como estas?" );
};
{% endhighlight %}

Este codigo imprime "hola". Esto es lo que pasó:

{% highlight javascript %}
function diHola() {
    console.log( "hola");
}
diHola(); // imprime “hola”
diHola = function() {
    console.log( "como estas?" );
};
{% endhighlight %}

`var diHola` fue ignorada a pesar de estar escrito antes de function `diHola()` porque las declaraciones de funciones son elevadas antes de las variables normales.

Pero cuidado porque aunque declaraciones duplicadas de `var` como las de arriba son básicamente ignoradas, declaraciones de funciones si se anulan la una a la otra. Mira esta pesadilla:

{% highlight javascript %}
diHola();
function diHola() {
    console.log( "hi");
}
var diHola = function() {
    console.log( "hello there" );
};
function diHola() {
    console.log( "Esto se imprime porque anula la declaración previa de la función" );
}
{% endhighlight %}

Que se imprime?

"Esto se imprime porque anula la declaración previa de la función"

Básicamente, hazte un favor y no hagas dupliques de funciones en el mismo contexto. Espero que esto te ayude y te ahorre un dolor de cabeza :)

👋
