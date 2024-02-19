# DWEC UT05: Introducción a los frameworks: React.

## Re-renderizar componentes

Hasta ahora, todas nuestras aplicaciones han sido tales que su apariencia sigue siendo la misma después de la renderización inicial. ¿Qué pasaría si quisiéramos crear un contador donde el valor aumentara en función del tiempo o con el clic de un botón?

Comencemos con lo siguiente. El archivo App.jsx se convierte en:

```jsx
const App = (props) => {
  const {counter} = props
  return (
    <div>{counter}</div>
  )
}

export default App
```
Y el archivo main.jsx se convierte en:

```jsx
import ReactDOM from 'react-dom/client'
import App from './App'

let counter = 1
ReactDOM.createRoot(document.getElementById('root')).render(
  <App counter={counter} />
)
```
El componente de la aplicación recibe el valor del contador a través de la `prop` `counter`. Este componente muestra el valor en la pantalla. ¿Qué sucede cuando cambia el valor de counter?

```jsx
counter += 1
```

El componente **no volverá** a renderizar. Podemos hacer que el componente se vuelva a renderizar llamando al método `ReactDOM.render` varias veces, por ejemplo, de la siguiente manera:

```jsx
let counter = 1

const refresh = () => {
  ReactDOM.createRoot(document.getElementById('root')).render(
    <App counter={counter} />
  )
}

refresh()
counter += 1
refresh()
counter += 1
refresh()
```

Podemos implementar una funcionalidad un poco más interesante volviendo a renderizar e incrementando el contador cada segundo usando setInterval:

```jsx
let counter = 10
const refresh = () => {
  ReactDOM.createRoot(document.getElementById('root')).render(
    <App counter={counter} />
  )
}

setInterval(() => {
  refresh()
  counter += 1
}, 1000)
```

Hacer llamadas repetidas al método `ReactDOM.render` no es la forma recomendada de volver a renderizar componentes. A continuación, presentaremos una mejor forma de lograr este efecto.

## Estado de un componente

Todos nuestros componentes hasta ahora han sido simples en el sentido de que no contienen ningún estado que pueda cambiar durante el ciclo de vida del componente.

A continuación, agreguemos estado al componente `App` de nuestra aplicación con la ayuda del `hook` de estado de React. Cambiaremos la aplicación a lo siguiente en el fichero `main.jsx`:

```js
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

Y el fichero `App.jsx` quedara de esta manera:

```js
import { useState } from 'react'

const App = () => {
  const [ counter, setCounter ] = useState(0)
  setTimeout(() => setCounter(counter + 1), 1000)
  return (
    <div>{counter}</div>
  )
}

export default App
```

En la primera fila, la aplicación importa la función `useState`. El cuerpo de la función que define el componente comienza con la llamada a la función:

```js
// const [algo, setAlgo] = useState(initialState)

const [ counter, setCounter ] = useState(0)
```
* `initialState`: El valor que deseas que tenga el estado inicialmente. Puede ser un valor de cualquier tipo, pero hay un comportamiento especial para las funciones. Este argumento se ignora después del renderizado inicial.
> Si pasa una función como `initialState`, se tratará como una función inicializadora. Debe ser pura, no debe aceptar argumentos y debe devolver un valor de cualquier tipo. React llamará a tu función de inicialización al inicializar el componente y almacenará su valor de devolución como el estado inicial.

* `useState` devuelve un array con exactamente dos valores:
  1. **El estado actual**. Durante el primer renderizado, coincidirá con el **initialState** que hayas pasado.
  2. La función `set` que te permite actualizar el estado a un valor diferente y **desencadenar un nuevo renderizado**.

> `useState` es un Hook, por lo que solo puedes llamarlo en el nivel superior de tu componente o en tus propios Hooks. No puedes llamarlo dentro de bucles o condiciones. Si lo necesitas, extrae un nuevo componente y mueva el estado a él.
> En [Modo estricto](https://es.react.dev/reference/react/StrictMode), React llamará a tu función de inicialización dos veces para ayudarte a encontrar impurezas accidentales. Este es un comportamiento exclusivo de desarrollo y no ocurre en producción. Si tu función de inicialización es pura (como debería ser), esto no debería afectar la lógica de tu componente. Se ignorará el resultado de una de las llamadas.

En este ejemplo la llamada a la función agrega `state` al componente y lo hace inicializado con el valor de *cero*. La función devuelve una matriz que contiene dos elementos. Asignamos los elementos a las variables `counter` y `setCounter` usando la sintaxis de asignación por desestructuración mostrada anteriormente.

* A la variable `counter` se le asigna el valor inicial de state que es cero. 
* La variable `setCounter` se asigna a una **función** que se utilizará para modificar el estado.

La aplicación llama a la función `setTimeout` y le pasa dos parámetros: una función para incrementar el estado del contador y un tiempo de espera de un segundo.

Cuando se llama a la función de modificación de estado `setCounter`, React **vuelve a renderizar el componente**, lo que significa que el cuerpo de la función del componente se vuelve a ejecutar:

```jsx
() => {
  const [ counter, setCounter ] = useState(0)
  setTimeout(
    () => setCounter(counter + 1),
    1000
  )
  return (
    <div>{counter}</div>
  )
}
```

La segunda vez que la función del componente es ejecutado, llama a la función `useState` y devuelve el nuevo valor del estado: `1`. 
Al ejecutar el cuerpo de la función nuevamente, también se realiza una nueva llamada de función a `setTimeout`, que ejecuta el tiempo de espera de un segundo e incrementa el estado `counter` nuevamente. Debido a que el valor de la variable `counter` es `1`, incrementar el valor en 1 es esencialmente lo mismo que una expresión que establece el valor de `counter` en 2.

Cada vez que `setCounter` modifica el estado, **hace que el componente se vuelva a renderizar**. El valor del estado se incrementará nuevamente después de un segundo y esto continuará repitiéndose mientras la aplicación esté en ejecución.

### Funciones `setAlgo`

```jsx
setAlgo(siguienteEstado)
```

La función `set` devuelta por `useState` te permite actualizar el estado a un valor diferente y desencadenar un nuevo renderizado. Puedes pasar el siguiente estado directamente, o una función que lo calcule a partir del estado anterior:

```jsx
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```
* `siguienteEstado`: El valor que deseas que tenga el estado. Puede ser un valor de cualquier tipo, pero hay un comportamiento especial para las funciones.
* Si pasas una función como `siguienteEstado`, se tratará como una función de actualización. Debe ser pura, debe tomar el estado pendiente como único argumento y debe devolver el siguiente estado. React pondrá tu función de actualización en una cola y volverá a renderizar tu componente. Durante el próximo renderizado, React calculará el siguiente estado aplicando todas las actualizaciones en cola al estado anterior. Ve un ejemplo debajo.

Las funciones `set` **no tienen un valor de devolución**.

> La función set solo actualiza la variable de estado para el próximo renderizado. Si lees la variable de estado después de llamar a la función set, seguirás obteniendo el valor anterior que estaba en la pantalla antes de tu llamada.

Supongamos que `age` es `42`. La función handler llama `setAge(age + 1)` tres veces:

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

¡Sin embargo, después de un click, age solo será `43` en lugar de `45`!  Esto se debe a que llamar a la función `set`  no actualizará  la variable de estado age en el código que ya se está ejecutando. Así que cada llamada `setAge(age + 1)` se convierte en `setAge(43)`.

Para resolver este problema, puedes pasar una  función de actualización  a `setAge` en lugar del siguiente estado:

```jsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

Aquí, `a => a + 1` es la función de actualización. Toma el estado pendiente y calcula el siguiente estado a partir de él.

React pone sus funciones de actualización en una cola. Entonces, durante el siguiente renderizado, las llamará en el mismo orden:

1. a => a + 1 recibirá 42 como estado pendiente y devolverá 43 como el siguiente estado.
2. a => a + 1 recibirá 43 como estado pendiente y devolverá 44 como el siguiente estado.
3. a => a + 1 recibirá 44 como estado pendiente y devolverá 45 como el siguiente estado.

No hay otras actualizaciones en cola, por lo que React almacenará 45 como el estado actual al final.

### Estados con array
### Estados con objetos
