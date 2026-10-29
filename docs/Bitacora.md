Decisiones de diseño — Semana 5
1. Dos implementaciones del mismo contrato
   `RepositorioArreglo` y `RepositorioEnlazado` implementan la interfaz `Repositorio`.
   El código que usa un `Repositorio` no depende de cuál implementación hay debajo.
   `RepositorioArreglo` no se modifica: se conserva como referencia de comparación.
2. Inserción al inicio
   Se prefiere la lista enlazada cuando predominan las inserciones al inicio.
   En el arreglo hay que desplazar los elementos; en la lista solo se cambian enlaces.

| Lecturas | Arreglo (movimientos) | Lista (saltos) |
|---:|---:|---:|
| 10.000 | 49.995.000 | 0 |
| 50.000 | 1.249.975.000 | 0 |
| 100.000 | 4.999.950.000 | 0 |

3. Acceso por posición
   Se prefiere el arreglo cuando predominan las consultas por posición.
   El arreglo accede directamente al índice; la lista debe recorrer desde `primero`.

| Lecturas | Arreglo (accesos) | Lista (saltos) |
|---:|---:|---:|
| 1.000 | 1.000 | 499.500 |
| 10.000 | 10.000 | 49.995.000 |
| 50.000 | 50.000 | 1.249.975.000 |

4. Recorrido de la lista
   Para recorrer una lista se avanza con un nodo `actual` y `getSiguiente()`.
   No se usa un ciclo que llame a `obtener(i)` por cada índice, porque cada llamada vuelve a empezar desde `primero` y el costo crece como O(n²).
5. Búsqueda binaria sobre una lista enlazada
   La búsqueda binaria no se utiliza sobre `RepositorioEnlazado`.
   Con 50.000 lecturas, la búsqueda lineal hizo 50.000 saltos y la binaria 749.990.
   Reducir comparaciones no reduce el trabajo cuando llegar a cada posición es costoso.
   La estructura de datos y el algoritmo se eligen en conjunto.
6. Eliminación en la lista
   `eliminar()` enlaza el nodo anterior con el siguiente del nodo eliminado, sin cortar la cadena.
   Actualiza `primero` y `ultimo` en los casos límite, y decrementa `cantidad` una sola vez.
   `contarRecorriendo()` se usa para comprobar que `tamano()` coincide con los nodos reales.
7. Historial navegable
   `HistorialNavegable` usa una lista doblemente enlazada para avanzar y retroceder sin volver al inicio.
   El costo es un enlace adicional por nodo y mantener ambos enlaces al agregar.
   Actualmente solo permite agregar; si se necesitara eliminar, habría que actualizar `siguiente` y `anterior` de los nodos vecinos.
8. Pruebas y medición
   Los experimentos de `BancoDeListas` miden rendimiento; no sustituyen las pruebas unitarias.
   Se ejecutaron 9 pruebas de `RepositorioEnlazadoTest` y 6 de `HistorialNavegableTest`, y todas pasaron.
   La comparación principal usa contadores (movimientos y saltos). Los milisegundos se conservan como evidencia experimental, porque dependen del equipo.
9. Evolución del proyecto
   La Semana 5 agrega una segunda forma de almacenar las mismas lecturas:

```
Sensores
   ↓
Ingesta
   ↓
Repositorio (arreglo o lista enlazada)
   ↓
Búsqueda / Navegación
   ↓
Medición de eficiencia
```

No existe una estructura que sea la mejor para todas las operaciones: cada una reparte el costo de forma distinta.




Preguntas para la Bitácora
1. ¿Por qué insertar al inicio es más económico en una lista enlazada que en un arreglo?En el arreglo: Como los elementos van pegaditos y ordenados por índices, para meter uno nuevo en la posición 0 te toca mover todo lo que ya estaba un espacio a la derecha, y eso te sale por $O(n)$. Y peor aún: si el arreglo ya está lleno, te toca agrandarlo y copiar todo de cero.En la lista enlazada: Aquí no hay que mover nada de sitio. Con agregarAlInicio() solo creas el nodo, lo enganchas con el que antes era el primero (nuevo.setSiguiente(primero)) y actualizas quién es el primero. Son unos pocos enlaces y ya, sin importar si la lista tiene 5 o 100.000 elementos, así que siempre es $O(1)$.En la práctica: Con 100.000 lecturas, por ejemplo, el arreglo se pone a hacer casi 5.000 millones de movimientos, mientras que la lista no tiene que recorrer absolutamente nada.
2. ¿Por qué el acceso a una posición arbitraria es más costoso en una lista?En el arreglo: Como la memoria es contigua, calcula la dirección exacta al instante y te da el dato en $O(1)$.En la lista: Los nodos están regados en la memoria y solo se salvan por los enlaces (siguiente). Si quieres llegar a la posición $i$, te toca arrancar obligatoriamente desde el primero y dar $i$ saltos con getSiguiente(). En el peor de los casos, te tragas un $O(n)$.El peligro: Esto se vuelve una pesadilla si te da por recorrer la lista con un ciclo que llama a obtener(i) en cada vuelta, porque en cada iteración la lista vuelve a empezar desde el inicio y terminas disparando el tiempo a $O(n^2)$ (unos 1.250 millones de saltos para 50.000 lecturas). La forma correcta es barrerla con un actual = actual.getSiguiente() mientras no sea nula, pasando por cada nodo una sola vez.
3. ¿Qué diferencia existe entre un experimento de rendimiento y una prueba unitaria?
   El experimento de rendimiento (como el que hacemos con el BancoDeListas) mide el músculo de las operaciones: cuánto tiempo tardan, cuántos movimientos o saltos hacen, y cómo se comportan al cambiar de tamaño. Te da la tendencia real, pero no te dice si el código está bueno o malo. Además, los números cambian dependiendo del computador, la JVM o qué más tengas abierto.
   La prueba unitaria es la que le hace control de calidad al código. Le tiras un caso conocido y validas con aserciones (assertEquals, assertSame) que devuelva exactamente lo que esperas. Es automática, se repite cuantas veces quieras y te canta la falta si hay un fallo (por ejemplo, si eliminar() no devuelve false con un índice chueco).
   En resumen: Se complementan, pero una no reemplaza a la otra. Una estructura puede volar en velocidad y aun así estar mal implementada por dentro.
4. ¿Qué caso límite consideras más importante probar en eliminar() y por qué?
   Para mí, el caso más crítico es eliminar un nodo del medio, porque es donde te puedes equivocar feo y armar un desastre silencioso. Si al puentear el nodo (anterior.setSiguiente(siguiente)) te equivocas y pones anterior.setSiguiente(null), cortas la cadena y mandas al olvido todo lo que venía después.
   Lo peor: Como el programa no bota una excepción, te quedas con una lista truncada donde el tamaño ya no pega con los nodos reales. Si no tienes una prueba que revise los elementos que siguen y compare el tamaño real con un conteo nodo a nodo, ese error pasa desapercibido.
   (Alternativa válida) Si prefieres irte por otro lado, eliminar el único nodo o el último también es pesado porque te obliga a actualizar bien primero y ultimo. Si descuidas el puntero de ultimo, la próxima inserción va a reventar el orden porque se enganchará desde un nodo que ya ni vive en la lista.
5. Para el histórico de lecturas, ¿qué estructura elegirías según predominen las inserciones al inicio o las consultas por posición?Si mandan las inserciones al inicio: Me voy de cabeza por la lista enlazada. Es $O(1)$, te evitas mover medio mundo y no tienes que andar redimensionando arreglos. Le viene como anillo al dedo a un histórico donde lo último que pasa es lo primero que se muestra. Lo único es que no puedes usar obtener(i) para leerla, sino recorrerla de corrido nodo por nodo.Si mandan las consultas por posición: Ahí sí prefiero el arreglo. Tener acceso directo por índice en $O(1)$ es una gran ventaja, y además te abre la puerta a usar algoritmos como la búsqueda binaria (que necesitan brincar al medio al instante). Intentar hacer eso en una lista sería eterno porque te obligaría a recalcular todo a punta de saltos.