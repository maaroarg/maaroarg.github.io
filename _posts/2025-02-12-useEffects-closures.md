---
layout: post
title: "React Hooks, Closures y FSM: Un Caso de Estudio"
date: 2025-02-12
categories: [React Native, Debugging, Desarrollo Móvil]
tags: [react-native, hypertrack, debugging, móvil, ios]
author: Mario Romero
---

# React Hooks, Closures y FSM: Un Caso de Estudio

## El Problema

Mientras trabajaba en una implementación de una máquina de estados finitos (FSM) en React para manejar el tracking de ubicación, me encontré con un problema interesante relacionado con closures y React hooks.

El escenario era el siguiente:

- Teníamos un hook personalizado `useFlags()` que obtenía configuraciones desde LaunchDarkly
- Necesitábamos que ciertas acciones de la FSM dependieran del valor de estos flags
- Aunque los flags se actualizaban correctamente y podíamos verlo en los logs, las acciones de la FSM siempre usaban el valor inicial del flag

```typescript
// Versión inicial problemática
export function useTrackingFSM(debug = false) {
  const { locationAwarenessEnabled } = useFlags();

  const handleTrackingStopped = async (currentStatus: TrackingStatusEnum) => {
    // ... código ...
    locationAwarenessEnabled && (await stopRemoteTrackingAsync());
    // ... más código ...
  };

  const config = useMemo(
    (): FSMConfig<TrackingStatusEnum> => ({
      states: [
        {
          name: TrackingStatusEnum.TRACKING_STOPPED,
          action: async (currentStatus) => {
            await handleTrackingStopped(currentStatus);
          },
        },
        // ... otros estados ...
      ],
    }),
    [handleTrackingStopped]
  );
}
```

## El Diagnóstico

El problema tenía varias capas:

1. **Closure sobre el valor inicial**: La función `handleTrackingStopped` estaba creando un closure sobre el valor inicial de `locationAwarenessEnabled`

2. **Referencias persistentes en la FSM**: La máquina de estados mantenía referencias a las acciones iniciales y no se actualizaba cuando el config cambiaba

3. **Timing de actualizaciones**: Aunque el hook `useFlags()` funcionaba correctamente y actualizaba sus valores, las acciones de la FSM seguían referenciando el valor original debido al closure

Los logs mostraban esta discrepancia claramente:

```
LOG  🎯 [useTrackingFSM] Flags recibidos: {"locationAwarenessEnabled": true}
LOG  🛑 useTrackingMachine Hook. Tracking stopped ... locationAwarenessEnabled=> undefined
```

## La Solución

La solución vino en forma de `useRef`, que nos permitió mantener una referencia mutable que se actualiza con el último valor del flag:

```typescript
export function useTrackingFSM(debug = false) {
  const { locationAwarenessEnabled } = useFlags();
  const locationAwarenessEnabledRef = useRef(locationAwarenessEnabled);

  // Mantener el ref actualizado
  useEffect(() => {
    locationAwarenessEnabledRef.current = locationAwarenessEnabled;
  }, [locationAwarenessEnabled]);

  const handleTrackingStopped = useCallback(
    async (currentStatus: TrackingStatusEnum) => {
      // Usar el ref en lugar del valor directo
      if (locationAwarenessEnabledRef.current) {
        await stopRemoteTrackingAsync();
      }
    },
    [stopRemoteTrackingAsync]
  );

  const config = useMemo(
    (): FSMConfig<TrackingStatusEnum> => ({
      // ... configuración ...
    }),
    [handleTrackingStopped]
  );
}
```

### ¿Por qué funciona esta solución?

1. **useRef mantiene una referencia mutable**: A diferencia de las variables normales o el estado, useRef nos da un objeto mutable que persiste entre renders

2. **Actualizaciones síncronas**: El useEffect asegura que el ref siempre tenga el valor más reciente del flag

3. **Sin problemas de closure**: Al usar el ref dentro del callback, siempre accedemos al valor actual en el momento de la ejecución, no al valor que existía cuando se creó el callback

## Lecciones Aprendidas

1. **Entender los closures en React**: Es crucial entender cómo los closures capturan valores en React, especialmente cuando trabajamos con callbacks que se ejecutan en momentos diferentes

2. **useRef para valores actualizados**: Cuando necesitas acceder a valores actualizados dentro de callbacks que podrían ejecutarse en cualquier momento, useRef es una herramienta poderosa

3. **Debugging efectivo**: Los logs estratégicamente ubicados fueron clave para identificar dónde y cuándo los valores estaban divergiendo

4. **Máquinas de estado y React**: Al trabajar con FSMs en React, hay que tener especial cuidado con cómo las acciones acceden a valores que pueden cambiar con el tiempo

## Conclusión

Este caso demuestra la importancia de entender profundamente cómo funcionan los closures en React y cómo pueden afectar a nuestras aplicaciones. La solución con useRef no solo resolvió el problema inmediato, sino que también proporcionó un patrón reutilizable para situaciones similares donde necesitamos acceder a valores actualizados dentro de callbacks en una FSM.

---
