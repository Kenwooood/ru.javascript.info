archive:
  ref: null

---


# Строки

Есть ряд улучшений и новых методов для строк.

Начнём с, пожалуй, самого важного.

## Строки-шаблоны

Добавлен новый вид кавычек для строк:
```js
let str = `обратные кавычки`;
```

Основные отличия от двойных `"…"` и одинарных `'…'` кавычек:

- **В них разрешён перевод строки.**

    Например:
    ```js run
    alert(`моя
      многострочная
      строка`);
    ```

    Заметим, что пробелы и, собственно, перевод строки также входят в строку, и будут выведены.
- **Можно вставлять выражения при помощи `${…}`.**

    Например:
    ```js run
    'use strict';
    let apples = 2;
    let oranges = 3;

    alert(`${apples} + ${oranges} = ${apples + oranges}`); // 2 + 3 = 5
    ```

    Как видно, при помощи `${…}` можно вставлять как и значение переменной `${apples}`, так и более сложные выражения, которые могут включать в себя операторы, вызовы функций и т.п. Такую вставку называют "интерполяцией".

## Функции шаблонизации

Можно использовать свою функцию шаблонизации для строк.

Название этой функции ставится перед первой обратной кавычкой:
```js
let str = func`моя строка`;
```

Эта функция будет автоматически вызвана и получит в качестве аргументов строку, разбитую по вхождениям параметров `${…}` и сами эти параметры.

Например:

```js run
'use strict';

function f(strings, ...values) {
  alert(JSON.stringify(strings));     // ["Sum of "," + "," =\n ","!"]
  alert(JSON.stringify(strings.raw)); // ["Sum of "," + "," =\\n ","!"]
  alert(JSON.stringify(values));      // [3,5,8]
}

let apples = 3;
let oranges = 5;

//          |  s[0] | v[0] |s[1]| v[1]  |s[2]  |      v[2]      |s[3]
let str = f`Sum of ${apples} + ${oranges} =\n ${apples + oranges}!`;
```

В примере выше видно, что строка разбивается по очереди на части: "кусок строки" -- "параметр" -- "кусок строки" -- "параметр".

- Участки строки идут в первый аргумент-массив `strings`.
- У этого массива есть дополнительное свойство `strings.raw`. В нём находятся строки в точности как в оригинале. Это влияет на спец-символы, например в `strings` символ `\n` -- это перевод строки, а в `strings.raw` -- это именно два символа `\n`.
- Дальнейший список аргументов функции шаблонизации -- это значения выражений в `${...}`, в данном случае их три.

```smart header="Зачем `strings.raw`?"
В отличие от `strings`, в `strings.raw` содержатся участки строки в "изначально введённом" виде.

То есть, если в строке находится `\n` или `\u1234` или другое особое сочетание символов, то оно таким и останется.

Это нужно в тех случаях, когда функция шаблонизации хочет произвести обработку полностью самостоятельно (свои спец. символы?). Или же когда обработка спец. символов не нужна -- например, строка содержит "обычный текст", набранный непрограммистом без учёта спец. символов.
```

Как видно, функция имеет доступ ко всему: к выражениям, к участкам текста и даже, через `strings.raw` -- к оригинально введённому тексту без учёта стандартных спец. символов.

Функция шаблонизации может как-то преобразовать строку и вернуть новый результат.

В простейшем случае можно просто "склеить" полученные фрагменты в строку:

```js run
'use strict';

// str восстанавливает строку
function str(strings, ...values) {
  let str = "";
  for(let i=0; i<values.length; i++) {
    str += strings[i];
    str += values[i];
  }

  // последний кусок строки
  str += strings[strings.length-1];
  return str;
}

let apples = 3;
let oranges = 5;

// Sum of 3 + 5 = 8!
alert( str`Sum of ${apples} + ${oranges} = ${apples + oranges}!`);
```

Функция `str` в примере выше делает то же самое, что обычные обратные кавычки. Но, конечно, можно пойти намного дальше. Например, генерировать из HTML-строки DOM-узлы (функции шаблонизации не обязательно возвращать именно строку).

Или можно реализовать интернационализацию. В примере ниже функция `i18n` осуществляет перевод строки.

Она подбирает по строке вида `"Hello, ${name}!"` шаблон перевода `"Привет, {0}!"` (где `{0}` -- место для вставки параметра) и возвращает переведённый результат со вставленным именем `name`:

```js run
'use strict';

let messages = {
  "Hello, {0}!": "Привет, {0}!"
};

function i18n(strings, ...values) {
  // По форме строки получим шаблон для поиска в messages
  // На месте каждого из значений будет его номер: {0}, {1}, …
  let pattern = "";
  for(let i=0; i<values.length; i++) {
    pattern += strings[i] + '{' + i + '}';
  }
  pattern += strings[strings.length-1];
  // Теперь pattern = "Hello, {0}!"

  let translated = messages[pattern]; // "Привет, {0}!"

  // Заменит в "Привет, {0}" цифры вида {num} на values[num]
  return translated.replace(/\{(\d)\}/g, (s, num) => values[num]);
}

// Пример использования
*!*
let name = "Вася";

// Перевести строку
alert( i18n`Hello, ${name}!` ); // Привет, Вася!
*/!*
```

Итоговое использование выглядит довольно красиво, не правда ли?

Разумеется, эту функцию можно улучшить и расширить. Функция шаблонизации -- это своего рода "стандартный синтаксический сахар" для упрощения форматирования и парсинга строк.

## Улучшена поддержка Юникода

Внутренняя кодировка строк в JavaScript -- это UTF-16, то есть под каждый символ отводится ровно два байта.

Но под всевозможные символы всех языков мира 2 байт не хватает. Поэтому бывает так, что одному символу языка соответствует два Юникодных символа (итого 4 байта). Такое сочетание называют "суррогатной парой".

Самый частый пример суррогатной пары, который можно встретить в литературе -- это китайские иероглифы.

Заметим, однако, что не всякий китайский иероглиф -- суррогатная пара. Существенная часть "основного" Юникод-диапазона как раз отдана под китайский язык, поэтому некоторые иероглифы -- которые в неё "влезли" -- представляются одним Юникод-символом, а те, которые не поместились (реже используемые) -- двумя.

Например:

```js run
alert( '我'.length ); // 1
alert( '𩷶'.length ); // 2
```

В тексте выше для первого иероглифа есть отдельный Юникод-символ, и поэтому длина строки `1`, а для второго используется суррогатная пара. Соответственно, длина -- `2`.

Китайскими иероглифами суррогатные пары, естественно, не ограничиваются.

Ими представлены редкие математические символы, а также некоторые символы для эмоций, к примеру:

```js run
alert( '𝒳'.length ); // 2, MATHEMATICAL SCRIPT CAPITAL X
alert( '😂'.length ); // 2, FACE WITH TEARS OF JOY
```

В современный JavaScript добавлены методы [String.fromCodePoint](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCodePoint) и [str.codePointAt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/codePointAt) -- аналоги `String.fromCharCode` и `str.charCodeAt`, корректно работающие с суррогатными парами.

Например, `charCodeAt` считает суррогатную пару двумя разными символами и возвращает код каждой:

```js run
// как будто в строке два разных символа (на самом деле один)
alert( '𝒳'.charCodeAt(0) + ' ' + '𝒳'.charCodeAt(1) ); // 55349 56499
```

...В то время как `codePointAt` возвращает его Unicode-код суррогатной пары правильно:

```js run
// один символ с "длинным" (более 2 байт) unicode-кодом
alert( '𝒳'.codePointAt(0) ); // 119987
```

Метод `String.fromCodePoint(code)` корректно создаёт строку из "длинного кода", в отличие от старого `String.fromCharCode(code)`.

Например:

```js run
// Правильно
alert( String.fromCodePoint(119987) ); // 𝒳
// Неверно!
alert( String.fromCharCode(119987) ); // 풳
```

Более старый метод `fromCharCode` в последней строке дал неверный результат, так как он берёт только первые два байта от числа `119987` и создаёт символ из них, а остальные отбрасывает.

### \u{длинный код}

Есть и ещё синтаксическое улучшение для больших Unicode-кодов.

В JavaScript-строках давно можно вставлять символы по Unicode-коду, вот так:

```js run
alert( "\u2033" ); // ″, символ двойного штриха
```

Синтаксис: `\uNNNN`, где `NNNN` -- четырёхзначный шестнадцатиричный код, причём он должен быть ровно четырёхзначным.

"Лишние" цифры уже не войдут в код, например:

```js run
alert( "\u20331" ); // Два символа: символ двойного штриха ″, а затем 1
```

Чтобы вводить более длинные коды символов, добавили запись `\u{NNNNNNNN}`, где `NNNNNNNN` -- максимально восьмизначный (но можно и меньше цифр) код.

Например:

```js run
alert( "\u{20331}" ); // 𠌱, китайский иероглиф с этим кодом
```

### Unicode-нормализация

Во многих языках есть символы, которые получаются как сочетание основного символа и какого-то значка над ним или под ним.

Например, на основе обычного символа `a` существуют символы: `àáâäãåā`. Самые часто встречающиеся подобные сочетания имеют отдельный Юникодный код. Но отнюдь не все.

Для генерации произвольных сочетаний используются несколько Юникодных символов: основа и один или несколько значков.

Например, если после символа `S` идёт символ "точка сверху" (код `\u0307`), то показано это будет как "S с точкой сверху" `Ṡ`.

Если нужен ещё значок над той же буквой (или под ней) -- без проблем. Просто добавляем соответствующий символ.

К примеру, если добавить символ "точка снизу" (код `\u0323`), то будет "S с двумя точками сверху и снизу" `Ṩ` .

Пример этого символа в JavaScript-строке:

```js run
alert("S\u0307\u0323"); // Ṩ
```

Такая возможность добавить произвольной букве нужные значки, с одной стороны, необходима, а с другой стороны -- возникает проблемка: можно представить одинаковый с точки зрения визуального отображения и интерпретации символ -- разными сочетаниями Unicode-кодов.

Вот пример:
```js run
alert("S\u0307\u0323"); // Ṩ
alert("S\u0323\u0307"); // Ṩ

alert( "S\u0307\u0323" == "S\u0323\u0307" ); // false
```

В первой строке после основы `S` идёт сначала значок "верхняя точка", а потом -- нижняя, во второй -- наоборот. По кодам строки не равны друг другу. Но символ задают один и тот же.

С целью разрешить эту ситуацию, существует *Юникодная нормализация*, при которой строки приводятся к единому, "нормальному", виду.

В современном JavaScript это делает метод [str.normalize()](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/String/normalize).

```js run
alert( "S\u0307\u0323".normalize() == "S\u0323\u0307".normalize() ); // true
```

Забавно, что в данной конкретной ситуации `normalize()` приведёт последовательность из трёх символов к одному: [\u1e68 (S с двумя точками)](http://www.fileformat.info/info/unicode/char/1e68/index.htm).

```js run
alert( "S\u0307\u0323".normalize().length ); // 1, нормализовало в один символ
alert( "S\u0307\u0323".normalize() == "\u1e68" ); // true
```

Это, конечно, не всегда так, просто в данном случае оказалось, что именно такой символ в Юникоде уже есть. Если добавить значков, то нормализация уже даст несколько символов.

Для большинства практических задач информации, данной выше, должно быть вполне достаточно, но если хочется более подробно ознакомиться с вариантами и правилами нормализации -- они описаны в приложении к стандарту Юникод [Unicode Normalization Forms](http://www.unicode.org/reports/tr15/).

## Полезные методы

Добавлен ряд полезных методов общего назначения:

- [str.includes(s)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes) -- проверяет, включает ли одна строка в себя другую, возвращает `true/false`.
- [str.endsWith(s)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/endsWith) -- возвращает `true`, если строка `str` заканчивается подстрокой `s`.
- [str.startsWith(s)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith) -- возвращает `true`, если строка `str` начинается со строки `s`.
- [str.repeat(times)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/repeat) -- повторяет строку `str` `times` раз.

Конечно, всё это можно было сделать при помощи других встроенных методов, но новые методы более удобны.

## Итого

Улучшения:

- Строки-шаблоны -- для удобного задания строк (многострочных, с переменными), плюс возможность использовать функцию шаблонизации для самостоятельного форматирования.
- Юникод -- улучшена работа с суррогатными парами.
- Полезные методы для проверок вхождения одной строки в другую.
