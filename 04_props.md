# DWEC UT05: Introducción a los frameworks: React.

## Pasar información a componentes

Los componentes de React utilizan **props** para comunicarse entre sí. Cada componente padre puede enviar información a sus componentes hijos mediante el uso de `props`. Las `props` pueden parecerte similares a los atributos HTML, pero permiten pasar cualquier valor de *JavaScript* a través de ellas, como objetos, arrays y funciones.

### props conocidas

Las props son los datos que se pasan a un elemento JSX. Por ejemplo, className, src, alt, width y height son algunas de las props que se pueden pasar a un elemento `<img>`.

```jsx
function Avatar(){
  return (
    <img
      className="avatar"
      src="https://randomuser.me/api/portraits/men/77.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```
Las props que puedes utilizar con una etiqueta `<img>` están predefinidas (ReactDOM se ajusta al [estándar HTML](https://www.w3.org/TR/html52/semantics-embedded-content.html#the-img-element)). Sin embargo, puedes pasar cualquier prop a tus propios componentes, como `<Avatar>`, para personalizarlos.

### Pasar props a componente hijo

Tenemos los siguientes componentes, un componente de nombre `Hello` y otro de nombre `App`.

```jsx
// componente Hello
const Hello = () => {  
    return (
    <div>
      <p>Hello World!</p>    
    </div>
  )
}

// componente App
const App = () => {
  return (
    <div>
      <h1>Greetings</h1>
      <Hello />      
      <Hello />    
    </div>
  )
}
```

Veamos como le pasariamos las props al componente `Hello`.

```jsx
// componente Hello
const Hello = (props) => {  
    return (
    <div>
      <p>Hello {props.name}</p>    
    </div>
  )
}
```

Ahora la función que define el componente tiene un parámetro `props`. Como argumento, el parámetro recibe un objeto, que tiene campos correspondientes a todos los `props` ("accesorios") que el usuario del componente define.

```jsx
// componente App
const App = () => {
  return (
    <div>
      <h1>Greetings</h1>
      <Hello name='George' />      
      <Hello name='Lisa' />    
    </div>
  )
}
```

Puede haber un número arbitrario de props y sus valores pueden ser strings "incrustados en el código" ("hard coded") o resultados de expresiones JavaScript. Si el valor del prop se obtiene usando JavaScript, debe estar envuelto con llaves.

Modifiquemos el código para que el componente `Hello` use dos props: 

```js
const Hello = (props) => {
  console.log(props)  
  return (
    <div>
      <p>Hello {props.name}, you are {props.age} years old</p>
    </div>
  )
}

const App = () => {
  const name = 'Peter'  
  const age = 10
  return (
    <div>
      <h1>Greetings</h1>
      <Hello name='Maya' age={26 + 10} />      
      <Hello name={name} age={age} />    
    </div>
  )
}
```

Los `props` enviados por el componente `App` son los valores de las variables, el resultado de la evaluación de la expresión de suma y un string regular.

El componente `Hello` también imprime en consola el valor del objeto props.
> La primera regla del desarrollo web frontend:
> * Deja la consola abierta todo el tiempo
>
> Repitamos esto juntos: Prometo dejar la consola abierta todo el tiempo durante este curso, y por el resto de mi vida mientras esté haciendo desarrollo web.

## Funciones auxiliares

Vamos a expandir nuestro componente `Hello` para que adivine el año de nacimiento de la persona que recibe la bienvenida. La lógica para adivinar el año de nacimiento se divide en su propia función que se llama cuando se representa el componente.

La edad de la persona no tiene que pasarse como parámetro a la función, ya que puede acceder directamente a todos los props que se pasan al componente.

```jsx
const Hello = (props) => {
  const bornYear = () => {    
    const yearNow = new Date().getFullYear()    
    return yearNow - props.age  
  }
  return (
    <div>
      <p>Hello {props.name}, you are {props.age} years old</p>
      <p>So you were probably born in {bornYear()}</p>    
    </div>
  )
}
```

Si examinamos nuestro código actual de cerca, notaremos que la función auxiliar está realmente definida dentro de otra función que define el comportamiento de nuestro componente.

## Desestructuración de props

En nuestro código anterior, teníamos que hacer referencia a los datos pasados ​​a nuestro componente como `props.name` y `props.age`. De estas dos expresiones, tuvimos que repetir `props.age` dos veces en nuestro código.

Dado que `props` es un objeto.

```js
props = {
  name: 'Marco Aurelio',
  age: 35,
}
```

Podemos optimizar nuestro componente asignando los valores de las propiedades directamente en dos variables name y age que luego podemos usar en nuestro código.

```jsx
const Hello = (props) => {
  const name = props.name  
  const age = props.age
  const bornYear = () => new Date().getFullYear() - age
  return (
    <div>
      <p>Hello {name}, you are {age} years old</p>      
      <p>So you were probably born in {bornYear()}</p>
    </div>
  )
}
```

La desestructuración facilita aún más la asignación de variables, ya que podemos usarla para extraer y reunir los valores de las `propiedades` de un `objeto` en variables separadas.

```jsx
  const { name, age } = props
```

Podemos llevar la desestructuración un paso más allá.

```jsx
const Hello = ({ name, age }) => {  
  const bornYear = () => new Date().getFullYear() - age
  return (
    <div>
      <p>Hello {name}, you are {age} years old</p>
      <p>So you were probably born in {bornYear()}</p>
    </div>
  )
}
```

### Valores por defecto de props

Si quieres asignar un valor predeterminado para una `prop` en caso de que no se especifique ningún valor, puedes hacerlo mediante la desestructuración colocando `=` seguido del valor predeterminado justo después del parámetro.

```jsx
const Hello = ({name, age = 25}) => {
  return (
    <div>
      <p>
        Hello {name}, you are {age} years old      
      </p>
    </div>
  )
}
```

### Reenviando props 

A veces, pasar props se vuelve muy repetitivo.

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

No hay ningún problema en tener código repetitivo, ya que puede ser más legible. Sin embargo, en ocasiones, es posible que prefieras ser más conciso. Algunos componentes reenvían todas sus props a sus hijos, como lo hace `Profile` con `Avatar`. Dado que no utilizan directamente ninguna de sus `props`, tiene sentido utilizar una sintaxis de **«propagación»** más concisa.

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

### Notas sobre errores

React se ha configurado para generar mensajes de error bastante claros. A pesar de esto, debes, al menos al principio, avanzar en pasos muy pequeños y asegurarte de que cada cambio funcione como se desea.

La consola siempre debe estar abierta. Si el navegador reporta errores, no es recomendable seguir escribiendo más código, esperando milagros. En su lugar, debes intentar comprender la causa del error y, por ejemplo, volver al estado funcional anterior.

También ten en cuenta que **los nombres de los componentes de React deben iniciar con mayúscula**. Si intentas definir un componente de la siguiente manera:

```jsx
const footer = () => {
  return (
    <div>
      greeting app created by <a href='https://github.com/mluukkai'>mluukkai</a>
    </div>
  )
}
```

Y lo usamos de esta manera:

```jsx
const App = () => {
  return (
    <div>
      <h1>Greetings</h1>
      <Hello name='Maya' age={26 + 10} />
      <footer />    </div>
  )
}
```

La página no mostrará el contenido definido dentro del componente `Footer`, y en su lugar React solo crea un elemento `footer` vacío. Si cambias la primera letra del nombre del componente a una letra mayúscula, React crea el elemento `div` definido en el componente `Footer`, que se representa en la página.

## Renderizado condicional

Tus componentes a menudo necesitarán mostrar diferentes cosas dependiendo de diferentes condiciones. En React, puedes renderizar JSX de forma condicional utilizando la sintaxis de JavaScript como las declaraciones `if`, `&&` y los operadores `? :`.

Supongamos que tienes un componente `PackingList` que muestra varios `Items`, que pueden ser marcados como empaquetados o no:

```jsx
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista de equipaje de Sally Ride</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Traje de vuelo" 
        />
        <Item 
          isPacked={true} 
          name="Casco con dorado a la hoja" 
        />
        <Item 
          isPacked={false} 
          name="Fotografía de Tam" 
        />
      </ul>
    </section>
  );
}
```

Observa que algunos de los componentes `Item` tienen su prop `isPacked` asignada a `true` en lugar de `false`. Se desea añadir una marca de verificación **(✔)** a los elementos empaquetados si `isPacked={true}`.

Puedes escribir esto como una declaración if/else así:

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```
### Devolución con null

En algunas situaciones, no querrás mostrar nada en absoluto. Por ejemplo, digamos que no quieres mostrar elementos empaquetados en absoluto. Un componente debe devolver algo. En este caso, puedes devolver `null`.

```jsx
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

En la práctica, devolver `null` en un componente no es común porque podría sorprender a un desarrollador que intente renderizarlo. Lo más frecuente es incluir o excluir condicionalmente el componente en el JSX del componente padre.

### Operador ternario

En el ejemplo anterior, controlabas qué árbol JSX (si es que había alguno) era devuelto por el componente. Es posible que ya hayas notado alguna duplicación en la salida de la renderización:

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

Aunque esta duplicación no es perjudicial, podría hacer que tu código sea más difícil de mantener. ¿Qué pasa si quieres cambiar el `className`? ¡Tendrías que hacerlo en dos lugares en tu código! En tal situación, podrías incluir condicionalmente un poco de JSX.

```jsx
function Item({ name, isPacked }) {
return (
  <li className="item">
    {isPacked ? name + ' ✔' : name}
  </li>
)}
```
Puedes leerlo como «si `isPacked` es verdadero, entonces (?) renderiza `name` + ' `✔`', de lo contrario (:) renderiza `name`»

Ahora digamos que quieres envolver el texto del elemento completado en otra etiqueta HTML, como `<del> `para tacharlo.

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {
      isPacked 
      ?(<del>{name + ' ✔'}</del>)
      :(name)
      }
    </li>
  );
}
```

### Operador lógico AND (`&&`)

Otro atajo común que encontrarás es el operador lógico `AND` (&&) de JavaScript. Dentro de los componentes de React, a menudo surge cuando quieres renderizar algún JSX cuando la condición es verdadera, o no renderizar nada en caso contrario. Con `&&`, podrías renderizar condicionalmente la marca de verificación sólo si `isPacked` es `true`:

```jsx
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

### Asignación condicional de JSX a una variable 

Cuando los atajos se interpongan en el camino de la escritura de código simple, prueba a utilizar una sentencia `if` y una variable. Puedes reasignar las variables definidas con `let`, así que empieza proporcionando el contenido por defecto que quieres mostrar, el nombre:

```jsx
let itemContent = name;
if (isPacked) {
  itemContent = name + " ✔";
}
<li className="item">
  {itemContent}
</li>
```