# Готовим Саги - Нагасаки

`Redux-saga` - это подход, позволяющий выполнять изменения в хранилище `Redux`, когда при получении `action` в `saga` уже содержатся `dispatch()` хранилища, готовые выполнится в том порядке в котором они описаны в `saga`.

# Поехали

Начнем с примера и задач, которые нужно было решать.

# Задачи

1. Асинхронные операции будут вызывать `action`, на который будет реагировать `saga`
2. Логику отправки асинхронных данных поместить в `saga`

# Пример

У нас есть поле `input` для ввода города и кнопка "добавить город", по нажатию на которую будет вызываться `action`.

```javascript
class CityInput extends Component {

  render() {
    const { handleSubmit } = this.props;

    return (
      <form className='row city-input' onSubmit={handleSubmit(this.handleFormSubmit)}>
        <Field
          name='cityName'
          component='input'
          className='u-full-width'
          type='text'
          placeholder='Enter city name'
          />
        <button
          className='button button-primary'
          type='submit'
        >
          Add city
        </button>
      </form>
    );
  }

  handleFormSubmit = (formProps) => {
    const cityName = formProps.cityName;
    if (cityName && cityName.length !== 0) {
      // Request new data to the API
      this.props.actions.fetchAddCity(cityName);
    }
    this.props.reset();
  }
}
```

`Action fetchAddCity` имеет следующий вид:

```javascript
export function fetchAddCity(name) {
  return {
    type: 'FETCH_CITY',
    amount: name
  }
}
```

Теперь нам нужна `saga`, которая будет срабатываться, как появиться `action FETCH_CITY`. 

#### Для этого нам нужно:

1. Создать `saga`, которая будет реагировать на `action` и вызывать нужные `dispatch()`
2. Подписать `saga` на `action`
3. Создать `saga middlevawe` для хранилища

С первым пунктом справляемся так:

```javascript
export function* fetchCity(action) {
  const argument = {
    type: 'weather', 
    settings: `?q=${action.amount}`,
  }
  const { response, error } = yield call(api.get, argument)
  if (response) {
      yield put(addCity(response))
  } else {
      console.log('error', error);
      alert(error);
  }
}
```

Подписка `saga` на `action` выглядит так:

```javascript
export function* mySagaCity() {
  yield [
    takeLatest('FETCH_CITY', fetchCity),
  ]
}
```

Третий пункт выглядит следующим образом:

```javascript
import { applyMiddleware } from 'redux';
import createSagaMiddleware from 'redux-saga'

// создаем saga мидлвар
const sagaMiddleware = createSagaMiddleware();
applyMiddleware(sagaMiddleware);

// запускаем сагу
sagaMiddleware.run(mySagaCity)
```

`Saga` есть ничто иное как функция-генератор в `javascript`, создается при помощи `function *`. Для того чтобы управлять вызовыми функций в генераторе есть `yield`, который остановливает и возобновляет функцию-генератор. Вызов отправки асинхроннго кода с нужными параметрами осуществляется с помощью `yield call()`. Когда данные будут получены, то функция-генератор возобновит свою работу и вызовет `dispatch()`, который в `saga` выглядит так `yield put()`.

`Action addCity` имеет следующий вид:

```javascript
export function addCity(city) {
  return {
    type: 'ADD_CITY',
    amount: city,
  }
}
```

В результате мы вызываем `dispatch()`, который позволяет нам добавить новый город в хранилище. Редьюсер тем временем очень прост и имеет следующий вид:

```javascript
function cities(state = [], action) {
  switch (action.type) {
    case 'ADD_CITY':
      return [...state, action.amount]
    default:
      return state;
  }
}
```
 
#### P.S.

[Исходный код и демо](https://github.com/s-kobets/weacher-spa).
