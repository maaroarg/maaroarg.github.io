---
layout: post
title: "Entendiendo useState, useRef y useCallback en React"
date: 2025-01-31
categories: react javascript frontend
author: Mario Romero
---

En el desarrollo con React, los hooks son una parte fundamental para manejar el estado y otros aspectos de nuestros componentes. Hoy vamos a profundizar en tres hooks esenciales: useState, useRef y useCallback, entendiendo sus diferencias, casos de uso y mejores prácticas.

## useState vs useRef: ¿Cuándo usar cada uno?

La principal diferencia entre estos hooks radica en su propósito y comportamiento con respecto a los re-renders.

### useState

- Causa re-renders cuando el valor cambia
- Se usa para valores que necesitan reflejarse en la UI
- Mantiene el estado entre renderizados
- Es reactivo: los componentes se actualizan cuando cambia

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

### useRef

- No causa re-renders cuando cambia
- Mantiene valores entre renderizados
- Útil para referencias al DOM
- Acceso inmediato al valor actualizado
- Ideal para valores que no necesitan reflejarse en la UI

```javascript
function StopWatch() {
  const intervalRef = useRef(null);
  const [time, setTime] = useState(0);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setTime((prev) => prev + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };

  return (
    <div>
      <p>Time: {time}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

## useCallback: Optimización de Funciones

useCallback es una herramienta de optimización que memoriza funciones. Es especialmente útil en tres escenarios principales:

1. Cuando pasas funciones a componentes memorizados (React.memo)
2. Cuando las funciones son dependencias de useEffect
3. Cuando las funciones son parte de un contexto

### ¿Por qué no usar useCallback siempre?

Aunque podría parecer tentador usar useCallback por defecto, hay razones por las que React no lo hace:

1. **Overhead de Memoria**: Cada useCallback necesita almacenar la función y sus dependencias
2. **Complejidad en Comparaciones**: React necesita comparar las dependencias en cada render
3. **Optimización Prematura**: En muchos casos, el costo de la memorización es mayor que el beneficio

```javascript
// Ejemplo de cuándo usar useCallback
function ParentComponent() {
  const [count, setCount] = useState(0);

  // Sin useCallback - se crea nueva función en cada render
  const handleClick = () => {
    console.log("clicked");
  };

  // Con useCallback - mantiene la misma referencia
  const handleClickMemoized = useCallback(() => {
    console.log("clicked");
  }, []);

  return <ExpensiveChild onAction={handleClickMemoized} />;
}

// Componente hijo memorizado
const ExpensiveChild = React.memo(({ onAction }) => {
  console.log("Child rendered");
  return <button onClick={onAction}>Click me</button>;
});
```

## Mejores Prácticas

1. **Para useState:**

   - Úsalo cuando el valor necesita reflejarse en la UI
   - Cuando otros componentes necesitan reaccionar al cambio del valor
   - Para estado que afecta el renderizado del componente

2. **Para useRef:**

   - Referencias al DOM
   - Valores que persisten entre renders pero no afectan la UI
   - Intervalos, timeouts, y otros valores que no requieren re-renders
   - Cuando necesitas acceso inmediato al valor actualizado

3. **Para useCallback:**
   - Cuando pasas funciones a componentes memorizados
   - Cuando la función es una dependencia de useEffect
   - En contextos donde la identidad de la función es importante
   - Cuando tienes evidencia de problemas de rendimiento

## Conclusión

La elección entre useState, useRef y useCallback depende del caso de uso específico. No hay una regla universal, pero entender sus diferencias y propósitos nos ayuda a tomar mejores decisiones en nuestro código. La clave está en usar cada hook según sus fortalezas y no caer en la optimización prematura.

Lo más importante es mantener nuestro código limpio, mantenible y eficiente, usando estas herramientas de manera estratégica cuando realmente aportan valor a nuestra aplicación.
