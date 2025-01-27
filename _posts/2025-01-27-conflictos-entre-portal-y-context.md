---
layout: post
title: "React Native: Conflicto entre Portal y Context en implementación de Modales"
date: 2024-01-27 09:00:00 -0300
categories: react-native react-native-paper context portal
---

Durante la implementación de una funcionalidad de tracking en una aplicación React Native, nos encontramos con un problema interesante relacionado con el componente Portal de React Native Paper y React Context. Aquí te cuento cómo identificamos y resolvimos el problema.

## El Problema

Teníamos una aplicación React Native usando:

- React Native Paper para componentes de UI
- React Context para manejo de estado
- Modales para confirmaciones de usuario

El problema específico ocurría al intentar detener una funcionalidad de tracking:

1. El modal se mostraba correctamente
2. Después de confirmar la acción de detener en el modal
3. El tracking se detenía pero inmediatamente se reiniciaba

Lo curioso era que la misma funcionalidad trabajaba perfectamente sin modales, lo que sugería un problema de interacción entre el Portal (usado por el Modal de React Native Paper) y nuestro Context.

## Investigación Inicial

Probamos varios enfoques:

1. Primero, agregamos logs extensivos para entender la secuencia de eventos
2. Intentamos controlar el estado de detención con refs
3. Intentamos reorganizar el orden de las actualizaciones de estado

Ninguna de estas soluciones funcionó completamente, pero nos ayudaron a identificar que el problema estaba relacionado con el componente Portal perdiendo el contexto durante el renderizado del modal.

## La Solución

La solución involucró crear nuestra propia implementación de modal usando el componente Modal nativo de React Native en lugar del modal basado en Portal de React Native Paper.

1. Primero, creamos un componente CustomModal base:

```typescript
// CustomModal.tsx
const CustomModal: React.FC<CustomModalProps> = ({
  visible,
  onDismiss,
  children,
}) => {
  return (
    <RNModal
      visible={visible}
      transparent
      animationType="fade"
      onRequestClose={onDismiss}
    >
      <View style={styles.overlay}>
        <View style={styles.modalContainer}>{children}</View>
      </View>
    </RNModal>
  );
};
```

2. Luego, creamos un componente ConfirmationModal que usa nuestro CustomModal:

```typescript
// ConfirmationModal.tsx
const ConfirmationModal: React.FC<ConfirmationModalProps> = ({
  visible,
  onConfirm,
  onDismiss,
  title,
  description,
  confirmButtonText,
  dismissButtonText = "Volver atrás",
}) => {
  return (
    <CustomModal visible={visible} onDismiss={onDismiss}>
      {/* Contenido del Modal */}
    </CustomModal>
  );
};
```

3. Finalmente, mantuvimos nuestras implementaciones específicas de modales pero usando los nuevos componentes base:

```typescript
// TrackingStopConfirmationModal.tsx
const TrackingStopConfirmationModal: React.FC<Props> = (props) => {
  return (
    <ConfirmationModal
      {...props}
      title="¿Has terminado este trabajo?"
      description="Al continuar, dejaremos de trackear tu ubicación..."
      confirmButtonText="Sí, el trabajo está completo"
    />
  );
};
```

## Aprendizajes Clave

1. **Problemas con Portal y Context**: En React Native, usar Portal puede causar problemas relacionados con el contexto ya que mueve el contenido a una parte diferente del árbol de componentes.

2. **Soluciones Nativas**: A veces usar componentes nativos (como el Modal de React Native) es mejor que soluciones más complejas que pueden introducir comportamientos inesperados.

3. **Consistencia de API**: Mantuvimos la misma API de componentes mientras cambiábamos la implementación interna, haciendo la solución transparente para el resto de la aplicación.

## La Solución en Acción

Después de implementar esta solución:

1. El tracking se detiene correctamente cuando se confirma a través del modal
2. El contexto se preserva durante todo el ciclo de vida del modal
3. La experiencia de usuario se mantiene consistente con la implementación anterior

## Conclusión

Este caso muestra cómo problemas aparentemente complejos a menudo pueden resolverse simplificando nuestro enfoque y usando componentes nativos más básicos. También resalta la importancia de entender cómo diferentes patrones de React (como Portal y Context) interactúan en un entorno React Native.

Recuerda considerar el impacto de las abstracciones de UI en el manejo de estado cuando debuguees problemas similares en aplicaciones React Native.
