# Taking your React application to the next level
There is already so many articles out there, that explain each and every library out there. The point of this article is to point out the problems, that may be solved better, to make our app more maintainable and readable. There is not going to be an overwhelming amount of code, but rather a conceptual overview of what some of our problems might be and the solutions to them.

Now chances are, you find yourself in one of the following groups. Either you have found a stack that gets the job done and have became content with your choice, or, you are JavaScript fatiqued and overwhelmed by the endless sea of choices, as can be seen on this figure:

![](https://i.redd.it/la9w4dr2mz9z.png)

This article targets the former group. More specifically, it will target the `React` + `Redux` tech stack, that has been dominating the mainstream market lately, although some parts may be applicable to other stacks.

## Before we start
So you heard that the cool kids are using `React` and `Redux`. So you, being a cool kid yourself, probably went ahead and got started with [create react app](https://github.com/facebookincubator/create-react-app), added [redux](http://redux.js.org/) in the mix and obviously created a TODO app, as we all have.

So in order to gain something from this article you should be comfortable with the set of tools mentioned above. Cool, now that we are on the same page, we are going to go ahead and assume, that you are interesting in learning how to further improve upon the foundations of knowledge that you have laid down.

You could also argue, that building an application is all about finding the best tool for the job, and that it may be counterproductive to use all of these libraries for a small scale project, and you would be right, BUT, thereis a big but. In our company, all of our frontend engineers are familiar with the libraries mentioned in the article, so using them does not present any overhead in terms of having to familiarize yourself with the technology. It also enables us to efficiently continue the development of the app if it ever outgrows the initial expectations without the need to switch to a different set of tools. Last but not least, using similar tech stack for all of our projects makes it easier to switch between them without spending too much of our time looking around as to find out what is where.

With that out of the way, let us proceed.

## Async
If you are anything like me, you were impressed by the genious of `redux`'s design, by how pure, explicit and understandable it is ,once you get the hang of it anyway. There is one central piece of every frontend application state that is missing in redux by default. It is its async capability.

You could imitate it with something along the lines of this:
```
componentDidMount() {
  ApiClient.getPeople()
    .then(result =>
      this.props.dispatch({ type: 'FETCH_PEOPLE', payload: result }))
}
```
However, this way of handling async just does not cut it for us. What if we wanted to work with the list of people in multiple different components? For each of those we would have to implement this `componentDidMount` method or fetch the data somewhere at the top of the component tree and pass it down to the appropriate component. I would strongly advise against this approach because it pollutes the UI code with business logic and throws separation of concerns right out the window.

### Redux thunk to the rescue!
Not really. Do not get me wrong, `redux-thunk` works great for a lot of usecases, but if you aim to build a highly scalable application of big enough scale, it may not be enough.

You get some seperation from the UI, as the business logic will now live in the action creators. But now the action creators Are filled with potentially a big amount of business logic, which may be hard to read through. Our aim is to keep our actions pure and simple and extract the side effects elsewhere.

### The real rescue
There are currently two very solid libraries to handle (asynchronous) side effects. Its [redux-saga](https://github.com/redux-saga/redux-saga) and [redux-observable](https://github.com/redux-observable/redux-observable).
Both of these are `redux` middlewares through which the actions dispatched within your application flow and allow you to act on them.

#### Redux-saga
In the case of sagas, you create so called sagas, that are watching actions being dispatched and allow you to react to it. Make API calls, dispatch other actions, anything really. It is using generators to handle the asynchronous nature of the middleware, which in my eyes is also the downside of this option.

#### Redux-observable
This middleware introduces the concept of epics. Which is simply a declaration through which every action flows, which is also what makes it so beautiful. It is a pure declaration of what will happen, as opposed to the imperative Saga counterpart. It is based on Observables (not yet available in EcmaScript, so you might need something like [RxJS](https://github.com/ReactiveX/rxjs)). Which is the downside, but only in the sense that the learning curve of it is steeper, but once you learn it, it becomes a powerfull tool in your toolbelt, that you can utilize on multiple places as opposed to sagas, knowledge of which, will most likely be rendered useless once you stop using it.

## Data mutation in reducers
Redux is functional and pure. While this is a very nice property to have, you can not say the same about JavaScript. JavaScript CAN be pure, however, it does not come without an extra effort and discipline required.
The EcmaScript comitee has been kind to us so far and has given us some sweet sweet syntactic sugar to deal with this. Namely it is the object and array destructuring and the spread operator.

However, when editing deeply nested JavaScript objects for example, it can very quickly become a very difficult task. Also, because the data is actually mutable, it does not give you a guarantee that when new programmer comes on board, he does not mutate something he should not and cause a hard to trace bug.

### Immutable to the rescue
Fabook has authored a very nice library for the very purpose of storing data in immutable data structures that enable us to perform pure operations on the data. The libary is called [ImmutableJS](https://facebook.github.io/immutable-js/).

It does not come without its pitfalls though. When working with `ImmutableJS` data structures, you either have to access data like this:
```
person.get('name')
```
Or, you have to define Immutable records, which requires additional writing on your part. Because we want to keep our components independent of the type of data it is working with, it is not uncommon to transform the data to plain JavaScript object using the `toJS` method. That way we are only using the `ImmutableJS` library to perform pure operations on immutable data structures and are losing some of its benefits.

For that very reason I have started playing with the idea of dropping the usage of `ImmutableJS` in favor of a functional utility library. Because it is functional, it does not mutate the parameter when applying changes to the structure.

An example of these can be either [underscore](https://www.npmjs.com/package/underscore) or [ramda](https://www.npmjs.com/package/ramda), which both do just that. I strongly favor the later, see [this excellent talk](https://www.youtube.com/watch?v=m3svKOdZijA) on the topic. Now these libraries are big in size, but there are webpack plugins that enable us to only import the functions we are using, making the size not a factor. `ImmutableJS` on the other hand is very big in size, and there is nothing you can do about it. Also, chances are, you already have one of these libraries introduced in the project, because their utility functions enable you to write elegant code, so you might be already be using it, it is just a matter of taking advantage of it in the reducers.

## Recompose
[Recompose](https://github.com/acdlite/recompose) is a compilation of higher order components that enable us to write more declarative, purer code. When writing react components, we favor `functional components` whenever we get the chance, because they are pure, easier to read and reason about. However, often times we need to add some local state to it. In that case, we need change the component from the `functional` form to the class syntax, where you can use take advantage of its state.

Or maybe  you are just worried about using functional components, because internally, they are using the regular class component. Which means it is potentially less performant than `PureComponent`.

Fear not, for both of these and many more use cases `recompose` offers a higher order component, that makes it possible for you to take advantage of that clean `functional` syntax and gain the peformance and state capabilities of a `PureComponent`.

## CSS
Now there is a ton of ways how one can approach writing stylesheets. Writing efficient and maintainable css requires experience and discipline.
It is very common for stylesheets for a component to be stored in a completely different place and navigating within such project or even maintaining it will quickly become a nightmare.

For these reasons it is a very good idea to couple your css files next to your component's definition, making the search for the styles that you need to edit a breeze.

However, that still leaves us with a ton of problems left to solve. If you are using too general class names you will quickly encounter conflicts in class names and unexpected behavior will happen.

Or maybe you are nesting your CSS inside some kind of `.testimonials-page` class. That leads to problems with specificity when trying to override the css.

One of the ways out of this is using a naming methodology such as [BEM](http://getbem.com/). But then you end up with very long class names that may be even harder to come up with than it needs to be.

All of this is solved by CSS-in-JS. [Styled components](https://github.com/styled-components/styled-components) is our favorite implementation of this and I highly encourage you to check it out.

## Think before you npm install
There is an [excellent talk](https://www.youtube.com/watch?v=4bZvq3nodf4) on this topic, so I highly recommend to watch it. It is now easier than ever to install an npm module for everything. But it all adds up to the resulting bundle size.

We are, for now, living in a world where fast internet is by no means a guarantee. You could either have a bad signal in the mountains or a metro station, or maybe your user lives in a country where fast internet access is a rare commodity.

So in order for our apps to be loaded in a reasonable time on slow networks, we HAVE to pay close attention to the resulting bundle size of our app. We should also be optimizing all of our assets, taking advantage of service workers and offline caching wherever possible.

There are great tools to help with this:
- network tab in Chome
- [Webpack bundle analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Cost of modules](https://cost-of-modules.herokuapp.com/)

So next time before you install [MomentJS](https://momentjs.com/) so you can just use it once, think about finding lighter weight alternative or writing it itself if the usage is trivial enough.
