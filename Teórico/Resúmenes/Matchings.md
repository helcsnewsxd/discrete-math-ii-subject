# Matchings

## Definición del problema

Dado un grafo $G$ bipartito, encontrar un matching en $G$ con la mayor cantidad de lados posibles

- Dado un grafo $G$ un _matching_ en $G$ es un subgrafo $M$ de $G$ tal que el grado de cada vértice de $M$ es uno (en $M$)
  - Es decir, es un conjunto de lados disjuntos

Un matching que tenga la mayor cantidad posible de lados se dice _matching maximal_

## Reducción del problema a flujo

1. Transformamos el grafo bipartito no dirigido en un network (el cual tendrá todas sus capacidades enteras)
    - Consideramos $G$ bipartito y $X,Y$ sus partes (de modo que todo lado conecta un $x\in X$ con un $y\in Y$)
    - Se agregan dos vértices nuevos $s,t$
    - Se consideran los siguientes lados (de capacidad $1$):
      - $\overrightarrow{xy}$ para todo $x\in X,y\in Y,xy\in E(G)$
      - $\overrightarrow{sx}$ para todo $x\in X$
      - $\overrightarrow{yt}$ para todo $y\in Y$

2. Encontramos un flujo entero maximal en ese network, que sabemos por el _teorema de la Integralidad_ que existe
    - Por ejemplo, usando FF, EK o Dinitz

3. A partir de ese flujo maximal obtenemos un matching maximal
    - Dado un flujo $f$ entero, si tomamos
      - $W=\{ x\in X:in_f(x)=1\}$
      - $U=\{ y\in Y:out_f(y)=1\}$
      - $F=\{ xy\in E(G):x\in W,y\in U:f(\overrightarrow{xy})=1\}$

      entonces $(W\cup U,F)$ es un subgrafo de $G$ que es el matching maximal, donde su cantidad de lados ( $Size(F)$ ) es igual al valor del flujo ( $v(f)$ )
    - Vale también la inversa (de matching a flujo), por lo que flujos maximales enteros se corresponden con matchings maximales

## Perfección y completitud de los matchings

Para resolver estos problemas simplemente se halla un matching maximal y se chequea la condición

### Matching Perfecto

Matching que "cubre" todos los vértices, es decir, donde el conjunto de sus vértices es igual que el de $G$

- Condición necesaria
  - $|X|=|Y|$
- **Teorema de Konig (del matrimonio)**
  - Todo grafo bipartito regular tiene un matching perfecto

### Matching completo

Matching que "cubre" todos los vértices de una de las partes del grafo, es decir, si cubre a todos los de $X$ se dice que es _completo sobre_ $X$

- Condición necesaria
  - $|X|\leq |Y|$
    - Si $|X|=|Y|$, entonces es también perfecto
- **Teorema de Hall**
  - Si $G$ es un grafo bipartito con partes $X,Y$ y $|S|\leq|\Gamma(S)|\forall S\subseteq X$, entonces existe un matching completo sobre $X$

## Matchings pesados

### Minimizando el mayor costo (Algoritmo de Gross)

- Se considera un grafo $G$ bipartito con pesos en los lados, con ambas partes con la misma cardinalidad (digamos $n$) y al menos un matching perfecto
- Se quiere encontrar de entre todos los matchings perfectos, uno que minimice el mayor peso
- El algoritmo es el siguiente:
  - Se hace búsqueda binaria sobre los costos de los lados en el grafo
  - Se calcula el matching maximal para el grafo donde se consideran solo las aristas con costo menor o igual a $edgeValue[m]$
  - Si tiene matching perfecto, hay que tomar un umbral más chico
    - Puede considerarse la mitad entre el $m$ y la cota inferior, pero una optimización (no mejora complejidad del peor caso) es considerar en vez del umbral actual, el máximo costo considerado en el matching perfecto
  - Caso contrario, si no lo tiene, hay que tomar uno más alto (el nuevo umbral es la mitad entre el $m$ y la cota superior)
- La complejidad del algoritmo de Gross es $O(n^{\frac{5}{2}}lg(n))$

### Minimizando la suma de los costos (Húngaro)

El problema que se quiere resolver ahora es el siguiente:

- Dado un grafo bipartito con partes iguales y pesos en los lados que representan costos, queremos hallar un matching perfecto que minimice la suma de estos
- Dado que un matching perfecto induce una biyección entre las filas y las columnas que representa al grafo, el problema es equivalente a lo siguiente:
  - Dada una matriz $C$, $n\times n$, encontrar una permutación $\sigma :\left\lbrace 1,2,...,n\right\rbrace\rightarrow\left\lbrace 1,2,...,n\right\rbrace$ tal que $\sum C_{i,\sigma(i)}\leq\sum C_{i,\tau(i)}$ para toda permutación $\tau :\left\lbrace 1,2,...,n\right\rbrace\rightarrow\left\lbrace 1,2,...,n\right\rbrace$

#### Preludio

- Supongamos que $C$ es una matriz de costos $C$ tal que $C_{i,j}\geq 0\forall i,j$. Sea $\sigma$ una permutación de $\left\lbrace 1,2,...,n\right\rbrace$ tal que $C_{i,\sigma(i)}=0\forall i$, entonces el matching asociado a $\sigma$ minimiza la suma de los costos
- Supongamos que $C$ es una matriz de costos $C$, $n\times n$ y sea $\tilde{C}$ una matriz que se obtiene de $C$ restándole una _constante_ a una fila o columna, entonces todo matching que minimice la suma de costos respecto de $C$ también minimiza la suma de costos respecto de $\tilde{C}$ (y viceversa)

Como consecuencia, estas propiedades sugieren el siguiente procedimiento:

1. Restar el mínimo de cada fila a cada fila (hay al menos un $0$ en cada fila)
    - Se puede restar un número negativo (i.e., sumar)
2. Restar el mínimo de cada columna a cada columna (hay al menos un $0$ en cada columna)
    - Se puede restar un número negativo (i.e., sumar)
3. Buscar un matching perfecto de $0s$ (en el grafo, dos vértices tienen aristas solo si $C_{x,y}=0$)
    - Si encontramos uno, este minimiza la suma
    - Caso contrario, se hace lo que se describe en el algoritmo
      - Si no existe matching, siempre tendremos un $S$ con $|S|\gt|\Gamma(S)|$ y sabemos que $S$ es:
        - Intersección del corte con $X$
        - En sistema de matrices, $S$ serán todas las filas etiquetadas cuando paramos

#### Descripción del algoritmo

1. Restar el mínimo de cada fila
2. Restar el mínimo de cada columna
3. Buscar matching perfecto de ceros
4. Si se encuentra, listo. Sino, habrá un $S$ con $|S|\gt |\Gamma(S)|$
    - Calcular $m=\text{ mínimo de las entradas de la matriz que estén en las filas de} S\text{ y las columnas del complemento de }\Gamma(S)$
    - Restar $m$ de las filas de $S$
    - Sumar $m$ a las columnas de $\Gamma(S)$
      - Los dos pasos anteriores se reducen a
        - Restar $m$ de $S\times\overline{\Gamma(S)}$
        - Sumar $m$ a $\overline{S}\times\Gamma(S)$
    - Continuar buscando el matching perfecto de ceros en la nueva matriz
      - Desde las filas $S$ manteniendo todas las etiquetas
    - Se puede cambiar la matriz y aún así no poder extender el matching ni en un lado (esto puede pasar a lo sumo $n$ veces)
      - Luego de un cambio de matriz, _o crece el matching o crece el_ $S$

### Algoritmos que surgen de problemas pedidos en el práctico

#### Minimizar la suma de los costos de entre todos los matchings que minimicen el mayor costo

Se aplica el algoritmo de Gross con la binaria en el array de los costos de los lados. Cuando se obtiene el resultado, se considera la matriz con la que se trabajó en ese umbral (i.e., solo con los lados menores e iguales al umbral) y allí se aplica Húngaro.
La complejidad es la suma de la complejidad de Gross y de Húngaro.

#### Minimizar el mayor costo de entre todos los matchings que minimicen la suma

Se aplica Húngaro primero y nos guardamos el valor de la menor suma posible a obtener.
Se aplica el algoritmo de Gross con la binaria en el array de los costos de los lados. En caso que se obtenga un matching perfecto con el umbral seleccionado, no vamos a considerar "que está bien" sino que eso significa que a la matriz se le va a aplicar Húngaro. En caso que el valor de la suma de lo mismo que la mínima obtenida al principio, se considera que está "todo OK" y se recursiona a la mitad inferior. Caso contrario, no es lo que se busca por lo que se recursiona a la mitad superior.
La complejidad es la complejidad de Gross más la de Húngaro multiplicada por $log_2(N)$ (donde $N$ es la cantidad de umbrales posibles)

#### Buscar el máximo en los matchings (sea de suma o de costo individual)

- Si lo que se busca es la suma, hay que aplicar Húngaro pero con la matriz de costes invertidos (i.e., multiplicados todos por $-1$)
- Si lo que se busca es el máximo costo (individual), hay que aplicar Gross y usar el umbral pero en vez de considerar la matriz con las aristas de coste $\le\text{umbral}$, vamos a considerar las de coste $\ge\text{umbral}$. Luego, tenemos una búsqueda binaria de la forma _True, True, True, False, False, ..._, por lo que en vez de quedarnos con la variable $end$, nos vamos a quedar con $start$
