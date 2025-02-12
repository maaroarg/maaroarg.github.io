---
layout: post
title: "Depurando Crashes del Event Emitter en React Native con HyperTrack SDK"
date: 2024-02-12
tags: [react-native, hypertrack, debugging, móvil, ios]
author: Mario Romero
---

## El Problema

Recientemente, mientras implementábamos el seguimiento de ubicación en nuestra aplicación React Native usando el SDK de HyperTrack, nos encontramos con un crash interesante que solo ocurría al detener el tracking. La aplicación funcionaba perfectamente al iniciar el seguimiento, pero se cerraba inesperadamente con un error críptico al intentar detenerlo.

Así se veía nuestra implementación inicial:

```typescript
const stopLocalTracking = async (fieldTicketId: string) => {
  HyperTrack.setIsTracking(false); // Esta línea causaba el crash
  const orderHandle = fieldTicketId ?? "StopAllTrackingExecuted";
  await HyperTrack.addGeotag(
    orderHandle,
    { type: "orderStatusCustom", value: "stopTracking" },
    { fieldTicketId }
  );
};
```

Al ejecutar este código, recibíamos el siguiente error:

```
NSInternalInconsistencyException, reason: 'Error when sending event:
isTracking with body: {
type = isTracking;
value = 0;
}. RCTCallableJSModules is not set. This is probably because you've explicitly
synthesized the RCTCallableJSModules in HyperTrackReactNativePlugin, even though
it's inherited from RCTEventEmitter.'
```

## Entendiendo el Error

El mensaje de error nos da dos pistas importantes:

1. El crash ocurre cuando se intenta enviar un evento `isTracking` con `value = 0` (false)
2. Hay un problema con `RCTCallableJSModules` en relación con `RCTEventEmitter`

Este error ocurre debido a cómo funciona el puente (bridge) de React Native con los módulos nativos. El SDK de HyperTrack utiliza el sistema de emisión de eventos de React Native para comunicar cambios de estado entre el código nativo y JavaScript. Cuando llamamos a `setIsTracking(false)` antes de completar otras operaciones, estamos intentando emitir eventos mientras el sistema de tracking está siendo desmantelado.

## La Solución

La solución es sorprendentemente simple: necesitamos asegurarnos de que todas las operaciones relacionadas con el tracking (como agregar geotags) se completen antes de detener el sistema de seguimiento:

```typescript
const stopLocalTracking = async (fieldTicketId: string) => {
  const orderHandle = fieldTicketId ?? "StopAllTrackingExecuted";
  // Primero, enviamos el geotag
  await HyperTrack.addGeotag(
    orderHandle,
    { type: "orderStatusCustom", value: "stopTracking" },
    { fieldTicketId }
  );
  // Luego detenemos el tracking
  await HyperTrack.setIsTracking(false);
};
```

## ¿Por qué Funciona?

Al reordenar las operaciones, nos aseguramos de que:

1. Todas las operaciones de geotag se completen mientras el sistema de tracking aún está activo
2. El puente de eventos está completamente funcional cuando necesitamos enviar datos de geotag
3. Solo desactivamos el tracking después de que todas las demás operaciones están completas

## Aprendizajes Clave

Cuando trabajamos con módulos nativos de React Native que utilizan emisores de eventos:

1. Prestar atención al orden de las operaciones, especialmente cuando se trata de limpieza o cambios de estado
2. Asegurarse de que las operaciones dependientes se completen antes de deshabilitar o limpiar sistemas
3. Buscar pistas en los mensajes de error sobre la emisión de eventos y problemas relacionados con el puente

## Consideraciones Adicionales

Si estás lidiando con problemas similares, podrías querer agregar medidas de seguridad adicionales:

```typescript
const stopLocalTracking = async (fieldTicketId: string) => {
  try {
    const orderHandle = fieldTicketId ?? "StopAllTrackingExecuted";
    await HyperTrack.addGeotag(
      orderHandle,
      { type: "orderStatusCustom", value: "stopTracking" },
      { fieldTicketId }
    );

    // Opcional: Agregar un pequeño delay para asegurar que el geotag se procesó
    await new Promise((resolve) => setTimeout(resolve, 100));

    await HyperTrack.setIsTracking(false);
  } catch (error) {
    console.error("Error al detener el tracking local:", error);
    // Manejar el error apropiadamente
  }
};
```

## Conclusión

Este caso ilustra un patrón común en el desarrollo con React Native donde el orden de las operaciones se vuelve crucial cuando se trabaja con módulos nativos y emisores de eventos. Lo que parecía una simple operación de detención de tracking en realidad requería una consideración cuidadosa del sistema de emisión de eventos subyacente.

Es importante recordar que cuando trabajamos con módulos nativos en React Native, debemos entender no solo qué hace cada operación, sino también cómo estas operaciones interactúan con el puente de React Native y el sistema de eventos.
