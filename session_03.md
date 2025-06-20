# Sesión 03 - Entendiendo la Arquitectura del contrato `PoolManager` - Leyendo el Estado del Pool (Reading Pool State)

## Introducción

A diferencia de versiones anteriores, v4 usa un enfoque diferente para almacenar y acceder a datos del pool, requiriendo entender el uso de `StateLibrary` y `extsload`.

## Arquitectura del PoolManager

### El Diseño Singleton

- **v3**: Cada pool era un contrato separado
- **v4**: Un solo contrato `PoolManager` administra todas las pools

Beneficios del diseño singleton:

- ✅ Reduce costos de deployment
- ✅ Simplifica upgrades del protocolo
- ✅ Permite interacciones más eficientes entre pools
- ✅ Facilita implementar nuevas features en todas las pools

### Pools como Library Calls

```solidity
contract PoolManager {
    using Pool for Pool.State;
    mapping(PoolId => Pool.State) internal pools;

    function swap(PoolId id, ...) external {
        pools[id].swap(...); // Library call
    }
}
```

- Los pools ahora son structs complejos
- Las librerías de Solidity manejan los cambios de estado
- Toda la lógica AMM está encapsulada en el PoolManager

## Estructura de Datos en v4

### El Mapping Principal

```solidity
mapping(PoolId id => Pool.State) internal _pools;
```

Cada `Pool.State` contiene:

```solidity
struct State {
    uint160 sqrtPriceX96;
    int24 tick;
    uint128 liquidity;
    uint256 feeGrowthGlobal0X128;
    uint256 feeGrowthGlobal1X128;
    mapping(int24 => TickInfo) ticks;
    mapping(bytes32 => Position.Info) positions;
    // ... otros campos
}
```

**Desafío**: Los mappings anidados hacen que los getters tradicionales sean ineficientes 🤔

## StateLibrary y extsload

### ¿Qué es extsload?

```solidity
function extsload(bytes32 slot) external view returns (bytes32) {
    assembly ("memory-safe") {
        mstore(0, sload(slot))
        return(0, 0x20)
    }
}
```

**¿Cómo funciona?**

- Toma un slot de storage como input
- Lee el valor usando `SLOAD` directamente
- Retorna el valor como `bytes32`

**Beneficios**:

- ⚡ Más eficiente en gas que getters tradicionales
- 📦 Reduce el tamaño del bytecode del contrato
- 🎯 Crucial para v4 (los contratos están cerca del límite de tamaño de Ethereum)

### TransientStateLibrary y exttload

- Similar a `StateLibrary` pero para **transient storage**
- Usa `exttload` (wrapper para `TLOAD`)
- Para datos que solo se necesitan durante una transacción

## Implementando un PoolStateReader

### Setup Básico

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

import {IPoolManager} from "v4-core/interfaces/IPoolManager.sol";
import {StateLibrary} from "v4-core/libraries/StateLibrary.sol";
import {PoolKey} from "v4-core/types/PoolKey.sol";
import {PoolId, PoolIdLibrary} from "v4-core/types/PoolId.sol";

contract PoolStateReader {
    using PoolIdLibrary for PoolKey;
    using StateLibrary for IPoolManager; // 🔑 Línea clave!

    IPoolManager public immutable poolManager;

    constructor(IPoolManager _poolManager) {
        poolManager = _poolManager;
    }
}
```

**Nota importante**: `using StateLibrary for IPoolManager;` permite llamar funciones de StateLibrary como métodos del poolManager.

## Funciones Principales para Leer Estado

### 1. getSlot0() - Estado Básico del Pool

```solidity
function getPoolState(PoolKey calldata key) external view returns (
    uint160 sqrtPriceX96,
    int24 tick,
    uint24 protocolFee,
    uint24 lpFee
) {
    return poolManager.getSlot0(key.toId());
}
```

**¿Qué retorna?**

- `sqrtPriceX96`: Precio actual codificado como raíz cuadrada
- `tick`: Tick actual (precio cuantizado)
- `protocolFee`: Fee del protocolo en centésimos de bip
- `lpFee`: Fee para LPs en centésimos de bip

**Casos de uso**:

- 🔮 Oráculos de precio
- 🤖 Bots de trading
- 💰 Sistemas de gestión de liquidez

### 2. getLiquidity() - Liquidez Total

```solidity
function getPoolLiquidity(PoolKey calldata key) external view returns (uint128 liquidity) {
    return poolManager.getLiquidity(key.toId());
}
```

**Casos de uso**:

- 📊 Evaluar profundidad del mercado y slippage potencial
- 📈 Monitorear crecimiento del pool
- 🎯 Análisis de liquidez disponible

### 3. getPositionInfo() - Info de Posiciones Específicas

```solidity
function getPositionInfo(
    PoolKey calldata key,
    address owner,
    int24 tickLower,
    int24 tickUpper,
    bytes32 salt
) external view returns (
    uint128 liquidity,
    uint256 feeGrowthInside0LastX128,
    uint256 feeGrowthInside1LastX128
) {
    return poolManager.getPositionInfo(key.toId(), owner, tickLower, tickUpper, salt);
}
```

**¿Qué retorna?**

- `liquidity`: Cantidad de liquidez en la posición
- `feeGrowthInside0/1LastX128`: Fees acumulados por unidad de liquidez

**Casos de uso**:

- 🎛️ Dashboards de gestión de liquidez
- 🔄 Sistemas de rebalanceo automático
- 📊 Herramientas de análisis de performance

### 4. getFeeGrowthGlobal() - Crecimiento Global de Fees

```solidity
function getFeeGrowthGlobal(PoolKey calldata key) external view returns (
    uint256 feeGrowthGlobal0X128,
    uint256 feeGrowthGlobal1X128
) {
    return poolManager.getFeeGrowthGlobal(key.toId());
}
```

**Casos de uso**:

- 💰 Calcular fees ganados en posiciones de largo plazo
- 📊 Analizar generación de fees histórica
- 🔍 Comparar performance entre pools

## Puntos Clave para Recordar

1. **Singleton Design**: Un solo PoolManager maneja todas las pools
2. **StateLibrary**: Herramienta esencial para leer estado eficientemente
3. **extsload**: Método más eficiente que getters tradicionales
4. **Transient Storage**: Para datos temporales durante transacciones
5. **Complex Mappings**: Requieren métodos especiales para acceso eficiente

## Próximos Pasos

En la siguiente sesión veremos:

- Cómo implementar hooks básicos
- Casos de uso prácticos de hooks
- Deployment y testing de hooks

---

**Recursos Adicionales**:

- [Documentación oficial de Uniswap v4](https://docs.uniswap.org/contracts/v4/guides/read-pool-state)
- [Ejemplo completo en GitHub](https://github.com/uniswapfoundation/v4-template)
