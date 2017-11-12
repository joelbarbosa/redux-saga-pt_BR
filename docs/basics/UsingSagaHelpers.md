# Utilizando os Saga Helpers

`redux-saga` fornece alguns efeitos auxiliares envolvendo funções internas para gerar tarefas quando algumas ações específicas são enviadas para o Store.

As funções auxiliares são criadas em cima da API de nível inferior. Na seção avançada, veremos como essas funções podem ser implementadas.

A primeira função, `takeEvery` é a mais familiar e fornece um comportamento semelhante ao `redux-thunk`.

Vamos ilustrar com o exemplo comum do AJAX. Em cada clique no botão, enviamos uma ação `FETCH_REQUESTED`. Queremos lidar com esta ação, iniciando uma tarefa que irá buscar alguns dados do servidor.

Primeiro criamos a tarefa que executará a ação assíncrona:

```javascript
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```

Para iniciar a tarefa acima em cada ação `FETCH_REQUESTED`:

```javascript
import { takeEvery } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeEvery('FETCH_REQUESTED', fetchData)
}
```

No exemplo acima, `takeEvery` permite que várias instâncias `fetchData` sejam iniciadas simultaneamente. Em um momento dado, podemos iniciar uma nova tarefa `fetchData` enquanto ainda há uma ou mais tarefas `fetchData` anteriores que ainda não foram encerradas.

Se quisermos obter apenas a resposta do request mais recente (por exemplo, para exibir sempre a versão mais recente dos dados), podemos usar o auxiliar `takeLatest`:

```javascript
import { takeLatest } from 'redux-saga/effects'

function* watchFetchData() {
  yield takeLatest('FETCH_REQUESTED', fetchData)
}
```

Ao contrário do `takeEvery`, `takeLatest` permite que apenas uma tarefa `fetchData` seja executada a qualquer momento. E será a última tarefa iniciada. Se uma tarefa anterior ainda estiver sendo executada quando outra tarefa `fetchData` for iniciada, a tarefa anterior será automaticamente cancelada.

Se você tiver várias Sagas escutando diferentes ações, você pode criar vários observadores com as funções auxiliares que se comportarão como se houvesse um `fork` usado para gerá-los (falamos sobre `fork` mais tarde. Por agora considere que isso seja um efeito que nos permita iniciar múltiplas sagas em segundo plano)

Por exemplo:

```javascript
import { takeEvery } from 'redux-saga'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// use them in parallel
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```
