<img src='https://redux-saga.js.org/logo/0800/Redux-Saga-Logo-Landscape.png' alt='Redux Logo Landscape' width='800px'>

# redux-saga

[![Join the chat at https://gitter.im/yelouafi/redux-saga](https://badges.gitter.im/yelouafi/redux-saga.svg)](https://gitter.im/yelouafi/redux-saga?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![npm version](https://img.shields.io/npm/v/redux-saga.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga) [![CDNJS](https://img.shields.io/cdnjs/v/redux-saga.svg?style=flat-square)](https://cdnjs.com/libraries/redux-saga)
[![OpenCollective](https://opencollective.com/redux-saga/backers/badge.svg)](#backers)
[![OpenCollective](https://opencollective.com/redux-saga/sponsors/badge.svg)](#sponsors)

`redux-saga` é uma lib que visa fazer side effects(efeitos colaterais) (i.e. coisas assincronas como data fetching e coisas impuras como acessar o cache do browser) em aplicações React/Redux mais fácil e melhor.


O modelo mental é que uma saga é como uma thread separada em sua aplicação que é unicamente responsável por side effects. `redux-saga` é um redux middleware, que significa que essa thread pode ser estartada, pausada e cancelada na aplicação principal com normais redux actions, ela tem acesso a todo o state(estado) da aplicação redux e pode também dispatch(dispachar) ações.


Saga usa uma feature(característica) do ES6 chamada Generators para fazer fazer fluxos assincronos fáceis de ler, escrever e testar. *(Se você não está familirizada com elas [Aqui tem alguns links introdutórios](https://redux-saga.js.org/docs/ExternalResources.html))* Para fazê-los, fluxos assincronos veja como o padrão assincrono JavaScript code. (algo como `async`/`await`, mas generators tem coisas mais incríveis que nós precisamos)


Talvez você já tenha usado `redux-thunk` para lidar com sua busca de dados. Contrários ao redux thunk, você não acaba em callback hell, você pode testar fácilmente seus fluxos assincronos e suas actions continuam puras.

# Iniciando

## Instalar

```sh
$ npm install --save redux-saga
```
ou

```sh
$ yarn add redux-saga
```

Alternativamente, você pode usar o fornecido UMD build diretamente no `<script>` tag de uma página HTML. Veja [Esta sessão](#using-umd-build-in-the-browser).

## Exemplo Usando

Supomos que temos uma UI para buscar alguns dados dos usuários de um servidor remoto quando um botão é clicado. ( Breve resumo, iremos mostrar apenas ações disparados no código)

```javascript
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

O Component dispara uma simples action Object na Store. Criaremos uma Saga que escuta todos os `USER_FETCH_REQUESTED` actions e dispara uma chamada na API para buscar usuários.

#### `sagas.js`

```javascript
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// trabalho do Saga: irá disparar uma ação USER_FETCH_REQUESTED
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  Estarta fetchUser em cada ação disparada `USER_FETCH_REQUESTED`.
  Permite concorrentes buscar de usuários
*/
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  Alternativa você pode usar takeLatest.

  Não permite buscar concorrentes de usuários. Se "USER_FETCH_REQUESTED" captura dispachados enquanto uma busca já está pendente, esta busca pendente é chamada e só a mais recente irá executar.
*/
function* mySaga() {
  yield takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

Para executar nosso Saga, teremos que conectar isto ao Redux Store usando o `redux-saga` middleware

#### `main.js`

```javascript
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// criação do saga middleware
const sagaMiddleware = createSagaMiddleware()
// agrupe na store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// então execute o saga
sagaMiddleware.run(mySaga)

// renderiza a aplicação
```

# Documentação

- [Introdução](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)
- [Conceitos Básico](https://redux-saga.js.org/docs/basics/index.html)
- [Conceitos Avançados](https://redux-saga.js.org/docs/advanced/index.html)
- [Recipes](https://redux-saga.js.org/docs/recipes/index.html)
- [Recursos Externos](https://redux-saga.js.org/docs/ExternalResources.html)
- [Soluções de Problemas](https://redux-saga.js.org/docs/Troubleshooting.html)
- [Glossário](https://redux-saga.js.org/docs/Glossary.html)
- [Refência da API](https://redux-saga.js.org/docs/api/index.html)

# Traduções

- [Chinês](https://github.com/superRaytin/redux-saga-in-chinese)
- [Chinês Tradicional](https://github.com/neighborhood999/redux-saga)
- [Japonês](https://github.com/redux-saga/redux-saga/blob/master/README_ja.md)
- [Coreano](https://github.com/mskims/redux-saga-in-korean)
- [Russo](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)
- [Português](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)

# Usando a build umd no browser

Exibe também uma build **umd** do `redux-saga` disponível no diretório `dist/`. Ao usar a build do umd `redux-saga` está disponível como `ReduxSaga` no objeto window. Isso permite que você crie o middleware Saga sem usar a sintaxe ES6 `import`, assim:

```javascript
var sagaMiddleware = ReduxSaga.default()
```

A versão umd é útil caso você não utilize Webpack ou Browserify. Você pode acessá-la diretamente através do [unpkg](https://unpkg.com/).

As seguintes builds estão disponíveis:

- [https://unpkg.com/redux-saga/dist/redux-saga.js](https://unpkg.com/redux-saga/dist/redux-saga.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.min.js](https://unpkg.com/redux-saga/dist/redux-saga.min.js)

**Importante!** Se o browser que você está trabalhando não suporta os *ES2015 generators*, você deve fornecer um polyfill válido, como o [fornecido pelo `babel`](https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.25/browser-polyfill.min.js). O polyfill deve ser importado antes do **redux-saga**:

```javascript
import 'babel-polyfill'
// then
import sagaMiddleware from 'redux-saga'
```

# Construindo exemplos a partir do código fonte

```sh
$ git clone https://github.com/redux-saga/redux-saga.git
$ cd redux-saga
$ npm install
$ npm test
```

Abaixo há os exemplos portados (até agora) dos repositórios do Redux.

### exemplos de contadores

Há três exemplos de contadores.

#### counter-vanilla

Exemplo usando Javascript puro e build UMD. Todo o código está inline no arquivo `index.html`.

Para executar o exemplo, basta abrir `index.html` no seu browser.

> Importante: seu browser deve suportar Generators. A última versão do Chrome/Firefox/Edge oferecem suporte.

#### counter

Exemplo usando `webpack` e a API de alto nível `takeEvery`.

```sh
$ npm run counter

# test sample for the generator
$ npm run test-counter
```

#### cancellable-counter

Exemplo usando uma API de baixo nível para demonstrar cancelamento de tarefas.

```sh
$ npm run cancellable-counter
```

### exemplo do carrinho de compras

```sh
$ npm run shop

# test sample for the generator
$ npm run test-shop
```

### exemplo async

```sh
$ npm run async

# test sample for the generators
$ npm run test-async
```

### exemplo real-world (com webpack hot reloading)

```sh
$ npm run real-world

# sorry, no tests yet
```

### TypeScript

Redux-Saga com TypeScript requer `DOM.Iterable` ou `ES2015.Iterable`. Se o seu `target` é `ES6`, provavelmente você não precisa fazer nada, mas para `ES5` você precisará adicionar isso você mesmo.
Verifique seu `tsconfig.json` e a documentação oficial para <a href="https://www.typescriptlang.org/docs/handbook/compiler-options.html">compiler options</a>.

### Logo

Você pode achar o logo oficial do Redux-Saga com diferentes versões no [diretório de logos](logo).


### Apoiadores
Apoie-nos com uma doação mensal e ajude-nos a continuar nossas atividades. \[[Tornar-se um apoiador](https://opencollective.com/redux-saga#backer)\]

<a href="https://opencollective.com/redux-saga/backer/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/29/avatar.svg"></a>


### Patrocinadores
Torne-se um patrocinador e obtenha seu logotipo em nosso README no Github com um link para seu site. \[[Tornar-se um patrocinador](https://opencollective.com/redux-saga#sponsor)\]

<a href="https://opencollective.com/redux-saga/sponsor/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/29/avatar.svg"></a>
