### Подробное объяснение EventEmitter в Node.js

#### **1. Введение в EventEmitter**

EventEmitter — это объект в Node.js, предоставляющий систему для работы с событиями. Он решает задачи управления асинхронными операциями через событийно-ориентированный подход.

- **Проблема**: Необходимость эффективно реагировать на события (например, действия пользователя) без блокировки выполнения кода.
- **Решение**: EventEmitter позволяет "подписываться" на события и "вызывать" их, обеспечивая реактивность.

> **Важно**: EventEmitter реализован на чистом JavaScript и не связан напрямую с libuv, event loop или асинхронностью низкого уровня. Это паттерн для улучшения читаемости и управления кодом.

---

#### **2. Базовый пример использования**

Демонстрация минимальной работы с EventEmitter:

```javascript
// Импорт модуля events
const EventEmitter = require("events");

// Создание класса, наследующего EventEmitter
class Emitter extends EventEmitter {}
const myEmitter = new Emitter();

// Подписка на событие 'foo'
myEmitter.on("foo", () => {
  console.log("Событие произошло!");
});

// Вызов события 'foo'
myEmitter.emit("foo"); // Вывод: "Событие произошло!"
```

**Пояснения**:

- `require('events')` импортирует модуль EventEmitter.
- `class Emitter extends EventEmitter` наследует методы EventEmitter.
- `.on(eventName, callback)` регистрирует обработчик для события.
- `.emit(eventName)` вызывает все обработчики для указанного события.

---

#### **3. Принцип работы EventEmitter**

##### **Аналогия с операционной системой**

- **Синхронный подход**: Один ядро CPU постоянно опрашивает устройство (например, клавиатуру), что неэффективно.
- **Асинхронный подход (Event-Driven)**: Устройство отправляет прерывание (событие) при действии. CPU обрабатывает его без постоянного опроса.

##### **Архитектура EventEmitter**

- **Структура хранения**:
  ```javascript
  {
    'foo': [callback1, callback2, ...], // Массив функций для события 'foo'
    'bar': [callback3]                 // Массив функций для события 'bar'
  }
  ```
- **Методы**:
  - `.on('event', callback)` добавляет функцию в массив для указанного события.
  - `.emit('event')` вызывает все функции из массива события.

**Визуализация**:

```
emit('foo')
│
├─> Выполняет callback1()
├─> Выполняет callback2()
└─> ...
```

---

#### **4. Расширенный пример с параметрами и множеством обработчиков**

```javascript
const EventEmitter = require("events");
class Emitter extends EventEmitter {}
const myE = new Emitter();

// Регистрация нескольких обработчиков для 'foo'
myE.on("foo", () => {
  console.log("Событие 1");
});

myE.on("foo", () => {
  console.log("Событие 2");
});

myE.on("foo", (x) => {
  console.log(`Событие с параметром: ${x}`);
});

// Вызов события с параметром
myE.emit("foo", "Hello!");
```

**Вывод**:

```
Событие 1
Событие 2
Событие с параметром: Hello!
```

**Особенности**:

- Обработчики выполняются в порядке регистрации.
- Параметры передаются через `emit(eventName, ...args)`.

---

#### **5. Метод `once()`**

Регистрирует обработчик, который срабатывает **единожды** и автоматически удаляется после вызова:

```javascript
myE.once("bar", () => {
  console.log("Сработает только один раз!");
});

myE.emit("bar"); // Вывод: "Сработает только один раз!"
myE.emit("bar"); // Ничего не произойдет
```

**Принцип работы**:  
После первого вызова, обработчик удаляется из массива событий.

---

#### **6. Обработка ошибок**

Для отлова ошибок используется специальное событие `'error'`:

```javascript
myE.on("error", (err) => {
  console.error("Ошибка:", err.message);
});

myE.emit("error", new Error("Что-то сломалось!"));
// Вывод: "Ошибка: Что-то сломалось!"
```

> **Важно**: Если событие `'error'` не обработано, Node.js завершит процесс.

---

#### **7. Другие методы EventEmitter**

Основные методы из [документации Node.js](https://nodejs.org/api/events.html):

- `listenerCount(eventName)` возвращает количество обработчиков для события.
- `removeListener(eventName, callback)` удаляет конкретный обработчик.
- `eventNames()` возвращает массив имен зарегистрированных событий.

Пример:

```javascript
console.log(myE.listenerCount("foo")); // Вывод: 3
```

---

#### **8. Кастомная реализация EventEmitter**

Упрощенная реализация из статьи [How to Code Your Own Event Emitter](https://www.freecodecamp.org/news/how-to-code-your-own-event-emitter-in-node-js-a-step-by-step-guide-e13b7e7908e1/):

```javascript
class CustomEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }

  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach((listener) => {
        listener.apply(this, args);
      });
    }
  }

  once(event, listener) {
    const wrapper = (...args) => {
      listener.apply(this, args);
      this.removeListener(event, wrapper);
    };
    this.on(event, wrapper);
  }

  removeListener(event, targetListener) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(
        (listener) => listener !== targetListener
      );
    }
  }
}

module.exports = CustomEmitter;
```

**Использование**:

```javascript
const CustomEmitter = require("./CustomEmitter");
const emitter = new CustomEmitter();
// Далее как с нативным EventEmitter!
```

Разберём этот JavaScript-код **до атомов** — максимально подробно, построчно, со всеми внутренними механизмами JavaScript, включая память, область видимости, методы объекта, `this`, и т.д.

---

## 📦 Общий обзор

Этот код реализует **собственный класс событийного эмиттера** — аналога `EventEmitter` из Node.js. Такой объект может:

- подписывать слушателей на события (`on`)
- испускать события (`emit`)
- подписывать одноразовые слушатели (`once`)
- удалять слушателей (`removeListener`)

---

## 🔍 Подробный разбор

### \`\`\`js

class CustomEmitter {

````

- `class` — ключевое слово ES6 (ECMAScript 2015), определяет **класс**, то есть **шаблон** для создания объектов с определённой структурой и поведением (методами).
- `CustomEmitter` — имя класса (по соглашению с заглавной буквы).

---

### ```js
  constructor() {
    this.events = {};
  }
````

#### `constructor()`:

- Специальный метод, **автоматически вызывается** при создании нового объекта через `new CustomEmitter()`.
- `this` в контексте конструктора ссылается на **новосозданный объект**, который будет возвращён.

#### `this.events = {}`:

- Создаёт у каждого экземпляра **собственное** свойство `events` — это пустой объект.
- Он будет использоваться как **таблица событий**: ключи — имена событий, значения — массивы слушателей.

🔧 В памяти:

```js
{
  events: {
    /* eventName: [listeners] */
  }
}
```

---

## 📥 Метод `.on()`

```js
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }
```

### Что делает:

- Позволяет **добавить обработчик (listener)** на конкретное событие.

### Аргументы:

- `event` — имя события (строка).
- `listener` — функция, которую нужно вызвать при этом событии.

### Под капотом:

1. `this.events[event]` — обращение к объекту `events` по ключу (имени события).
2. `if (!this.events[event])` — если массив обработчиков не существует, создаём его.
3. `this.events[event].push(listener)` — добавляем обработчик в массив.

💡 В JavaScript объект `events` по сути — это **ассоциативный массив** (hash map), где ключ — это строка (имя события), а значение — массив функций.

---

## 📣 Метод `.emit()`

```js
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(listener => {
        listener.apply(this, args);
      });
    }
  }
```

### Что делает:

- **Испускает событие**, вызывая всех слушателей, которые подписаны на `event`.

### Аргументы:

- `event` — имя события.
- `...args` — любые дополнительные аргументы, передающиеся в обработчики.

### Детали:

- `if (this.events[event])` — проверяет, есть ли слушатели.
- `forEach(listener => ...)` — проходит по массиву слушателей и вызывает каждый.

### `listener.apply(this, args)`:

- `apply()` — встроенный метод функции в JS:

  - первый аргумент (`this`) — что будет `this` внутри вызываемой функции.
  - второй аргумент (`args`) — массив аргументов, которые передаются в обработчик.

- Здесь `this` передаётся в `apply()` чтобы обработчики могли использовать методы или свойства эмиттера.

---

## 🔂 Метод `.once()`

```js
  once(event, listener) {
    const wrapper = (...args) => {
      listener.apply(this, args);
      this.removeListener(event, wrapper);
    };
    this.on(event, wrapper);
  }
```

### Что делает:

- Регистрирует **одноразовый** обработчик события — он будет вызван только один раз.

### Механизм:

1. Создаёт **обёртку** `wrapper`, которая вызывает `listener`, а затем удаляет себя.
2. Обёртка запоминается в `this.on(event, wrapper)` как обычный обработчик.
3. После вызова `emit`, обёртка удаляет себя из массива.

🔍 Почему обёртка нужна:

- Чтобы можно было удалить именно `wrapper`, а не `listener`, так как `removeListener` сравнивает функции по ссылке.

---

## ❌ Метод `.removeListener()`

```js
  removeListener(event, targetListener) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(
        listener => listener !== targetListener
      );
    }
  }
```

### Что делает:

- Удаляет конкретного слушателя `targetListener` из массива по имени события `event`.

### Как:

- Проверяет, есть ли массив обработчиков для этого события.
- Использует `.filter()`:

  - Создаёт **новый массив**, исключая `targetListener`.
  - Сравнение идёт по **ссылке** на функцию (`===`).

🔧 Важно:

- Две функции в JS равны (`===`) **только если они одна и та же функция в памяти**.
- Именно поэтому `.once()` создаёт обёртку — чтобы иметь уникальную ссылку.

---

## 📤 Экспорт класса (Node.js)

```js
module.exports = CustomEmitter;
```

- `module.exports` — системный механизм Node.js.
- Делает `CustomEmitter` **доступным для импорта в других файлах** с помощью `require()`.

Пример:

```js
const CustomEmitter = require("./CustomEmitter");
const emitter = new CustomEmitter();
```

---

## 🧠 Вся структура в памяти

Если вызвать:

```js
const emitter = new CustomEmitter();

function greet(name) {
  console.log("Hello,", name);
}

emitter.on("sayHi", greet);
emitter.emit("sayHi", "Murad");
```

В памяти:

```js
emitter = {
  events: {
    sayHi: [greet],
  },
};
```

---

## 🧩 Вывод

| Метод            | Назначение                                     | Особенности                            |
| ---------------- | ---------------------------------------------- | -------------------------------------- |
| `constructor`    | Создаёт структуру хранения событий             | `this.events = {}`                     |
| `on`             | Регистрирует слушателя события                 | Сохраняет в массив по имени события    |
| `emit`           | Запускает всех слушателей события              | Передаёт им аргументы                  |
| `once`           | Регистрирует одноразового слушателя            | После первого срабатывания — удаляется |
| `removeListener` | Удаляет конкретного слушателя по имени события | Функции сравниваются по ссылке         |

---

Отличный вопрос! Это действительно важный момент — давай объясню **предельно подробно и наглядно**, зачем используется:

```js
listener.apply(this, args);
```

в методах `emit()` и `once()`.

---

## 📌 Что такое `Function.prototype.apply()`?

`apply()` — это встроенный метод любой функции в JavaScript.

Полная сигнатура:

```js
func.apply(thisArg, argsArray);
```

- **`thisArg`** — это то, что будет использоваться как `this` внутри `func`.
- **`argsArray`** — массив (или массивоподобный объект), элементы которого передаются как **аргументы** в `func`.

---

## 📦 Пример без `.apply()`

Представим у нас есть:

```js
function sayHello(name, age) {
  console.log(`${this.prefix} ${name}, age ${age}`);
}

const context = { prefix: "👋 Hello" };

sayHello("Murad", 30); // undefined Murad, age 30
```

`this.prefix` здесь будет `undefined`, потому что `this` — это `undefined` (в строгом режиме) или `window` (в браузере без strict mode).

Чтобы задать `this`, мы можем использовать `apply()`:

```js
sayHello.apply(context, ["Murad", 30]);
// 👋 Hello Murad, age 30
```

---

## 🧠 Возвращаемся к `listener.apply(this, args)`

Теперь — к нашему коду:

```js
listener.apply(this, args);
```

### ✅ Что здесь происходит:

- `listener` — это **функция-обработчик события** (которую кто-то добавил через `.on()` или `.once()`).
- `.apply(this, args)` вызывает её и:

  - передаёт в неё массив аргументов `args`, как отдельные значения.
  - устанавливает `this` внутри `listener` равным `this` текущего экземпляра `CustomEmitter`.

---

### 🔍 А что будет без `.apply()`?

Если бы ты написал просто:

```js
listener(args); // ❌
```

Ты бы передал **один аргумент — массив**, а не список значений.

А если бы написал:

```js
listener(...args); // ✅
```

Это сработает корректно — это современный аналог `.apply()`, но:

```js
listener(...args);
```

**не устанавливает `this`** — внутри `listener` `this` будет `undefined` (в strict mode) или глобальный объект.

---

## 🧪 Пример, где `this` действительно важен

```js
class CustomEmitter {
  constructor() {
    this.name = "CustomEmitterInstance";
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
  }

  emit(event, ...args) {
    this.events[event]?.forEach((listener) => {
      listener.apply(this, args);
    });
  }
}

const emitter = new CustomEmitter();

function myListener(data) {
  console.log(this.name); // ← вот здесь важно!
  console.log("Received:", data);
}

emitter.on("myEvent", myListener);
emitter.emit("myEvent", "someData");
```

### Вывод:

```
CustomEmitterInstance
Received: someData
```

Если бы ты написал:

```js
listener(...args);
```

То внутри `myListener` `this.name` стал бы `undefined`.

---

## ✅ Зачем `apply(this, args)` нужен:

| Что делает                       | Почему это важно                            |
| -------------------------------- | ------------------------------------------- |
| Вызывает `listener()`            | Срабатывает слушатель события               |
| Передаёт все аргументы из `emit` | Все данные события доходят до обработчика   |
| Устанавливает `this` на эмиттер  | Внутри `listener` можно использовать `this` |

---

## 📌 Подведём итог

```js
listener.apply(this, args);
```

- 🔁 Вызывает слушателя события (`listener`)
- 🔗 Передаёт ему аргументы (`args`) как обычные параметры, не массив
- 📍 Устанавливает `this` внутри слушателя на текущий `CustomEmitter`, чтобы слушатель мог обращаться к методам/свойствам эмиттера

---

---

#### **9. Реальный код EventEmitter в Node.js**

Код из [исходников Node.js (events.js)](https://github.com/nodejs/node/blob/master/lib/events.js):

- Основан на прототипах (не классах ES6).
- Содержит дополнительные проверки ошибок и оптимизации.
- Ключевой фрагмент метода `emit`:

  ```javascript
  EventEmitter.prototype.emit = function emit(type, ...args) {
    const events = this._events;
    const handler = events[type];
    if (handler === undefined) return false;

    if (typeof handler === "function") {
      handler.apply(this, args);
    } else {
      // Массив обработчиков
      const listeners = handler.slice();
      for (let i = 0; i < listeners.length; i++) {
        listeners[i].apply(this, args);
      }
    }
    return true;
  };
  ```

---

#### **10. Итоги**

- **EventEmitter** — ядро событийной модели Node.js.
- **Используется в**:
  - Модулях `stream`, `http`, `net`.
  - Популярных пакетах (например, Express.js).
- **Ключевые методы**: `.on()`, `.emit()`, `.once()`.
- **Важно**: Это паттерн JavaScript, а не часть асинхронности libuv.

> **Совет**: Для глубокого понимания изучите [исходный код EventEmitter](https://github.com/nodejs/node/blob/master/lib/events.js) и экспериментируйте с кастомной реализацией.
