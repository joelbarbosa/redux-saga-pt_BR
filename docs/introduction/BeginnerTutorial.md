# Tutorial para iniciantes

## Objetivos deste tutorial

Este tutorial tenta introduzir o redux-saga de uma maneira (esperamos que) acessível.

Para o nosso tutorial inicial, vamos usar a demonstração trivial do Counter do repositório Redux. O aplicativo é bastante simples, mas é um bom exemplo para ilustrar os conceitos básicos de redux-saga sem se perder em detalhes excessivos.

### A configuração inicial

Antes de iniciarmos, clone o [repositório tutorial](https://github.com/redux-saga/redux-saga-beginner-tutorial).

> O código final deste tutorial está presente no branch `sagas`.

Então na linha de comando, execute:

```sh
$ cd redux-saga-beginner-tutorial
$ npm install
```

Para iniciar a aplicação, execute:

```sh
$ npm start
```

Estamos começando com o caso de uso mais simples: 2 botões para `Increment` e `Decrement` um contador. Mais tarde, vamos apresentar chamadas assíncronas.

Se as coisas estiverem bem, você deve ver 2 botões `Increment` e `Decrement` junto com uma mensagem abaixo mostrando `Counter: 0`.

> Caso você tenha encontrado algum problema executando a aplicação. Sinta-se livre para criar uma issue no [repositório tutorial](https://github.com/redux-saga/redux-saga-beginner-tutorial/issues).

## Olá Sagas!

Nós vamos criar nossa primeira Saga. Seguindo a tradição, nós iremos escrever a nossa própria versão Saga de 'Hello, world'.

Crie um arquivo `sagas.js` e adicione o seguinte trecho:

```javascript
export function* helloSaga() {
  console.log('Hello Sagas!')
}
```

Então, nada de assustador, apenas uma função normal (exceto o `*`). Tudo o que faz é imprimir uma mensagem de saudação no console.

Para executar a nossa Saga, precisamos:

- criar um middleware Saga com uma lista de Sagas para executar (até agora temos apenas um `helloSaga`)
- conectar o middleware Saga com o store do Redux

Vamos fazer as alterações no `main.js`:

```javascript
// ...
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

// ...
import { helloSaga } from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(helloSaga)

const action = type => store.dispatch({type})

// rest unchanged
```

Primeiro importamos nossa Saga do módulo `./sagas`. Em seguida, criamos um middleware usando a função `createSagaMiddleware` exportada pela biblioteca `redux-saga`.

Antes de executar nosso `helloSaga`, devemos conectar nosso middleware ao Store usando `applyMiddleware`. Então podemos usar o `sagaMiddleware.run(helloSaga)` para iniciar nossa Saga.

Até agora, nossa Saga não faz nada de especial. Ela apenas registra uma mensagem e sai.

## Fazendo chamadas assíncronas

Agora vamos adicionar algo mais perto da demo original do Counter. Para ilustrar chamadas assíncronas, adicionaremos outro botão para incrementar o contador 1 segundo após o clique.

A primeira coisa que devemos fazer é fornecer um botão adicional e um callback `onIncrementAsync` ao componente UI.

```javascript
const Counter = ({ value, onIncrement, onDecrement, onIncrementAsync }) =>
  <div>
    <button onClick={onIncrementAsync}>
      Increment after 1 second
    </button>
    {' '}
    <button onClick={onIncrement}>
      Increment
    </button>
    {' '}
    <button onClick={onDecrement}>
      Decrement
    </button>
    <hr />
    <div>
      Clicked: {value} times
    </div>
  </div>
```

Em seguida, devemos conectar o `onIncrementAsync` do componente a uma action do Store.

Vamos modificar o módulo `main.js` da seguinte forma

```javascript
function render() {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')}
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}
```

Note que, ao contrário do redux-thunk, nosso componente envia uma ação de objeto simples.

Agora vamos apresentar outra Saga para executar a chamada assíncrona. Nosso caso de uso é o seguinte:

> Em cada ação `INCREMENT_ASYNC`, queremos iniciar uma tarefa que fará o seguinte

> - Aguarde 1 segundo e, em seguida, incremente o contador

Adicione o seguinte código ao módulo `sagas.js`:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery } from 'redux-saga/effects'

// ...

// Our worker Saga: will perform the async increment task
export function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}

// Our watcher Saga: spawn a new incrementAsync task on each INCREMENT_ASYNC
export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
```

Hora de dar algumas explicações.

Nós importamos `delay`, uma função de utilidade que retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que se resolverá após um número especificado de milissegundos. Usaremos esta função para *bloquear* o Generator.

As sagas são implementadas como [Generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) que *produzem (yield)* objetos para o middleware do redux-saga. Os objetos produzidos (yielded) são um tipo de instrução a ser interpretada pelo middleware. Quando uma Promise é enviada ao middleware, o middleware irá suspender a Saga até que a Promise seja concluída. No exemplo acima, a saga `incrementAsync` é suspensa até que a Promise retornada pelo `delay` se resolva, o que acontecerá após 1 segundo.

Once the Promise is resolved, the middleware will resume the Saga, executing code until the next yield. In this example, the next statement is another yielded object: the result of calling `put({type: 'INCREMENT'})`, which instructs the middleware to dispatch an `INCREMENT` action.

`put` is one example of what we call an *Effect*. Effects are simple JavaScript objects which contain instructions to be fulfilled by the middleware. When a middleware retrieves an Effect yielded by a Saga, the Saga is paused until the Effect is fulfilled.

So to summarize, the `incrementAsync` Saga sleeps for 1 second via the call to `delay(1000)`, then dispatches an `INCREMENT` action.

Next, we created another Saga `watchIncrementAsync`. We use `takeEvery`, a helper function provided by `redux-saga`, to listen for dispatched `INCREMENT_ASYNC` actions and run `incrementAsync` each time.

Now we have 2 Sagas, and we need to start them both at once. To do that, we'll add a `rootSaga` that is responsible for starting our other Sagas. In the same file `sagas.js`, add the following code:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery, all } from 'redux-saga/effects'

// ...

// single entry point to start all Sagas at once
export default function* rootSaga() {
  yield all([
    helloSaga(),
    watchIncrementAsync()
  ])
}
```

This Saga yields an array with the results of calling our two sagas, `helloSaga` and `watchIncrementAsync`. This means the two resulting Generators will be started in parallel. Now we only have to invoke `sagaMiddleware.run` on the root Saga in `main.js`.

```javascript
// ...
import rootSaga from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = ...
sagaMiddleware.run(rootSaga)

// ...
```

## Making our code testable

We want to test our `incrementAsync` Saga to make sure it performs the desired task.

Create another file `sagas.spec.js`:

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  // now what ?
});
```

`incrementAsync` is a generator function. When run, it returns an iterator object, and the iterator's `next` method returns an object with the following shape

```javascript
gen.next() // => { done: boolean, value: any }
```

The `value` field contains the yielded expression, i.e. the result of the expression after
the `yield`. The `done` field indicates if the generator has terminated or if there are still
more 'yield' expressions.

In the case of `incrementAsync`, the generator yields 2 values consecutively:

1. `yield delay(1000)`
2. `yield put({type: 'INCREMENT'})`

So if we invoke the next method of the generator 3 times consecutively we get the following
results:

```javascript
gen.next() // => { done: false, value: <result of calling delay(1000)> }
gen.next() // => { done: false, value: <result of calling put({type: 'INCREMENT'})> }
gen.next() // => { done: true, value: undefined }
```

The first 2 invocations return the results of the yield expressions. On the 3rd invocation
since there is no more yield the `done` field is set to true. And since the `incrementAsync`
Generator doesn't return anything (no `return` statement), the `value` field is set to
`undefined`.

So now, in order to test the logic inside `incrementAsync`, we'll simply have to iterate
over the returned Generator and check the values yielded by the generator.

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next(),
    { done: false, value: ??? },
    'incrementAsync should return a Promise that will resolve after 1 second'
  )
});
```

The issue is how do we test the return value of `delay`? We can't do a simple equality test
on Promises. If `delay` returned a *normal* value, things would've been easier to test.

Well, `redux-saga` provides a way to make the above statement possible. Instead of calling
`delay(1000)` directly inside `incrementAsync`, we'll call it *indirectly*:

```javascript
import { delay } from 'redux-saga'
import { put, takeEvery, all, call } from 'redux-saga/effects'

// ...

export function* incrementAsync() {
  // use the call Effect
  yield call(delay, 1000)
  yield put({ type: 'INCREMENT' })
}
```

Instead of doing `yield delay(1000)`, we're now doing `yield call(delay, 1000)`. What's the difference?

In the first case, the yield expression `delay(1000)` is evaluated before it gets passed to the caller of `next` (the caller could be the middleware when running our code. It could also be our test code which runs the Generator function and iterates over the returned Generator). So what the caller gets is a Promise, like in the test code above.

In the second case, the yield expression `call(delay, 1000)` is what gets passed to the caller of `next`. `call` just like `put`, returns an Effect which instructs the middleware to call a given function with the given arguments. In fact, neither `put` nor `call` performs any dispatch or asynchronous call by themselves, they simply return plain JavaScript objects.

```javascript
put({type: 'INCREMENT'}) // => { PUT: {type: 'INCREMENT'} }
call(delay, 1000)        // => { CALL: {fn: delay, args: [1000]}}
```

What happens is that the middleware examines the type of each yielded Effect then decides how to fulfill that Effect. If the Effect type is a `PUT` then it will dispatch an action to the Store. If the Effect is a `CALL` then it'll call the given function.

This separation between Effect creation and Effect execution makes it possible to test our Generator in a surprisingly easy way:

```javascript
import test from 'tape';

import { put, call } from 'redux-saga/effects'
import { delay } from 'redux-saga'
import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  assert.deepEqual(
    gen.next().value,
    call(delay, 1000),
    'incrementAsync Saga must call delay(1000)'
  )

  assert.deepEqual(
    gen.next().value,
    put({type: 'INCREMENT'}),
    'incrementAsync Saga must dispatch an INCREMENT action'
  )

  assert.deepEqual(
    gen.next(),
    { done: true, value: undefined },
    'incrementAsync Saga must be done'
  )

  assert.end()
});
```

Since `put` and `call` return plain objects, we can reuse the same functions in our test code. And to test the logic of `incrementAsync`, we simply iterate over the generator and do `deepEqual` tests on its values.

In order to run the above test, run:

```sh
$ npm test
```

which should report the results on the console.
