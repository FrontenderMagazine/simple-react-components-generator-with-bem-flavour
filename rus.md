*TL;DR*: [dumb-bem][1] — библиотека для простого использования BEM с React компонентами.

## Введение

Уверен, все фронтенд-разработчики слышали про [БЭМ][2] и [React][3].

И если про каждый из этих подходов уже было сказано немало слов, я хочу поговорить про то, как их совмещать в разработке.

В Wimdu мы используем правила по наименованию компонентов из БЭМ и некоторые концепты SMACSS.

Например, типичный компонент выглядит так:
```html
<header className='header header--landing'>
  <h1 className='header__title'>...</h1>
  <h2 className='header__subtitle is-hidden'>...</h2>
</header>
```

Этот компонент состоит из

*   блока `header` с модификатором `landing`
*   элемента `header__title`
*   элемента `header__subtitle` с состоянием `is-hidden`

## Компоненты

*Ниже в примерах будем использовать [ES6][4] и [функциональные компоненты][5] для описания компонентов React.*

Попробуем сделать компонент `Header` с заголовком и подзаголовком.

Начнём с простого:

```js
// components/header/index.js

export default ({ modifier, title, subtitle }) => (
  <header className=`header header--${modifier}`>
    <h1 className='header__title'>{ title }</h1>
    <h2 className=`header__subtitle ${!subtitle ? 'is-hidden' : ''}`>
      { subtitle }
    </h2>
  </header>
)
```

Отлично, теперь можно передавать различные параметры в получившийся компонент и использовать его на разных страницах:

```js
import Header from 'components/header'

// например, поместим компонент на главную страницу
ReactDOM.render(
  <Header
    modifier='landing'
    title='City Apartments'
    subtitle='Over 5 Million Nights Booked'
  />
, node)

// или на страницу /about
ReactDOM.render(
  <Header
    modifier='about'
    title='About Wimdu'
    subtitle='Meet our team and learn about what we do'
  />
, node)
```

Сделаем модификатор необязательным. Чтобы не писать кучу тернарных операторов, возьмем удобную библиотеку для работы с классами [classnames][6].

```js
// components/header/index.js

import cx from 'classnames'

export default ({ modifier, title, subtitle }) => (
  <header className={cx('header', { [`header--${modifier}`]: modifier })}>
    <h1 className='header__title'>{ title }</h1>
    <h2 className={cx('header__subtitle', { 'is-hidden': subtitle })}>
      { subtitle }
    </h2>
  </header>
)
```


Получили то, что и хотели — компонент, готовый к использованию в разных контекстах. Решение рабочее, хотя и выглядит довольно неаккуратно.
Если нам предстоит написать несколько десятков таких компонентов — лучше придумать что-то более элегантное.

## Удобство использования

Честно говоря, мы хотим, чтобы `Header` выглядел максимально просто: блок с модификатором и двумя элементами внутри.

Попробуем вынести элементы `Header`, `Title` и `Subtitle` из данного компонента наружу.

```js
// components/header/index.js

import { Header, Title, Subtitle } from './elements'

export default ({ modifier, title, subtitle }) => (
  <Header modifier={modifier}>
    <Title>{ title }</Title>
    <Subtitle hidden={!subtitle}>{ subtitle }</Subtitle>
  </Header>
)
```
То, что нужно! Похоже на слегка параметризованный html-компонент из начала статьи.

Теперь определим вынесенные наружу элементы.

Основная идея — взять React элементы и поколдовать над их свойствами.

Будем использовать библиотеку [transform-props-with][7] для изменения свойств:

```js
// components/header/elements.js

import cx from 'classnames'
import tx from 'transform-props-with'

const addElementStyles = (oldProps) => {
  const { hidden, modifier, name, ...props } = oldProps

  return {
    className: cx({
      [`header__${name}`]: name,
      [`header__${name}--${modifier}`]: name && modifier,
      ['is-hidden']: hidden
    }),
    ...props
  }
}

// добавляем React-элементу header класс .header
export const Header = tx({ className: 'header' })('header')
// добавляем React-элементу h1 класс .header__title
export const Title = tx([{ name: 'title' }, addElementStyles])('h1')
// добавляем React-элементу h2 класс .header__title
export const Subtitle = tx([{ name: 'subtitle' }, addElementStyles])('h2')
```

Фактически, мы декорировали React элементы `header`, `h1` и `h2`.

Теперь передавая свойства `modifier` или `hidden` в наши атомарные компоненты — для них будет сгенерирован соответствующий класс.

## Встречаем dumb-bem

[dumb-bem][1] — это библиотека, которую мы написали для создания атомарных React компонентов с BEM/SMACSS правилами по наименованию классов.

Передавая имя блока в фyнкцию `dumbBem`, вы получаете назад функцию-декоратор, которая меняет свойства у React элемента.

```js
import dumbBem from 'dumb-bem'
import tx from 'transform-props-with'

// здесь header — название BEM блока
const dumbHeader = dumbBem('header')

const Header = tx(dumbHeader)('header')
const Title = tx([dumbHeader, { element: 'title' }])('h1')
const Subtitle = tx([dumbHeader, { element: 'subtitle' }])('h2')

export default ({ modifier, title, subtitle }) => (
  <Header modifier={modifier}>
    <Title>{ title }</Title>
    <Subtitle hidden={!subtitle}>{ subtitle }</Subtitle>
  </Header>
)
```

Страницы проекта: [github][1] и [npm][8].

О более подробном использовании данной библиотеки читайте в [документации][9], либо в моей следующей статье.

 [1]: https://github.com/agudulin/dumb-bem
 [2]: http://getbem.com/
 [3]: https://facebook.github.io/react/
 [4]: https://github.com/lukehoban/es6features#readme
 [5]: https://facebook.github.io/react/docs/reusable-components.html#stateless-functions
 [6]: https://www.npmjs.com/package/classnames
 [7]: https://github.com/robinpokorny/transform-props-with
 [8]: https://www.npmjs.com/package/dumb-bem
 [9]: https://github.com/agudulin/dumb-bem/blob/master/README.md
