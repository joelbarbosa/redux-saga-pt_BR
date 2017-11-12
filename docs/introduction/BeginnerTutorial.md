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

Nós importamos `delay`, uma função de utilidade que retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) que se resolverá após um número especificado de milissegundos. Usaremos esta função para *bloquear* o gerador (Generator).

As sagas são implementadas como [Generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) que *produzem (yield)* objetos para o middleware do redux-saga. Os objetos produzidos (yielded) são um tipo de instrução a ser interpretada pelo middleware. Quando uma Promise é enviada ao middleware, o middleware irá suspender a Saga até que a Promise seja concluída. No exemplo acima, a saga `incrementAsync` é suspensa até que a Promise retornada pelo `delay` se resolva, o que acontecerá após 1 segundo.

Uma vez que a Promise for resolvida, o middleware retomará a Saga, executando o código até o próximo yield. Neste exemplo, a próxima declaração é outro objeto produzido (yielded): o resultado de chamar `put({type: 'INCREMENT'})`, que instrui o middleware a enviar a ação `INCREMENT`.

`put` é um exemplo do que chamamos de *Efeito*. Os efeitos são simples objetos do JavaScript que contêm instruções para serem cumpridas pelo middleware. Quando um middleware recupera um efeito produzido por uma Saga, a Saga é pausada até que o efeito seja cumprido.

Então, para resumir, a saga `incrementAsync` pausa por 1 segundo através da chamada do `delay(1000)`, então envia uma ação `INCREMENT`.

Em seguida, criamos outra Saga `watchIncrementAsync`. Usamos `takeEvery`, uma função auxiliar fornecida pelo `redux-saga`, para escutar as ações `INCREMENT_ASYNC` despachadas e executar `incrementAsync` cada vez.

Agora temos 2 Sagas, e precisamos iniciá-las ambas de uma só vez. Para fazer isso, adicionaremos um `rootSaga` que é responsável por iniciar nossas outras Sagas. No mesmo arquivo `sagas.js`, adicione o seguinte código:

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

Esta Saga produz uma matriz com os resultados ao chamar nossas duas sagas, `helloSaga` e `watchIncrementAsync`. Isso significa que os dois geradores (Generators) resultantes serão iniciados em paralelo. Agora, só temos que invocar `sagaMiddleware.run` na Saga raiz em `main.js`.

```javascript
// ...
import rootSaga from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = ...
sagaMiddleware.run(rootSaga)

// ...
```

## Tornando o nosso código testável

Queremos testar a nossa saga `incrementAsync` para certificar-se de que ela executa a tarefa desejada.

Crie outro arquivo `sagas.spec.js`:

```javascript
import test from 'tape';

import { incrementAsync } from './sagas'

test('incrementAsync Saga test', (assert) => {
  const gen = incrementAsync()

  // now what ?
});
```

`incrementAsync` é a função geradora. Quando executada, irá retornar um objeto iterador e o método `next` do iterador retorna um objeto com a seguinte forma:

```javascript
gen.next() // => { done: boolean, value: any }
```

O campo `value` contém a expressão produzida, isto é, o resultado da expressão após o `yield`. O campo `done` indica se o gerador terminou ou se ainda há mais expressões de 'yield'.

No caso de `incrementAsync`, o gerador produz 2 valores consecutivamente:

1. `yield delay(1000)`
2. `yield put({type: 'INCREMENT'})`

Então, se invocarmos o próximo método do gerador 3 vezes consecutivamente, obtemos so seguintes resultados:

```javascript
gen.next() // => { done: false, value: <result of calling delay(1000)> }
gen.next() // => { done: false, value: <result of calling put({type: 'INCREMENT'})> }
gen.next() // => { done: true, value: undefined }
```

As primeiras 2 invocações retornam os resultados das expressões de yield. Na 3ª invocação
uma vez que não há mais yield, o campo `done` está definido como verdadeiro. E como o gerador `incrementAsync`
não retorna nada (sem declaração `return`), o campo` value` está configurado para `undefined`.

Então, agora, para testar a lógica dentro de `incrementAsync`, simplesmente teremos que iterar
sobre o gerador (Generator) retornado e verifique os valores gerados pelo gerador.

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

A questão é como podemos testar o valor de retorno de `delay`? Não podemos fazer uma simples prova de igualdade
nas Promises. Se `delay` retornasse um valor *normal*, as coisas teriam sido mais fáceis de testar.

Bem, `redux-saga` fornece uma maneira de tornar possível a declaração acima. Em vez de chamar
`delay(1000)` diretamente dentro de `incrementAsync`, chamaremos *indiretamente*:

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

Ao invés de executar `yield delay(1000)`, nós agora estamos executando `yield call(delay, 1000)`. Qual a diferença?

No primeiro caso, a expressão de rendimento `delay(1000)` é avaliada antes de passar para o caller de `next` (o caller pode ser o middleware ao executar nosso código. Também pode ser o nosso código de teste que executa a função do gerador (Generator) e itera sobre o gerador retornado). Então, o que o caller obtém é a Promise, como no código de teste acima.

No segundo caso, a expressão de yield `call(delay, 1000)` é o que passa para o caller de `next`. `call`, como `put`, retorna um efeito que instrui o middleware a chamar uma determinada função com os argumentos fornecidos. Na verdade, nem `put` nem `call` executam qualquer dispatch ou chamada assíncrona por si só, eles simplesmente retornam objetos simples do JavaScript.

```javascript
put({type: 'INCREMENT'}) // => { PUT: {type: 'INCREMENT'} }
call(delay, 1000)        // => { CALL: {fn: delay, args: [1000]}}
```

O que acontece é que o middleware examina o tipo de cada efeito produzido, em seguida, decide como cumprir esse efeito. Se o tipo de efeito for um `PUT`, ele enviará uma ação para o Store. Se o efeito for um `CALL`, ele chamará a função dada.

Essa separação entre criação de efeitos e execução de efeitos possibilita testar o nosso gerador de forma surpreendentemente fácil:

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

Uma vez que `put` e `call` retornam objetos simples, podemos reutilizar as mesmas funções em nosso código de teste. E para testar a lógica do `incrementAsync`, simplesmente iteramos sobre o gerador e fazemos o testes utlizando a função `deepEqual` em seus valores.

Para executar o teste acima, execute:

```sh
$ npm test
```

que deve reportar os resultados no console.
