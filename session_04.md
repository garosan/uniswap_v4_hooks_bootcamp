# Sesión 04 – Matemáticas de Uniswap: Ticks, Q64.96 y Precios

Vamos a ver 2 conceptos fundamentales para construir nuestros propios hooks:

- Ticks
- Q64.96

### Curvas de Precio Discretas

- Uniswap v2 es un CFMM simple, LPs siempre agregan liquidez en todo el rango de precios
- Usa la fórmula `x * y = k` donde `k` es una constante.
- En otras palabras, sin importar cuánto intercambies, nunca agotarás completamente un token en el pool.
- Esto produce una curva de precios continua e infinita que nunca toca los ejes.

![Imagen tomada de Atrium Academy](./assets/04_curva_continua.png)

En conclusión: La curva de precio en Uniswap v2 (imagen arriba) nunca toca los ejes. Es una curva continua, infinita.

- A partir de Uniswap v3 esto cambia. Uniswap v3 introduce la **liquidez concentrada**.
- Ahora los LPs pueden proveer liquidez en un rango específico y la ecuación `x * y = k` ya no es suficiente para determinar ciertos cálculos que en este momento no son muy relevantes.
- Lo importante es que al tener liquidez concentrada, la curva ahora se ve así:

![Imagen tomada de Atrium Academy](./assets/04_curva_discreta.png)

- Ahora tenemos una curva finita, discreta.
- Si la curva ahora es finita, puede dividirse en un número finito de _secciones_. Estas secciones dan origen a los conceptos de _ticks_ y _tick spacing_ (espacio entre ticks).

### Distribución de Liquidez Concentrada

Para recapitular, en Uniswap v2 la liquidez se ve así, ya que todos los LPs tienen qué poner la liquidez en todo el rango:

![Imagen tomada de Atrium Academy](./assets/04_liquidez_v2.png)

Sin embargo, en v3, la distribución de la liquidez se vería así:

![Imagen tomada de Atrium Academy](./assets/04_liquidez_v3.png)

Esto quiere decir 2 cosas:

- Estas _áreas de liquidez_ pueden solaparse entre sí.
- Las _áreas de liquidez_ podrían nunca tocarse.

En la vida real esto se ve más o menos así:

![Imagen tomada de Atrium Academy](./assets/04_eth_usdc_pool.png)

La liquidez generalmente se agrupa alrededor del precio actual, y va disminuyendo conforme te alejas de ese precio.

### Ticks

Regresemos a esta imagen:

![Imagen tomada de Atrium Academy](./assets/04_liquidez_v3.png)

Nota los números del `-9` a `+9`. Estos son los _ticks_.

Un _tick_ entonces es un punto específico en la curva donde un trade puede ejecutarse. Cada tick, es un precio específico para el token. Nota también que todos los ticks están separados por la misma distancia (en este ejemplo, esa distancia es 1).

Conclusión #1 👉: Los ticks dividen la curva finita de precios en puntos discretos, equidistantes. Cada tick representa un precio específico en el que puede ocurrir un trade.

➡️ Sobre tick spacing

Ahora veamos esta imagen:

![Imagen tomada de Atrium Academy](./assets/04_tick_moving.png)

Supongamos que `1` representa el precio actual en el pool.

- Cuando se hace un trade, el precio solo puede moverse hacia 0 o hacia 2 (los trades solo pueden ocurrir en los ticks)
- Todos los ticks deben ser enteros, pero puede pasar que según la fórmula, el resultado de un swap debería colocar el próximo precio en `4.5`, lo que haremos en ese caso es redondear al tick más cercano disponible.
- Tomemos en cuenta que estos enteros son representaciones, aunque la curva es finita, hay millones de puntos o ticks en ella, por lo que estas variaciones son mínimas.

Conclusión #2 👉: El espacio entre 2 ticks adyacentes _tick spacing_, es el movimiento de precio más pequeño posible para un par de tokens en un pool.

### Ticks y Precios

Mira esta ecuación, determina el precio relativo de un token en un tick específico:

![Imagen tomada de Atrium Academy](./assets/04_p_de_i_ecuacion.png)

¿Cómo? Un ejemplo:

Asumamos un pool con tokens `A` y `B`.

Tick 0: 1 Token A = 1 Token B

Tick 10: 1 Token A = 1.001 Token B

Tick -10: 1 Token A = 0.999 Token B

¿Por qué 1.0001? Porque implica que cada tick representa un cambio del 0.01% (1 basis point) en el precio relativo.

Hagamos un recap rápido de lo que hemos cubierto:

1. Los _ticks_ son puntos discretos en una curva de precio finita, cada tick representa un precio relativo en el que puede ocurrir un trade.
2. _Tick spacing_ es el espacio entre 2 ticks adyacentes, y el cambio de precio relativo más pequeño que puede haber al tradear.
3. ¿Por qué usamos 1.0001? Porque implica que cada tick representa un cambio del 0.01% (1 punto base) en el precio relativo. Esto facilita el análisis financiero.

Finalmente, como la curva es finita, podemos tener en el código valores como `MIN_TICK` y `MAX_TICK`. Uniswap v4 guarda todos estos valores en un tipo de dato `int24` que tiene un rango de `[-8,388,608, 8,388,607]`, pero esto es muy grande para nuestras necesidades. **El rango real que pueden tener los ticks en Uniswap v4 es `[-887,272, 887,272]`.**

### Necesitamos más que ticks

Para ciertas operaciones, como calcular cuánto Token Y necesitas para aportar liquidez dado un rango de precios y una cantidad de Token X, necesitamos fórmulas adicionales. Por ejemplo:

Supongamos que tienes 2 ETH y quieres aportar liquidez en un pool ETH/USDC en el rango de 1500 a 2500 USDC por ETH. Necesitas calcular cuántos USDC se requieren para balancear la posición.

Estas fórmulas implican usar raíces cuadradas de precios (√P, √Pₐ, √P_b), y como Solidity no maneja decimales bien, usamos representaciones con enteros de alta precisión: Q64.96 notation.

## Q64.96

### Qué significa Q64.96

### ¿Por qué es importante?

### Conclusión
