# Обработка событий

<VideoLesson href="https://vueschool.io/lessons/user-events-in-vue-3?friend=vuejs" title="Узнайте как обрабатывать события на Vue School">Узнайте как обрабатывать события в бесплатном уроке Vue School</VideoLesson>

## Прослушивание событий

Можно использовать директиву `v-on`, которую обычно сокращают до символа `@`, чтобы прослушивать события DOM и запускать какой-нибудь JavaScript-код по их наступлению. Используется как `v-on:click="methodName"` или в сокращённом виде `@click="methodName"`.

Например:

```html
<div id="basic-event">
  <button @click="counter += 1">Добавить 1</button>
  <p>Кнопка выше была нажата {{ counter }} раз</p>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      counter: 0
    }
  }
}).mount('#basic-event')
```

Результат:

<common-codepen-snippet title="Обработка событий: основы" slug="xxGadPZ" tab="result" :preview="false" />

## Обработчики событий

Логика многих обработчиков событий будет довольно сложной, поэтому оставлять JavaScript-код в значении атрибута `v-on` бессмысленно. Вот почему `v-on` также принимает имя метода, который потребуется вызвать.

Например:

```html
<div id="event-with-method">
  <!-- `greet` — это название метода, объявленного ниже -->
  <button @click="greet">Поприветствовать</button>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      name: 'Vue.js'
    }
  },
  methods: {
    greet(event) {
      // `this` в методе указывает на текущий активный экземпляр
      alert('Привет, ' + this.name + '!')
      // `event` — нативное событие DOM
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
}).mount('#event-with-method')
```

Результат:

<common-codepen-snippet title="Обработка событий: использование метода" slug="jOPvmaX" tab="result" :preview="false" />

## Методы в inline-обработчиках

Кроме указания имени метода, можно также использовать методы в инлайн-выражениях:

```html
<div id="inline-handler">
  <button @click="say('hi')">Скажи hi</button>
  <button @click="say('what')">Скажи what</button>
</div>
```

```js
Vue.createApp({
  methods: {
    say(message) {
      alert(message)
    }
  }
}).mount('#inline-handler')
```

Результат:

<common-codepen-snippet title="Обработка событий: метод в инлайн-выражении" slug="WNvgjda" tab="result" :preview="false" />

Иногда может потребоваться доступ к оригинальному событию DOM в inline-обработчиках. В этом случае его можно передать в метод, используя специальную переменную `$event`:

```html
<button @click="warn('Форма не может быть отправлена.', $event)">
  Отправить
</button>
```

```js
// ...
methods: {
  warn(message, event) {
    // теперь есть доступ к нативному событию
    if (event) {
      event.preventDefault()
    }

    alert(message)
  }
}
```

## Несколько обработчиков события

В обработчике событий можно задать несколько методов, разделив их оператором запятой:

```html
<!-- методы one() и two() будут вызваны при клике -->
<button @click="one($event), two($event)">
  Отправить
</button>
```

```js
// ...
methods: {
  one(event) {
    // логика первого обработчика события...
  },
  two(event) {
    // логика второго обработчика события...
  }
}
```

## Модификаторы событий

Часто может потребоваться вызвать `event.preventDefault()` или `event.stopPropagation()` внутри обработчиков события. Хоть это и легко можно сделать внутри методов, лучше если методы будут содержать в себе только логику и не имеют дела с деталями события DOM.

Для решения этой задачи Vue предоставляет **модификаторы событий** для `v-on`. Вспомните, что модификаторы — это постфиксы директивы, отделяемые точкой:

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- всплытие события click будет остановлено -->
<a @click.stop="doThis"></a>

<!-- событие submit перестанет перезагружать страницу -->
<form @submit.prevent="onSubmit"></form>

<!-- модификаторы можно объединять в цепочки -->
<a @click.stop.prevent="doThat"></a>

<!-- можно использовать без обработчиков -->
<form @submit.prevent></form>

<!-- можно отслеживать события в режиме capture, т.е. событие, нацеленное -->
<!-- на внутренний элемент, обрабатывается здесь до обработки этим элементом -->
<div @click.capture="doThis">...</div>

<!-- вызов обработчика только в случае наступления события непосредственно -->
<!-- на данном элементе (то есть не на дочернем компоненте) -->
<div @click.self="doThat">...</div>
```

:::tip Совет
При использовании модификаторов имеет значение их порядок, потому что в той же очерёдности генерируется соответствующий код. Поэтому `@click.prevent.self` предотвратит **действие клика по умолчанию на самом элементе и на его дочерних элементах**, в то время как `@click.self.prevent` предотвратит действие клика по умолчанию только на самом элементе.
:::

```html
<!-- обработчик click будет вызван максимум 1 раз -->
<a @click.once="doThis"></a>
```

В отличие от других модификаторов, которые поддерживают только нативные события DOM, модификатор `.once` можно использовать и с [пользовательскими событиями компонентов](component-custom-events.md). Если ещё не читали о компонентах — не беспокойтесь об этом пока.

Также Vue предоставляет модификатор `.passive`, соответствующий опции [`passive` для `addEventListener`](https://developer.mozilla.org/ru/docs/Web/API/EventTarget/addEventListener#Syntax).

```html
<!-- по умолчанию событие scroll (при прокрутке) произойдёт -->
<!-- незамедлительно, вместо ожидания окончания `onScroll`  -->
<!-- на случай, если там будет `event.preventDefault()`     -->
<div @scroll.passive="onScroll">...</div>
```

Модификатор `.passive` особенно полезен для улучшения производительности на мобильных устройствах.

:::tip Совет
Не используйте `.passive` и `.prevent` вместе — `.prevent` будет проигнорирован и браузер скорее всего покажет предупреждение. Запомните, что `.passive` сообщает браузеру, что для события _не будет предотвращаться поведение по умолчанию_.
:::

## Модификаторы клавиш

При прослушивании событий клавиатуры часто нужно отслеживать конкретные клавиши. Vue позволяет использовать модификаторы клавиш при использовании `v-on` или `@` при прослушивании событий клавиш:

```html
<!-- вызвать `vm.submit()` только если `key` будет `Enter` -->
<input @keyup.enter="submit" />
```

Можно напрямую использовать любые допустимые имена клавиш, предоставляемые через [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) в качестве модификаторов, указывая их имена в kebab-case формате.

```html
<input @keyup.page-down="onPageDown" />
```

В примере выше обработчик будет вызван только когда `$event.key` будет `'PageDown'`.

### Псевдонимы клавиш

Vue предоставляет псевдонимы для наиболее часто используемых клавиш:

- `.enter`
- `.tab`
- `.delete` (ловит как «Delete», так и «Backspace»)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

## Системные модификаторы клавиш

Можно использовать следующие модификаторы для прослушивания событий мыши или клавиатуры только при зажатой клавиши-модификатора:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

:::tip Примечание
На клавиатурах Apple клавиша meta отмечена знаком ⌘. На клавиатурах Windows клавиша meta отмечена знаком ⊞. На клавиатурах Sun Microsystems клавиша meta отмечена символом ромба ◆. На некоторых клавиатурах, особенно MIT и Lisp machine и их преемников, таких как Knight или space-cadet клавиатуры, клавиша meta отмечена словом «META». На клавиатурах Symbolics, клавиша meta отмечена словом «META» или «Meta».
:::

Например:

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Сделать что-нибудь</div>
```

:::tip Совет
Обратите внимание, клавиши-модификаторы отличаются от обычных клавиш и при отслеживании событий `keyup` должны быть нажаты, когда событие происходит. Другими словами, `keyup.ctrl` будет срабатывать только если отпустить клавишу, удерживая нажатой `ctrl`. Это не сработает, если отпустить только клавишу `ctrl`.
:::

### Модификатор `.exact`

Модификатор `.exact` позволяет контролировать точную комбинацию системных модификаторов, необходимых для запуска события.

```html
<!-- сработает, даже если также нажаты Alt или Shift -->
<button @click.ctrl="onClick">A</button>

<!-- сработает, только когда нажат Ctrl и не нажаты никакие другие клавиши -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- сработает, только когда не нажаты никакие системные модификаторы -->
<button @click.exact="onClick">A</button>
```

### Модификаторы клавиш мыши

- `.left`
- `.right`
- `.middle`

Эти модификаторы ограничивают обработчик события только вызовами определённой кнопкой мыши.

## Почему слушатели событий указываются в HTML?

Может показаться, что такой подход к прослушиванию событий нарушает старое доброе правило «отделения мух от котлет». Не волнуйтесь — поскольку во Vue все обработчики и выражения строго привязаны к ViewModel, ответственной за текущее представление, то и трудностей в поддержке не возникнет. На самом деле, есть несколько преимуществ при использовании `v-on` или `@`:

1. Проще находить реализацию обработчика в JS-коде, пропустив HTML-шаблон.

2. Поскольку не нужно вручную прикреплять прослушиватели событий в JS, код ViewModel остаётся независимым от DOM и содержит только необходимую логику. Это облегчает тестирование.

3. При уничтожении ViewModel все слушатели событий автоматически удаляются. Не надо беспокоиться о том, что нужно очищать что-то самостоятельно.
