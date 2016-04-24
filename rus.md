# React-компоненты с привкусом БЭМ

Уверен, все фронтенд-разработчики слышали про [БЭМ][2] и [React][3].

И если про каждый из этих подходов уже было сказано немало, я хочу поговорить про то, как совмещать их в разработке.

Мы в [Wimdu][11] используем правила по наименованию компонентов из БЭМ и некоторые концепты [SMACSS][10].

Например, типичный компонент выглядит так:

    <header className='header header--landing'>
        <h1 className='header__title'>…</h1>
        <h2 className='header__subtitle is-hidden'>…</h2>
    </header>


Этот компонент состоит из:

* блока `header` с модификатором `landing`
* элемента `header__title`
* элемента `header__subtitle` с состоянием `is-hidden`


## Компоненты

*Ниже в примерах будем использовать [ES6][4] и [функциональные компоненты][5] для описания компонентов React.*

Попробуем сделать компонент `Header` с заголовком и подзаголовком.

Начнём с простого:


    // components/header/index.js
     
    export default ({ modifier, title, subtitle }) => (
        <header className=`header header--${modifier}`>
            <h1 className='header__title'>{ title }</h1>
            <h2 className=`header__subtitle ${!subtitle ? 'is-hidden' : ''}`>
                { subtitle }
            </h2>
        </header>
    )

Отлично, теперь можно передавать различные параметры в получившийся компонент и использовать его на разных страницах:

    import Header from 'components/header'
     
    // например, поместим компонент на главную страницу
    ReactDOM.render(
        <Header
            modifier='landing'
            title='Городские апартаменты'
            subtitle='Более 5 миллионов резерваций'
        />
    , node)
     
    // или на страницу /about
    ReactDOM.render(
        <Header
            modifier='about'
            title='Про Wimdu'
            subtitle='Познакомьтесь с нашей командой и посмотрите, что мы умеем'
        />
    , node)

Сделаем модификатор необязательным. Чтобы не писать кучу тернарных операторов, возьмем удобную библиотеку для работы с классами [classnames][6].


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

Получили то, что и хотели — компонент, готовый к использованию в разных контекстах. Решение рабочее, хотя и выглядит довольно неаккуратно.
Если нам предстоит написать несколько десятков таких компонентов, лучше придумать что-то более элегантное.


## Удобство использования

Честно говоря, мы хотим, чтобы `Header` выглядел максимально просто: блок с модификатором и двумя элементами внутри.

Попробуем вынести элементы `Header`, `Title` и `Subtitle` из компонента наружу:

    // components/header/index.js
     
    import { Header, Title, Subtitle } from './elements'
     
    export default ({ modifier, title, subtitle }) => (
        <Header modifier={modifier}>
            <Title>{ title }</Title>
            <Subtitle hidden={!subtitle}>{ subtitle }</Subtitle>
        </Header>
    )

То, что нужно! Похоже на слегка параметризованный html-компонент из начала статьи.

Теперь определим вынесенные наружу элементы.

Основная идея — взять React-элементы и поколдовать над их свойствами.

Для изменения свойств будем использовать библиотеку [transform-props-with][7]:

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
    // добавляем React-элементу h2 класс .header__subtitle
    export const Subtitle = tx([{ name: 'subtitle' }, addElementStyles])('h2')

Фактически, мы декорировали React-элементы `header`, `h1` и `h2`.

Теперь при передаче в наши атомарные компоненты свойства `modifier` или `hidden` для них будет сгенерирован соответствующий класс.


## Встречаем dumb-bem

[dumb-bem][1] — это библиотека, которую мы написали для создания атомарных React компонентов с BEM/SMACSS правилами по наименованию классов.

Передавая имя блока в фyнкцию `dumbBem`, вы получаете функцию-декоратор, которая меняет свойства у React-элемента.

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

Страницы проекта: [github][1] и [npm][8].

Более подробно об использовании данной библиотеки читайте в [документации][9] либо в моей следующей статье.

 [1]: https://github.com/agudulin/dumb-bem
 [2]: http://getbem.com/
 [3]: https://facebook.github.io/react/
 [4]: https://github.com/lukehoban/es6features#readme
 [5]: https://facebook.github.io/react/docs/reusable-components.html#stateless-functions
 [6]: https://www.npmjs.com/package/classnames
 [7]: https://github.com/robinpokorny/transform-props-with
 [8]: https://www.npmjs.com/package/dumb-bem
 [9]: https://github.com/agudulin/dumb-bem/blob/master/README.md
 [10]: https://smacss.com/
 [11]: http://www.wimdu.com/
