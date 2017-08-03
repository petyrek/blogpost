# Taking your React application to the next level
There are already many articles out there, that explain each and every library out there. The purpose of this article is to point out some problems that may be solved better, and make our app more maintainable and readable. There is not going to be an overwhelming amount of code, but rather a conceptual overview of what some of our problems might look like and a possible solution to solve them.

Now, chances are you find yourself in one of the following two groups: You have found a stack that gets the job done and have became content with your choice. Or you are feeling overwhelmed by the endless sea of choices, also known as "Javascript fatigue".

![](https://i.redd.it/la9w4dr2mz9z.png)

This article targets the first group. Specifically, it will target the `React` + `Redux` tech stack that has been dominating the mainstream lately. Some parts may be applicable to other stacks.

## Before we start
You may have heard that all the cool kids are using `React` and `Redux` these days. Being a cool kid yourself, you probably went ahead and got started with [create react app](https://github.com/facebookincubator/create-react-app), added [redux](http://redux.js.org/) in the mix, and created a TODO app as we all have.

In order to gain something from this article you should be comfortable with the set of tools mentioned above. Now that we are on the same page, we are going to go ahead and assume that you are interested in learning how to further improve upon the foundations of your React and Redux knowledge.

You could argue that building an application is all about finding the right tool for the job. It may seem counterproductive to use all of these libraries for a small scale project, and you would be right, BUT, that is a big but. At our company, all of our frontend engineers are familiar with the libraries mentioned in this article, so we feel using them does not present any significant cognitive overhead.

Using these tools will also enables us to efficiently continue the development of the app if it ever outgrows the initial expectations without the need to switch to a different set of tools.

Last but not least, using a similar tech stack for all of our projects makes it easier for our engineers to switch between projects without spending too much time looking around to find out what is where.

With that out of the way, let us proceed.

## Async
If you are anything like me, you were impressed by the simplicity of `redux`'s design. It's pure, explicit, and understandable, at least once you get the hang of it anyway. Unfortunately there is one central piece missing in Redux that every frontend application should have. That is async capability.

You could imitate it with something along the lines of this:
```js
componentDidMount() {
  ApiClient.getPeople()
    .then(result =>
      this.props.dispatch({ type: 'FETCH_PEOPLE', payload: result }))
}
```
This way of handling async just does not cut it for us. What if we wanted to work with a list of people in multiple different components? For each of those we would have to implement this `componentDidMount` method or fetch the data somewhere at the top of the component tree and pass it down to the appropriate component. I strongly advise against this approach because it pollutes the UI code with business logic and throws separation of concerns right out the window.

### Redux thunk to the rescue!
Not really. Don't get me wrong, `redux-thunk` works great for many simple use cases, but if you aim to build a large application that can scale over time, it may not be enough.

You get some separation from the UI, as the business logic will now live in the action creators. However, now the action creators are filled with potentially a significant amount of business logic, which may be hard to read through. Our aim is to keep our actions pure and simple, and extract the side effects elsewhere.

### The real rescue
Currently there are two very solid libraries to handle (asynchronous) side effects. They are [redux-saga](https://github.com/redux-saga/redux-saga) and [redux-observable](https://github.com/redux-observable/redux-observable).
Both of these are `redux` middleware which actions flow through and new actions can be dispatched.

#### Redux-saga
In the case of sagas, you create the so called sagas that watch actions being dispatched and allows you to react to them. Make API calls and dispatch other actions. It uses generators to handle the asynchronous nature of the middleware, which in my eyes is also the downside of this option.

#### Redux-observable
This middleware introduces the concept of epics. Epics are simply a declaration through which every action flows. It is a pure declaration of what will happen, as opposed to the imperative Saga counterpart. It is based on Observables (not yet available in EcmaScript, so you might need something like [RxJS](https://github.com/ReactiveX/rxjs)). One main downside of using observables is that the learning curve might be steeper. Once you learn it, however, it becomes a powerful tool in your tool-set that you can utilize in many situations as opposed to sagas.

## Data mutation in reducers
Redux is functional and pure. While this is a wonderful property to have, you can not say the same about JavaScript. JavaScript CAN be pure, however, it does not come without extra effort and discipline is required.
The EcmaScript committee has been kind to us so far and has given us some sweet sweet syntactic sugar to deal with functional programming. Namely it is the object and array destructuring and the spread operator.

When editing deeply nested JavaScript objects, it can quickly become a difficult task. Because plain javascript object data is mutable, it does not give you any guarantee that an object doesn't get mutated somewhere in the code by an inexperienced colleague, which will introduce a hard to trace bug. We are expecting the object references to be different after an operation is performed, which doesn't happen when mutating an object.

### Immutable to the rescue
Facebook has authored a very nice library for the very purpose of storing data in immutable data structures. The library is called [ImmutableJS](https://facebook.github.io/immutable-js/). It enables us to perform pure operations on our data.

Immutable does not come without its pitfalls though. When working with `ImmutableJS` data structures, you either have to access data like this:
```js
person.get('name')
```
Or, you have to define immutable records, which requires additional writing on your part. Because we want to keep our components independent of the type of data it is working with, it is not uncommon to transform the data to a plain JavaScript object using the `toJS` method. That way we are only using the `ImmutableJS` library to perform pure operations on immutable data structures, but are losing some of its benefits.

For that very reason I have started playing with the idea of dropping the usage of `ImmutableJS` in favor of a functional utility library that is functional, and does not mutate the parameter when applying changes to the structure.

An example of this can be either [underscore](https://www.npmjs.com/package/underscore) or [ramda](https://www.npmjs.com/package/ramda), which both do just that. I strongly favor the later, see [this excellent talk](https://www.youtube.com/watch?v=m3svKOdZijA) on the topic. Now these libraries are big in size, but there are webpack plugins that enable us to only import the functions we are using, eliminating the size factor. `ImmutableJS` on the other hand is very big in size, and there is nothing you can do about minimizing it's size footprint.

Because their utility functions enable you to write elegant code, it is just a matter of taking advantage of it in the reducers. Chances are, you already have one of these libraries installed in your project.

## Recompose
[Recompose](https://github.com/acdlite/recompose) is a compilation of higher order components (HOC) for react that enable you to write more declarative, purer code. When writing react components, we favor `functional components` whenever we get the chance. They are pure, easier to read, and easier to reason about. Often times we need to add some local state to our component, so in that case we may need to change the component from the `functional` form to the class syntax. Or we may use the `withState` HOC to achieve a similar result without using the class syntax.

One might also be worried about using functional components because internally they are using the regular class component. Which means it is potentially less performant than `PureComponent`.

Fear not, for both of these and many more use cases `recompose` offers a higher order component, that makes it possible for you to take advantage of that clean `functional` syntax and gain the performance and state capabilities of a `PureComponent`.

## CSS
Now there are a ton of ways how one can approach writing stylesheets. Writing efficient and maintainable css requires experience and discipline.
It is very common for stylesheets that belong to a component to be stored in a completely different place. With this approach, navigating a project and maintaining it, will quickly become a nightmare.

For these reasons we think it is a good idea to couple your css files next to your component's definition, making the search for the styles that you need to edit a breeze.

This still leaves us with a ton of problems to solve. If you are using too general class names you will quickly encounter conflicts in class names and unexpected behavior will occur.

Or maybe you are nesting your CSS inside some kind of `.testimonials-page` class. That leads to problems with specificity when trying to override the css.

One of the ways out of this is by using a naming methodology such as [BEM](http://getbem.com/). But then you end up with very long class names.

All of this is solved by CSS-in-JS. [Styled components](https://github.com/styled-components/styled-components) is our favorite implementation of this and I highly encourage you to check it out.

## Think before you npm install
There is an [excellent talk](https://www.youtube.com/watch?v=4bZvq3nodf4) on this topic, so I suggest you to watch it. It is now easier than ever to install an npm module for everything. But it all adds up to the resulting bundle size.

For now, we are living in a world where fast internet is by no means a guarantee. You could either have a bad signal in the mountains or a metro station, or maybe your user lives in a country where fast internet access is a rare commodity.

In order for our apps to be loaded in a reasonable time on slow networks, we NEED to pay close attention to the resulting bundle size of our app. We should also be optimizing all of our assets, taking advantage of service workers, and offline caching wherever possible.

There are great tools to help with this:
- network tab in Chrome
- [Webpack bundle analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Cost of modules](https://cost-of-modules.herokuapp.com/)

So the next time you are in your terminal and thinking about npm installing [MomentJS](https://momentjs.com/) for a simple time conversion, think about finding a light weight alternative, or writing it yourself.
