---
title: Входные параметры
type: guide
order: 102
---

> Подразумевается, что вы уже изучили и разобрались с разделом [Основы компонентов](components.html). Если нет — прочитайте его сначала.

## Стиль именования входных параметров (camelCase vs kebab-case)

Имена HTML-атрибутов являются регистро-независимыми, поэтому браузеры интерпретируют любые прописные символы как строчные. Это означает, что при использовании шаблонов в DOM, входные параметры в camelCase стиле должны использовать свои эквиваленты в стиле kebab-case (разделённые дефисами):

``` js
Vue.component('blog-post', {
  // camelCase в JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

``` html
<!-- kebab-case в HTML -->
<blog-post post-title="hello!"></blog-post>
```

Опять же, если вы используете строковые шаблоны, то это ограничение не применяется.

## Статические и динамические входные параметры

До сих пор вы встречали, что во входные параметры передавались статические значения, например:

```html
<blog-post title="My journey with Vue"></blog-post>
```

Вы также встречали входные параметры, присваивающие динамическое значение с помощью `v-bind`, например:

```html
<blog-post v-bind:title="post.title"></blog-post>
```

В этих двух примерах мы передаём строковые значения, но могут передаваться значения _любого типа_ во входной параметр.

### Передача чисел

```html
<!-- Несмотря на то, что `42` статическое значение, нам нужен v-bind, -->
<!-- чтобы сообщить Vue, что это выражение JavaScript, а не строка.   -->
<blog-post v-bind:likes="42"></blog-post>

<!-- Динамическое присвоение значения переменной. -->
<blog-post v-bind:likes="post.likes"></blog-post>
```

### Передача булевых значений

```html
<!-- Указание входного параметра без значения будет означать `true`. -->
<blog-post favorited></blog-post>

<!-- Несмотря на то, что `false` статическое значение, нам нужен v-bind -->
<!-- чтобы сообщить Vue, что это выражение JavaScript, а не строка.     -->
<base-input v-bind:favorited="false">

<!-- Динамическое присвоение значения переменной. -->
<base-input v-bind:favorited="post.currentUserFavorited">
```

### Передача массивов

```html
<!-- Несмотря на то, что указан статический массив, нам нужен v-bind -->
<!-- чтобы сообщить Vue, что это выражение JavaScript, а не строка.  -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- Динамическое присвоение значения переменной. -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

### Передача объектов

```html
<!-- Несмотря на то, что указан статический объект, нам нужен v-bind -->
<!-- чтобы сообщить Vue, что это выражение JavaScript, а не строка.  -->
<blog-post v-bind:comments="{ id: 1, title: 'My Journey with Vue' }"></blog-post>

<!-- Динамическое присвоение значения переменной. -->
<blog-post v-bind:post="post"></blog-post>
```

### Передача свойств объекта

Если вы хотите передать все свойства объекта в качестве входных параметров, вы можете использовать `v-bind` без аргументов (`v-bind` вместо `v-bind:prop-name`). Например, для объекта `post`:

``` js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

Следующий шаблон:

``` html
<blog-post v-bind="post"></blog-post>
```

Будет аналогичен:

``` html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

## Однонаправленный поток данных

Все входные параметры образуют **одностороннюю привязку** между дочерним свойством и родительским: когда родительское свойство обновляется — оно будет передаваться дочернему, но не наоборот. Это предотвращает случайное изменение дочерними компонентами родительского состояния, что может затруднить понимание потока данных вашего приложения.

Кроме того, каждый раз, когда обновляется родительский компонент, все входные параметры дочернего компонента будут обновлены актуальными значениями. Это означает, что вы **не должны** пытаться изменять входной параметр внутри дочернего компонента. Если вы это сделаете, Vue отобразит предупреждение в консоли.

Обычно встречаются два случая, когда возникает соблазн изменять входной параметр:

1. **Входной параметр используется для передачи начального значения; дочерний компонент хочет использовать его как локальное свойство данных в дальнейшем.** В этом случае лучше всего определить локальное свойство в данных, которое использует значение входного параметра в качестве начального:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```

2. **Входной параметр передаётся как необработанное значение, которое необходимо преобразовать.** В этом случае лучше всего определить вычисляемое свойство с использованием входного параметра:

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Обратите внимание, что объекты и массивы в JavaScript передаются по ссылке, поэтому если входной параметр является массивом или объектом, то изменение объекта или массива внутри дочернего компонента **будет влиять** на состояние родителя.</p>

## Валидация входных параметров

Компоненты могут указывать требования к своим входным параметрам. Если эти требования не выполнены — Vue предупредит вас сообщением в JavaScript-консоли браузера. Это особенно полезно при разработке компонента, который предназначен для использования другими.

Чтобы указать валидации входного параметра, вы можете предоставить в `props` объект с валидациями для проверки значения, вместо массива строк. Например:

``` js
Vue.component('my-component', {
  props: {
    // Просто проверка типа (`null` означает любой тип)
    propA: Number,
    // Несколько допустимых типов
    propB: [String, Number],
    // Обязательное значение строкового типа
    propC: {
      type: String,
      required: true
    },
    // Число со значением по умолчанию
    propD: {
      type: Number,
      default: 100
    },
    // Объект со значением по умолчанию
    propE: {
      type: Object,
      // Для объектов или массивов значения по умолчанию
      // должны возвращаться из функции
      default: function () {
        return { message: 'hello' }
      }
    },
    // Пользовательская функция для валидации
    propF: {
      validator: function (value) {
        // Значение должно соответствовать одной из этих строк
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

Когда валидация входного параметра заканчивается ошибкой — Vue выдаст предупреждение в консоли (если используется сборка для разработки).

<p class="tip">Обратите внимание, что входные параметры валидируются **перед** созданием экземпляра компонента, поэтому свойства экземпляра (например, `data`, `computed`, и т.д.) не будут доступны внутри `default` или функций `validator`.</p>

### Проверка типа

Значением `type` может быть один из следующих нативных конструкторов:

- String
- Number
- Boolean
- Function
- Object
- Array
- Symbol

Кроме того, `type` также может быть пользовательской функцией-конструктором и валидация будет выполняться проверкой с помощью `instanceof`. Например, если существует следующая функция-конструктор:

```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```

Вы можете использовать:

```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

чтобы проверить, что значение входного параметра `author` было создано с помощью `new Person`.

## Передача обычных атрибутов

Обычные атрибуты — это атрибуты, передаваемые в компонент, но не имеющие определения соответствующего входного параметра в компоненте.

Хотя явно определённые свойства предпочтительны для передачи информации дочернему компоненту, авторы библиотек компонентов не всегда могут предвидеть все контексты, в которых будут использованы их компоненты. Вот почему компоненты могут принимать произвольные атрибуты, которые добавляются в корневой элемент компонента.

Например, представьте, что мы используем сторонний компонент `bootstrap-date-input` с плагином Bootstrap, который требует указания атрибута `data-date-picker` на элементе `input`. Мы можем добавить этот атрибут к нашему экземпляру компонента:

``` html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```

И атрибут `data-date-picker="activated"` будет автоматически добавлен в корневой элемент `bootstrap-date-input`.

### Замена/Объединение существующих атрибутов

Представьте, что это шаблон для `bootstrap-date-input`:

``` html
<input type="date" class="form-control">
```

Чтобы добавить тему для нашего плагина выбора даты, нам может понадобиться добавить определённый класс, например:

``` html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

В этом случае определены два разных значения для `class`:

- `form-control`, который задаётся компонентом в его шаблоне
- `date-picker-theme-dark`, который передаётся компоненту его родителем

Для большинства атрибутов значение, предоставляемое компоненту, будет заменять значение, заданное компонентом. Например, передача `type="text"` будет заменять `type="date"` и, вероятно, ломать всё! К счастью, работа с атрибутами `class` и `style` немного умнее, поэтому оба значения будут объединены в итоговое: `form-control date-picker-theme-dark`.

### Отключение наследования атрибутов

Если вы **не хотите**, чтобы корневой элемент компонента наследовал атрибуты, вы можете установить `inheritAttrs: false` в опциях компонента. Например:

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```

Это может быть особенно полезно в сочетании со свойством экземпляра `$attrs`, которое содержит имена атрибутов и значения, переданные компоненту, например:

```js
{
  class: 'username-input',
  placeholder: 'Enter your username'
}
```

С помощью `inheritAttrs: false` и `$attrs` вы можете вручную определять к какому элементу должны применяться атрибуты, что часто требуется для [базовых компонентов](../style-guide/#Именование-базовых-компонентов-настоятельно-рекомендуется):

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```

Этот шаблон позволяет вам использовать базовые компоненты больше как обычные HTML-элементы, не беспокоясь о том, какой элемент будет у него корневым:

```html
<base-input
  v-model="username"
  class="username-input"
  placeholder="Enter your username"
></base-input>
```
