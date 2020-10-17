# Conjunto de elementos, cada uno con un conjunto de estados que aporta un valor específico a un total.

En esta aplicación se tiene como entrada un conjunto de elementos <img src="https://render.githubusercontent.com/render/math?math=E=\{e_1, e_2, ..., e_n\}">. Cada uno de estos elementos puede tomar alguno de <img src="https://render.githubusercontent.com/render/math?math=S_i=\{s_{i_1}, s_{i_2}, ..., s_{i_{m_i}}\}"> estados y cada uno de estos estados aporta un valor <img src="https://render.githubusercontent.com/render/math?math=V_i=\{v_{i_1}, v_{i_2}, ..., v_{i_{m_i}}\}"> a un total.

La construcción de una solución requiere seleccionar un estado para cada uno de los elementos con el objetivo de minimizar, maximizar, contar, etc el valor total de la construcción.

Observa que esta aplicación incluye también aquellos problemas en dónde se pueden tomar algunos elementos y otros no. Simplemente cada elemento tiene un estado extra que representa el *no aporto valor* (que sería equivalente a no estoy presente en la solución).

## Solución mediante búsqueda
Una solución mediante búsqueda intentaría construir todas las opciones posibles, es decir, todas los estados posibles para el elemento **1** con todos los estados posibles para el elemento **2** con todos los estados posibles del elemento **3**, etc.  Por ejemplo, si cada elemento tuviera sólo dos estados válidos (estar o no estar en la construcción) y hubiera **n** elementos, la cantidad de construcciones posibles sería de <img src="https://render.githubusercontent.com/render/math?math=2^n">. Para el caso general, en el que cada elemento puede tener un número distinto de estados posible la cantidad de construcciones sería igual a <img src="https://render.githubusercontent.com/render/math?math=m_1 * m_2 * m_3 * ... * m_n"> donde <img src="https://render.githubusercontent.com/render/math?math=m_i"> es el número de estados distintos del elemento *i*.

## ¿Cuándo es posible usar la optimización de programación dinámica? 
La **PISTA** para saber que el problema puede optimizarse mediante programación dinámica es observar si la cantidad de valores distintos que puede tomar la construcción es mucho más pequeño que la cantidad de construcciones.  Por ejemplo, si se tienen <img src="https://render.githubusercontent.com/render/math?math=2^n"> construcciones pero la cantidad de valores que puede tomar cada construcción es proporcional a *n*, es muy probable que el problema se pueda resolver mediante esta aplicación de DP.

Lo anterior sucede normalmente porque el aporte de cada estado de un elemento al valor total se hace de forma aditiva, mientras que el número de construcciones distintas con cada elemento aumenta multiplicativamente.

##  Aplicación de DP

Sea <img src="https://render.githubusercontent.com/render/math?math=f(i, v)"> la optimización habiendo procesado los primeros *i* elementos y teniendo un valor total de *v*. 

Dado que el elemento *i* tiene que estar en uno de sus estados permitidos, la única forma de llegar al valor *v* con el elemento *i* es desde algún valor al que se llegó con los primeros *i - 1* elementos y quitando el valor que aportan los diferentes estados permitidos del elemento *i*.

Formalmente: <img src="https://render.githubusercontent.com/render/math?math=f(i, v) = optimo(f(i - 1, g'(v, v_{i_1})), f(i - 1, g'(v, v_{i_2})), ..., f(i - 1, g'(v, v_{i_{m_i}})))"> donde <img src="https://render.githubusercontent.com/render/math?math=g'(v_o, v)"> es una función que devuelve el valor que queda tras eliminar la aportación de *v* al valor <img src="https://render.githubusercontent.com/render/math?math=v_o">.

## Ejemplos

### Problema de la mochila (versión 0 - 1)
Se tiene una mochila con volumen *V* y se tiene una serie de objetos, cada uno ocupa un volumen <img src="https://render.githubusercontent.com/render/math?math=v_i"> y tiene un costo <img src="https://render.githubusercontent.com/render/math?math=c_i">.  El objetivo es llenar la mochila con el mayor valor posible.

* ¿Cuáles son los elementos?: Los objetos
* ¿Cuáles son los estados permitidos para cada elemento?: {Dentro de la mochila, Fuera de la mochila}
* ¿Cuáles son los valores para cada estado de un elemento?: Para el estado **Dentro** el valor que aporta es igual al volumen que ocupa.  Para el estado **Fuera** el valor que aporta es cero.
* ¿Qué se desea optimizar?: El precio máximo para cada posible volumen.

Dado que el aporte de valor es una suma aritmética, entonces el eliminar el aporte de valor de un elemento es simplemente una resta, de modo que el código queda de la siguiente forma:
```format:c++
  for(int i = 1; i <= n; ++i){   // Recorre todos los elementos
    for(int v = 1; i <= V; ++v){ // Recorre todos los valores (en este caso volúmenes) distintos
      f[i][v] = f[i - 1][v];     // Si va a ser **Fuera** no aporta valor y se toma el óptimo de los elementos previos
      if (v - volumen[i] >= 0)
        f[i][v] = max(f[i][v], f[i - 1][v - volumen[i]] + precio[i]); // Revisa la opción de que este **Dentro** de la mochila
    }
  }
```

### Wedding shopping (https://onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=2445)
Se desea comprar un atuendo para una boda.  Hay *C* tipos de prenda (zapatos, camisa, corbata, etc) y se debe comprar uno de cada tipo.  La tiene tiene varias opciones para cada prenda (varios zapatos, camisas, etc), por supuesto cada modelo de prenda tiene un valor distinto. Por último se cuenta con un presupuesto máximo a invertir. El objetivo del problema es encontrar saber si será posible completar el atuendo y en caso de que sí, cuál es el precio máximo que puedes gastar en un atuendo completo dentro del presupuesto.

* ¿Cuáles son los elementos?: Las prendas
* ¿Cuáles son los estados permitidos para cada elemento?: Los modelos de cada prenda
* ¿Cuál es el valor que aporta cada estado (modelo)?: Su costo
* ¿Cuáles es el universo permitido de valores?: El presupuesto
* ¿Qué se desea optimizar?: En este probema no deseas optimizar, sólo saber si es posible lograr un atuendo completo con un valor específico.

En este problema no quieres optimizar un parámetro, sólo saber si es posible llegar a un valor dado con un número de elementos.  De modo que el código queda como sigue:

```
  for(int i = 1; i <= C; ++i){  // Recorre todos los elementos (prendas)
    for(int j = 1; j <= presupuesto; ++j){  // Recorre todos los valores (presupuesto)
      for(int k = 1; k <= modelos[i]; ++k){ // Revisa todos los modelos de esa prenda
        // Si con los elementos previos se podía llegar al presupuesto "j - precio[i][k]" entonces llegar a este estado es posible 
        if (precio[i][k] <= j) dp[i][j] = dp[i - 1][j - precio[i][k]];  
      }
    }
  }
```

## Complejidad en tiempo y memoria
La solución dinámica revisa todos los estados de los elementos y para cada uno de ellos revisa todos los valores posibles, de modo que la complejidad total es:

<img src="https://render.githubusercontent.com/render/math?math=O(R \times \sum_{i=1}^{n}e_i)">

donde *R* es el tamaño de posibles valores y <img src="https://render.githubusercontent.com/render/math?math=e_i"> es la cantidad de estados del elemento *i*

La complejidad en memoria es de *O(RN)*, aunque dado que para procesar un elemento sólo es necesario saber los resultados hasta el elemento anterior, si se desea ahorrar espacio se pueden tener 2 columnas de la tabla únicamente y el espacio sería de complejidad *O(R)*.

## Preguntas que se pueden contestar/optimizar
* ¿Es posible llegar a un valor específico?: Se contesta con una tabla de booleanos marcando los valores a los que va siendo posible llegar.
* ¿Cuál es el mínimo/máximo de elementos/costo que se puede lograr en un valor específico?: Se contesta manteniendo el mínimo/máximo de un parámetro para cada valor específico.
* ¿Cuál es la cantidad de formas de llegar a un valor específico?: Se contesta acumulando todas las veces que se llega a un mismo valor.

## Ejercicios
* Dreamoon and WiFi (https://codeforces.com/problemset/problem/476/B)
