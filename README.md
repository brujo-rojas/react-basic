# React - Basic Theoretical Concepts

> **🇪🇸 Traducción**: Este documento ha sido traducido al español y es un fork del repositorio [reactjs/react-basic](https://github.com/reactjs/react-basic).

This document is my attempt to formally explain my mental model of React. The intention is to describe this in terms of deductive reasoning that lead us to this design.

> **🇪🇸** Este documento es mi intento de explicar formalmente mi modelo mental de React. La intención es describirlo en términos de razonamiento deductivo que nos lleva a este diseño.

There may certainly be some premises that are debatable and the actual design of this example may have bugs and gaps. This is just the beginning of formalizing it. Feel free to send a pull request if you have a better idea of how to formalize it. The progression from simple -> complex should make sense along the way without too many library details shining through.

> **🇪🇸** Puede que haya algunas premisas discutibles y que el diseño real de este ejemplo tenga errores o huecos. Esto es solo el comienzo de su formalización. Si tienes una mejor idea de cómo formalizarlo, siéntete libre de enviar un pull request. La progresión de simple → complejo debería tener sentido a lo largo del camino sin demasiados detalles de librería que distraigan.

The actual implementation of React.js is full of pragmatic solutions, incremental steps, algorithmic optimizations, legacy code, debug tooling and things you need to make it actually useful. Those things are more fleeting, can change over time if it is valuable enough and have high enough priority. The actual implementation is much more difficult to reason about.

> **🇪🇸** La implementación real de React.js está llena de soluciones pragmáticas, pasos incrementales, optimizaciones algorítmicas, código heredado, herramientas de depuración y cosas que hacen que sea realmente útil. Esas partes son más efímeras, pueden cambiar con el tiempo si vale la pena y tienen mayor prioridad. La implementación real es mucho más difícil de razonar.

I like to have a simpler mental model that I can ground myself in.

> **🇪🇸** Me gusta tener un modelo mental más sencillo en el que apoyarme.

## Transformation

The core premise for React is that UIs are simply a projection of data into a different form of data. The same input gives the same output. A simple pure function.

> **🇪🇸** La premisa central de React es que las interfaces de usuario son simplemente una proyección de datos hacia una forma distinta de datos. La misma entrada produce la misma salida. Una función pura y simple.

```js
function NameBox(name) {
  return { fontWeight: 'bold', labelContent: name };
}
```

```
'Sebastian Markbåge' ->
{ fontWeight: 'bold', labelContent: 'Sebastian Markbåge' };
```

## Abstraction

You can't fit a complex UI in a single function though. It is important that UIs can be abstracted into reusable pieces that don't leak their implementation details. Such as calling one function from another.

> **🇪🇸** No puedes encajar una interfaz compleja en una sola función. Es importante que las UIs puedan abstraerse en piezas reutilizables que no filtren los detalles de implementación. Por ejemplo, llamar a una función desde otra.

```js
function FancyUserBox(user) {
  return {
    borderStyle: '1px solid blue',
    childContent: [
      'Name: ',
      NameBox(user.firstName + ' ' + user.lastName)
    ]
  };
}
```

```
{ firstName: 'Sebastian', lastName: 'Markbåge' } ->
{
  borderStyle: '1px solid blue',
  childContent: [
    'Name: ',
    { fontWeight: 'bold', labelContent: 'Sebastian Markbåge' }
  ]
};
```

## Composition

To achieve truly reusable features, it is not enough to simply reuse leaves and build new containers for them. You also need to be able to build abstractions from the containers that *compose* other abstractions. The way I think about "composition" is that they're combining two or more different abstractions into a new one.

> **🇪🇸** Para lograr características verdaderamente reutilizables no basta con reutilizar hojas y construir nuevos contenedores para ellas. También necesitas poder construir abstracciones a partir de contenedores que *componen* otras abstracciones. Yo pienso en "composición" como la combinación de dos o más abstracciones distintas en una nueva.

```js
function FancyBox(children) {
  return {
    borderStyle: '1px solid blue',
    children: children
  };
}

function UserBox(user) {
  return FancyBox([
    'Name: ',
    NameBox(user.firstName + ' ' + user.lastName)
  ]);
}
```

## State

A UI is NOT simply a replication of server / business logic state. There is actually a lot of state that is specific to an exact projection and not others. For example, if you start typing in a text field. That may or may not be replicated to other tabs or to your mobile device. Scroll position is a typical example that you almost never want to replicate across multiple projections.

> **🇪🇸** Una UI NO es simplemente una réplica del estado de negocio/servidor. Hay mucho estado que es específico de una proyección concreta y no de otras. Por ejemplo, si empiezas a escribir en un campo de texto. Eso puede o no replicarse en otras pestañas o en tu móvil. La posición de desplazamiento (scroll) es un ejemplo típico que casi nunca quieres replicar entre múltiples proyecciones.

We tend to prefer our data model to be immutable. We thread functions through that can update state as a single atom at the top.

> **🇪🇸** Tendemos a preferir que nuestro modelo de datos sea inmutable. Encadenamos funciones que pueden actualizar el estado como un único átomo en la cima.

```js
function FancyNameBox(user, likes, onClick) {
  return FancyBox([
    'Name: ', NameBox(user.firstName + ' ' + user.lastName),
    'Likes: ', LikeBox(likes),
    LikeButton(onClick)
  ]);
}

// Implementation Details

var likes = 0;
function addOneMoreLike() {
  likes++;
  rerender();
}

// Init

FancyNameBox(
  { firstName: 'Sebastian', lastName: 'Markbåge' },
  likes,
  addOneMoreLike
);
```

*NOTE: These examples use side-effects to update state. My actual mental model of this is that they return the next version of state during an "update" pass. It was simpler to explain without that but we'll want to change these examples in the future.*

> **🇪🇸** *NOTA: Estos ejemplos usan efectos secundarios para actualizar el estado. Mi modelo mental real es que devuelven la siguiente versión del estado durante una pasada de "actualización". Fue más simple explicarlo sin eso, pero querríamos cambiar estos ejemplos en el futuro.*

## Memoization

Calling the same function over and over again is wasteful if we know that the function is pure. We can create a memoized version of a function that keeps track of the last argument and last result. That way we don't have to reexecute it if we keep using the same value.

> **🇪🇸** Llamar a la misma función una y otra vez es ineficiente si sabemos que la función es pura. Podemos crear una versión memoizada de una función que recuerde el último argumento y el último resultado. Así no tenemos que volver a ejecutarla si seguimos usando el mismo valor.

```js
function memoize(fn) {
  var cachedArg;
  var cachedResult;
  return function(arg) {
    if (cachedArg === arg) {
      return cachedResult;
    }
    cachedArg = arg;
    cachedResult = fn(arg);
    return cachedResult;
  };
}

var MemoizedNameBox = memoize(NameBox);

function NameAndAgeBox(user, currentTime) {
  return FancyBox([
    'Name: ',
    MemoizedNameBox(user.firstName + ' ' + user.lastName),
    'Age in milliseconds: ',
    currentTime - user.dateOfBirth
  ]);
}
```

## Lists

Most UIs are some form of lists that then produce multiple different values for each item in the list. This creates a natural hierarchy.

> **🇪🇸** La mayoría de las UIs son de alguna forma listas que luego producen múltiples valores distintos por cada elemento de la lista. Esto crea una jerarquía natural.

To manage the state for each item in a list we can create a Map that holds the state for a particular item.

> **🇪🇸** Para gestionar el estado de cada elemento de una lista podemos crear un Map que mantenga el estado para un elemento en particular.

```js
function UserList(users, likesPerUser, updateUserLikes) {
  return users.map(user => FancyNameBox(
    user,
    likesPerUser.get(user.id),
    () => updateUserLikes(user.id, likesPerUser.get(user.id) + 1)
  ));
}

var likesPerUser = new Map();
function updateUserLikes(id, likeCount) {
  likesPerUser.set(id, likeCount);
  rerender();
}

UserList(data.users, likesPerUser, updateUserLikes);
```

*NOTE: We now have multiple different arguments passed to FancyNameBox. That breaks our memoization because we can only remember one value at a time. More on that below.*

> **🇪🇸** *NOTA: Ahora tenemos múltiples argumentos distintos pasados a FancyNameBox. Eso rompe nuestra memoización porque solo podemos recordar un valor a la vez. Más sobre esto abajo.*

## Continuations

Unfortunately, since there are so many lists of lists all over the place in UIs, it becomes quite a lot of boilerplate to manage that explicitly.

> **🇪🇸** Desafortunadamente, dado que hay muchas listas de listas por todas partes en las UIs, se vuelve mucho boilerplate gestionarlo explícitamente.

We can move some of this boilerplate out of our critical business logic by deferring execution of a function. For example, by using "currying" ([`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) in JavaScript). Then we pass the state through from outside our core functions that are now free of boilerplate.

> **🇪🇸** Podemos mover parte de este boilerplate fuera de nuestra lógica de negocio crítica aplazando la ejecución de una función. Por ejemplo, usando "currying" ([`bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) en JavaScript). Entonces pasamos el estado desde fuera a nuestras funciones centrales que ahora están libres de ese boilerplate.

This isn't reducing boilerplate but is at least moving it out of the critical business logic.

> **🇪🇸** Esto no reduce el boilerplate, pero al menos lo mueve fuera de la lógica de negocio crítica.

```js
function FancyUserList(users) {
  return FancyBox(
    UserList.bind(null, users)
  );
}

const box = FancyUserList(data.users);
const resolvedChildren = box.children(likesPerUser, updateUserLikes);
const resolvedBox = {
  ...box,
  children: resolvedChildren
};
```

## State Map

We know from earlier that once we see repeated patterns we can use composition to avoid reimplementing the same pattern over and over again. We can move the logic of extracting and passing state to a low-level function that we reuse a lot.

> **🇪🇸** Sabemos por lo anterior que una vez que vemos patrones repetidos podemos usar composición para evitar reimplementar el mismo patrón una y otra vez. Podemos mover la lógica de extraer y pasar estado a una función de bajo nivel que reutilicemos mucho.

```js
function FancyBoxWithState(
  children,
  stateMap,
  updateState
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState
    ))
  );
}

function UserList(users) {
  return users.map(user => {
    continuation: FancyNameBox.bind(null, user),
    key: user.id
  });
}

function FancyUserList(users) {
  return FancyBoxWithState.bind(null,
    UserList(users)
  );
}

const continuation = FancyUserList(data.users);
continuation(likesPerUser, updateUserLikes);
```

## Memoization Map

Once we want to memoize multiple items in a list memoization becomes much harder. You have to figure out some complex caching algorithm that balances memory usage with frequency.

> **🇪🇸** Una vez que queremos memoizar múltiples elementos en una lista, la memoización se vuelve mucho más difícil. Tienes que idear algún algoritmo de caché complejo que equilibre el uso de memoria con la frecuencia.

Luckily, UIs tend to be fairly stable in the same position. The same position in the tree gets the same value every time. This tree turns out to be a really useful strategy for memoization.

> **🇪🇸** Por suerte, las UIs tienden a ser bastante estables en la misma posición. La misma posición en el árbol obtiene el mismo valor cada vez. Este árbol resulta ser una estrategia útil para la memoización.

We can use the same trick we used for state and pass a memoization cache through the composable function.

> **🇪🇸** Podemos usar el mismo truco que usamos para el estado y pasar una caché de memoización a través de la función componible.

```js
function memoize(fn) {
  return function(arg, memoizationCache) {
    if (memoizationCache.arg === arg) {
      return memoizationCache.result;
    }
    const result = fn(arg);
    memoizationCache.arg = arg;
    memoizationCache.result = result;
    return result;
  };
}

function FancyBoxWithState(
  children,
  stateMap,
  updateState,
  memoizationCache
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState,
      memoizationCache.get(child.key)
    ))
  );
}

const MemoizedFancyNameBox = memoize(FancyNameBox);
```

## Algebraic Effects

It turns out that it is kind of a PITA to pass every little value you might need through several levels of abstractions. It is kind of nice to sometimes have a shortcut to pass things between two abstractions without involving the intermediates. In React we call this "context".

> **🇪🇸** Resulta que es un fastidio pasar cada pequeño valor que puedas necesitar a través de varios niveles de abstracciones. A veces es agradable tener un atajo para pasar cosas entre dos abstracciones sin involucrar a los intermedios. En React llamamos a esto "context".

Sometimes the data dependencies don't neatly follow the abstraction tree. For example, in layout algorithms you need to know something about the size of your children before you can completely fulfill their position.

> **🇪🇸** A veces las dependencias de datos no siguen limpiamente el árbol de abstracción. Por ejemplo, en algoritmos de layout necesitas saber algo sobre el tamaño de tus hijos antes de poder completar su posición.

Now, this example is a bit "out there". I'll use [Algebraic Effects](http://math.andrej.com/eff/) as [proposed for ECMAScript](https://esdiscuss.org/topic/one-shot-delimited-continuations-with-effect-handlers). If you're familiar with functional programming, they're avoiding the intermediate ceremony imposed by monads.

> **🇪🇸** Ahora, este ejemplo es un poco "experimental". Usaré [Efectos algebraicos](http://math.andrej.com/eff/) como se [propuso para ECMAScript](https://esdiscuss.org/topic/one-shot-delimited-continuations-with-effect-handlers). Si estás familiarizado con la programación funcional, evitan la ceremonia intermedia impuesta por los mónados.

```js
function ThemeBorderColorRequest() { }

function FancyBox(children) {
  const color = raise new ThemeBorderColorRequest();
  return {
    borderWidth: '1px',
    borderColor: color,
    children: children
  };
}

function BlueTheme(children) {
  return try {
    children();
  } catch effect ThemeBorderColorRequest -> [, continuation] {
    continuation('blue');
  }
}

function App(data) {
  return BlueTheme(
    FancyUserList.bind(null, data.users)
  );
}
```
