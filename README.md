# Taking your React application to the next level
There is already so many articles out there explain every and each of the new JavaScript libraries, so the point of this article is to point out the problems that may be solved better, to make our app more maintainable and readable. There is not going to be an overwhelming number of code, but rather a conceptual overview of what our problems and solutions to them are.

Now chances are, you find yourself in one of the following groups. Either you have found a stack that gets the job done and have became content with your choice, or you are JavaScript fatiqued and overwhelmed by the endless sea of choices, as can be seen on this figure:

![](https://i.redd.it/la9w4dr2mz9z.png)

This article targets the former group, more specifically, it will target the `React` + `Redux` tech stack, that has been dominating the mainstream market lately, although some parts may be applicable to other stacks.

## Before we start
So you heard that the cool kids use `React` and `Redux`. So you being a cool kid yourself, you probably went ahead and got started with [create react app](https://github.com/facebookincubator/create-react-app), added [redux](http://redux.js.org/) and obviously created a TODO app, as we all have.

So in order to gain something from this article you should be comfortable with the set of tools mentioned above. Cool, now that we are on the same page, we are going to go ahead and assume, that you are eager to learn how to improve upon the foundations of knowledge that you have laid down.

You could also argue, that building an application is all about finding the best tool for the job, and that it may be counterproductive to use all of these libraries for a small scale project. In STRV, all our frontend engineers are familiar with these libraries, so using them does not mean any overhead in terms of having to familiarize yourself with the technology. It also enables us to continue the development of the app if it outgrows the initial expectations without the need to use different tools, because if it gets better, more advanced tools may become more appropriate. Last but not least, by using similar tech stack for all of our projects makes it easier to switch between them without spending too much time on looking around as to find out what is where.

With that out of the way, let us proceed.

## Async
If you are anything like me, you were impressed by the genious of `redux`'s design, by how pure, explicit and understandable it is, once you get the hang of it. There is one central piece of every frontend application state that is missing in redux by default. It is its async capability.

You could imitate it with something along the lines of this:
```
componentDidMount() {
  ApiClient.getPeople()
    .then(result =>
      this.props.dispatch({ type: 'FETCH_PEOPLE', payload: result }))
}
```
However, this way of handling async just does not cut it for us. What if we wanted to work with the list of people in multiple different components? For each of those we would have to implement this `componentDidMount` method. I would strongly advise against this approach because it pollutes the UI code with business logic and throws separation of concerns out the window.

### Redux thunk to the rescue!
Not really. Do not get me wrong, `redux-thunk` works great for a lot of usecases, but if you aim to build a highly scalable application of bigger scale, it may not be enough.

You get some seperation from the UI as the business logic will now live in the action creators. So now action creators can be filled with potentially a big amount of business logic, which may be hard to read trough. Our aim is to keep our actions pure and simple and extract the side effects elsewhere.

### The real rescue
There are currently two very solid libraries to handle (asynchronous) side effects. Its [redux-saga](https://github.com/redux-saga/redux-saga) and [redux-observable](https://github.com/redux-observable/redux-observable).
Both of these are `redux` middlewares through which the actions dispatched within your application flow.

#### Redux-saga
In the case of sagas, you create so called sagas, that are watching actions being dispatched and allow you to react to it. Make API calls, dispatch other actions, anything really. It is using generators to handle the asynchronous nature of the middleware, which in my eyes is also the downside of this option.

#### Redux-observable
This middleware introduces the concept of epics. Which is simply a declaration through which the action flows, which is what makes it so beautiful. It is a pure declaration of what will happen, as opposed to the imperative Saga counterpart. It is based on Observables (not yet available in EcmaScript, so you might need something like [RxJS](https://github.com/ReactiveX/rxjs)). Which is the downside, but only in the sense that the learning curve of it is steeper, but once you learn it, it becomes a powerfull tool that you can utilize on multiple places as opposed to sagas, knowledge of which will most likely be rendered useless once you stop using it.

### Takeaway
The aim of this article is not to go deeply into these libraries, there is already a lot of higher quality materials out there, so the aim is merely to point out that there are better options out there, so why settle for the inferior one.

## Data mutation in reducers
Redux is functional and pure. While this is a very nice property to have, you can not say the same about JavaScript. JavaScript CAN be pure, however, it does not come without an extra effort.
The EcmaScript comitee has been kind to us so far and has given us some sweet sweet syntactic sugar to deal with this. Namely it is the object and array destructuring and the spread operator.

However, when editing deeply nested JavaScript objects for example, it can very quickly become a very difficult task. Also because the data is actually mutable, it does not give you a guarantee that when new programmer comes on board, he does not mutate something he should not.

### Immutable to the rescue
Fabook has authored a very nice library for the very purpose of storing data in immutable data strctures that enable us to perform pure operations on the data. The libary is called [ImmutableJS](https://facebook.github.io/immutable-js/).

It does not come without its pitfalls though. When working with `ImmutableJS` data structures, you either have to access data like so:
```
person.get('name')
```
Or, you have to define Immutable records, which requires additional writing on your part. Because we want to keep our components independent of the type of data it is working with, it is not uncommon to transform the data to plain JavaScript object using the `toJS` method. That way we are only using the `ImmutableJS` library to perform pure operations on immutable data structures and are losing some fo this benefits.

For that very reason I have started playing with the idea of dropping the usage of `ImmutableJS` and instead use a functional library, that, because it is functional, does not mutate the parameter when applying changes to the structure.

There are [underscore](https://www.npmjs.com/package/underscore) or [ramda](https://www.npmjs.com/package/ramda), which are functional utility libraries that do just that. I strongly favor the later, see [this excellent talk](https://www.youtube.com/watch?v=m3svKOdZijA) on the topic. Now these libraries are very big, bit there are webpack plugins that enable us to only import the functions we are using, making the size not a factor. `ImmutableJS` on the other hand is very big in size and there is nothing you can do about it. Also chances are, you already have one of these libraries introduced in the project, because their utility functions enable you to write elegant code, so you might already be using it, it is just a matter of taking advantage of it in the reducer.

### Takeaway
So whichever you choose, the result will likely be much better than with plain JavaScript objects.

## Recompose
[Recompose](https://github.com/acdlite/recompose) is a compilation of higher order components that enable us to write more declarative, purer code. When writing react components, when we have the opportunity, we favor `functional components`, because they are pure, easier to read and reason about. However, often times we need to add some local state that does not make sense to be stored in redux. In that case, we need change the component from the `functional` form to the class syntax, where you can use the state.

Or maybe  you are just worried about using functional components, because they are internally using the regular class component. Which means it is potentially less performant than `PureComponent`.

Fear not, for both of these use cases, `recompose` offers a higher order component, that makes it possible for you to take advantage of that clean `functional` form of component and gain the peformance and state capabilities of a `PureComponent`.

## CSS
Now there is a ton of ways how one can approach writing stylesheets. Writing efficient and maintainable css requires experience and discipline.
It is very common for stylesheets for a component to be stored in a completely different place and navigating in and maintaining such a project will quickly become a nightmare when it scales.

For these reasons it is a very good idea to couple your css files next to your component definition, making the search for the styles that you need to edit a breeze.

However, that still leaves us with a ton of problems left to solve. If you are using too general class names you will quickly encounter conflicts in class names and unexpected behavior will happen.

Or maybe you are nesting your CSS inside some kind of `.testimonials-page` class. That leads to problems with specificity when trying to override the css.

One of the ways out of this is using a naming methodology such as [BEM](http://getbem.com/). But then you end up with very long class names that may be even harder to come up with than it needs to be.

All of this is solved by CSS-in-JS. [Styled components](https://github.com/styled-components/styled-components) is our favorite implementation of this and I highly encourage you to check it out.

## Think before you npm install
There is an [excellent talk](https://www.youtube.com/watch?v=4bZvq3nodf4) on this topic, so I highly recommend to watch it. It is now easier than ever to install an npm module for everything. But it all adds up to the resulting bundle size.

We are, for now, living in a world where fast internet is by no means a guarantee. You could either have a bad signal in the mountains, metro station, or you live in a country where fast internet access is a rare commodity.

So in order for our apps to be loaded in a reasonable time, we HAVE to pay a close attention to our resulting bundle size, optimizing our assets, taking advantage of service workers and offline caching wherever possible.

There are great tools to help with this:
- network tab in Chome
- [Webpack bundle analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Cost of modules](https://cost-of-modules.herokuapp.com/)

So before you install [MomentJS](https://momentjs.com/) so you can use it once, think about finding lighter weight alternative or writing it itself if the usage is trivial enough.
