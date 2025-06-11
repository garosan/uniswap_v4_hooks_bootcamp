# Sesión 02 - Historia y Evolución de Uniswap (v1 -> v4)

## Historia de Uniswap de v1 a v4

- Hayden Adams, desempleado conoce el mundo de Ethereum gracias a [Karl Floersch](https://karl.tech/)
- La idea original de un DEX proviene de [Martin Koppelman](https://www.linkedin.com/pulse/martin-koppelman-defi-visionary-who-sparked-paul-mcolaka-mscfe-usmhf/) y fue expandida por Vitalik
- Hayden se pone a implementar estas ideas en código, aprendiendo sobre la marcha
- Uniswap v1 se lanza en noviembre 2018
- Fun Fact 🤓☝️: Se iba a llamar Unipeg
- En Abril 2019 levantan capital, ronda liderada por [Paradigm](https://www.paradigm.xyz/) y comienza crecimiento exponencial 🚀

### Uniswap v1

- Factory contract, cualquiera puede desplegar una pool para un par ETH <-> ERC-20 token
- Primera implementación de un AMM, más especificamente un CPMM: La ecuación que todos conocemos y amamos: `xy = k`
- Problemas con v1:
  - No podias hacer swaps directos ERC-20 <-> ERC-20 (UX mala, costos de gas innecesarios)

### Uniswap v2

- Lanzado a mediados de 2020
- Un hit 🎯, el protocolo más forkeado de la historia
- 3 mejores importantes:
  - Ahora puedes hacer swaps ERC-20 <-> ERC-20 usando un contrato router
  - TWAP oracle, no muy usado ahora, pero en su momento, gran innovación
  - Introdujo Flash Swaps (como Flash Loans, pero para swaps)

Nuevos problemas:

- Proveedores de liquidez sufrían mucha impermanent loss debido a la manera en que funciona la curva `xy = k`
- Provoca ineficiencias de precio ya que muchos tokens realmente no van a ser tradeados en los extremos de la curva

### Uniswap v3

- Lanzado en 2021
- La gran innovación 🔥: Liquidez Concentrada
- LPs ahora pueden decidir en qué rango quieren proveer su liquidez
- Downside: solo pueden ganar fees cuando se tradea en ese rango
- Upside: Capital trabajando más eficiente que nunca
- Fun fact 🤓☝️: El frontend oficial de Uniswap busca el mejor precio entre la liquidez de v2 y v3.

### Uniswap v4

- El problema primero:
  - ❌ Demasiados forks ➡️ Liquidez fragmentada
  - ❌ Confusión para usuari@s finales
  - ❌ Forks no auditados = Vulnerabilidades introducidas, posibles hacks o rugpull
- 🦄 Llega Uniswap v4 al rescate!
- Lanzado en enero 2025
- Introduce hooks 🪝🔥, que nos permiten crear funcionalidades nuevas sobre Uniswap sin forkear todo un DEX!
- Más innovaciones 🤓:
  - Nuevo sistema de contabilidad para liquidar saldos entre _maker_ y _taker_
  - Se utiliza por primera vez **Transient Storage (EIP-1153)** para optimización de gas
  - **Flash Accounting**, permite liquidar saldos pendientes al final de la transancción (introduce funcionalidad adicional)
  - Usuarios pueden no retirar sus tokens de las pools para ahorrar fees

### ¿Qué podemos construir con esto?

- Libros de órdenes onchain
- Curvas de precios personalizadas para crear mercados más eficientes para ciertos tokens (por ejemplo, stablecoins)
- Menos MEV tóxico
- Comisiones dinámicas que reaccionan ante situaciones del mundo real
- Depositar de liquidez fuera de rango en protocolos de lending, para que los LPs ganen rendimiento incluso cuando no se generan fees por swaps
- Auto-compounding de comisiones para reinvertir automáticamente en la posición de LP
- Creación de DEXes empresariales con pools que cumplen con KYC

## Arquitectura v4

### Diseño Singleton

➡️ Diseño v3

➡️ Diseño v4

### Flash Accounting Y Locking

➡️ Flash Accounting

➡️ Locking

➡️ Transient Storage

### ERC-6909 Claims

## v4 Hooks

### Introducción a los Hooks

### Bitmap de una address de un hook

### Flujo de un Swap

### Flujo para modificar una Posición de Liquidez

### Temas Adicionales

➡️ Consideraciones de Gas

➡️ Fragmentación de Liquidez

➡️ Licencia BSL
