*TL;DR*: We implemented a library [dumb-bem][1] which helps you use BEM in React
components.

## Intro

There are two buzzwords in the front-end world. You’ve probably heard about
both of them: BEM and React.

[BEM][2] is a methodology, that provides you with simple naming convention, and
[React][3] is a JavaScript library for building user interfaces.

Combining these two approaches, help you build a complex UI, while codebase
stays maintainable.

At Wimdu we take a hybrid approach using BEM naming convention and some of the
SMACSS concepts.

An example of a typical component:

        <header className='header header--landing'>
          <h1 className='header__title'>...</h1>
          <h2 className='header__subtitle is-hidden'>...</h2>
        </header>
        

Here we have

*   `header` block with `landing` modifier
*   `header__title` element
*   `header__subtitle` element with `is-hidden` state

## Reusable components

*We use [ES6][4] JavaScript version and [stateless function components][5] for
describing React components.*

Let’s try to make a `Header` component displaying a title, a subtitle, and make
it re-usable for the future.

We start with a simple React component:

        // components/header/index.js
        
        export default ({ modifier, title, subtitle }) => (
          <header className=`header header--${modifier}`>
            <h1 className='header__title'>{ title }</h1>
            <h2 className=`header__subtitle ${!subtitle ? 'is-hidden' : ''}`>
              { subtitle }
            </h2>
          </header>
        )
        

Now we can render it with different props:

        import Header from 'components/header'
        
        // For example, we can render it on a landing page
        ReactDOM.render(
          <Header
            modifier='landing'
            title='City Apartments'
            subtitle='Over 5 Million Nights Booked'
          />
        , node)
        
        // or on the About page
        ReactDOM.render(
          <Header
            modifier='about'
            title='About Wimdu'
            subtitle='Meet our team and learn about what we do'
          />
        , node)
        

Let’s make `modifier` optional. We may want to use [classnames][6] library for
gently joining class names.

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
        

Quite messy and too much code to write for every new component.

## Ease of use

Honestly, all we want to do in this component is simply provide a modifier for
`header` block and place `title` and `subtitle` elements inside.

It would be great to abstract `Header`, `Title` and `Subtitle` elements outside
of this component.

        // components/header/index.js
        
        import { Header, Title, Subtitle } from './elements'
        
        export default ({ modifier, title, subtitle }) => (
          <Header modifier={modifier}>
            <Title>{ title }</Title>
            <Subtitle hidden={!subtitle}>{ subtitle }</Subtitle>
          </Header>
        )
        

Let’s define `Header` block and two elements: `Title` and `Subtitle`.

How can we do it? The idea is to take a React element and modify its props.

We will use [transform-props-with][7] library for that:

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
        
        export const Header = tx({ className: 'header' })('header')
        export const Title = tx([{ name: 'title' }, addElementStyles])('h1')
        export const Subtitle = tx([{ name: 'subtitle' }, addElementStyles])('h2')
        

So basically we have just decorated React elements `header`, `h1` and `h2`.

Whenever we pass `modifier` or `hidden` props to our components — a proper
class name will be generated.

## Meet the dumb-bem

[dumb-bem][1] is a library we created for making atomic React components with
BEM/SMACSS naming rules.

When you pass a block name to `dumbBem` function, it returns a decorator which
you can use to transform props on React elements.

        // components/header/elements.js
        
        import dumbBem from 'dumb-bem'
        import tx from 'transform-props-with'
        
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
        

It’s available on [github][1] and [npm][8].

Read more detailed usage of this library in the future blog post.

 [1]: https://github.com/agudulin/dumb-bem
 [2]: http://getbem.com/
 [3]: https://facebook.github.io/react/
 [4]: https://github.com/lukehoban/es6features#readme

 [5]: https://facebook.github.io/react/docs/reusable-components.html#stateless-functions
 [6]: https://www.npmjs.com/package/classnames
 [7]: https://github.com/robinpokorny/transform-props-with
 [8]: https://www.npmjs.com/package/dumb-bem