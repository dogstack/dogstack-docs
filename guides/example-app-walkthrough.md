# Creating your first `dogstack` app

🐕Welcome🐕 

This is a walkthrough of our example app. If you want to skip the doggy day care and get right into the code, we recommend you jump over to our [How to create a dogstack app](https://dogstack.js.org/guides/how-to-create-app.html) walkthrough. This walkthrough is for people that are familiar with `react` and `redux` but are new to functional Javascript or want a more in-depth knowledge of the modules and tooling `dogstack` uses.

## What we'll be creating
![An example of what we're creating](https://i.imgur.com/zQxPdoW.gif)

We'll be giving dogs a new leash on life(geddit?) with this 'adopt a dog' generator.
Users can adopt a dog(add a dog to our list) or give a dog to a friend(remove a dog from our list)

## Pre-requisites

This walkthough assumes you have a basic knowledge of `react`, `redux` and functional programming. If you don't, that's cool, but we reccoemnd learning the basics and coming back once you feel comfortable with them. Here's a list of resources we like.

- React
  - TODO: Need example that's not using classes
- Redux
  - https://egghead.io/courses/getting-started-with-redux
  - https://medium.com/@rajaraodv/step-by-step-guide-to-building-react-redux-apps-using-mocks-48ca0f47f9a
  - https://github.com/reactjs/redux/blob/master/docs/introduction/ThreePrinciples.md
- Functional Programming
  - https://medium.freecodecamp.org/functional-programming-in-js-with-practical-examples-part-1-87c2b0dbc276 (if you only read one thing from this article, make it the Currying section)

Before we start, please

- [install `node@8` and `npm@5`](https://dogstack.js.org/guides/how-to-install-js.html)

## Let's code

Let's start by cloning our example repo

`git clone git://github.com/root-systems/dogstack-example`

Installing the dependencies
```shell
cd dogstack-example
npm install

```

and setting up the database for the first time
```shell
npm run db migrate:latest
npm run db seed:run

```

Now that we've set up the scaffolding of our projects, let's dive in and see what we've got.

### File structure

The first thing you'll notice is how the directories are set up. 

![file structure](https://i.imgur.com/phigXbc.png)

Our project is split into directories for each conceptual topic, where each topic contains the various types of files within that topic.

This leads to a directory structure of `${topic}/${type}/${name}`.

You can read more about what topics, types and names are [here](https://dogstack.js.org/conventions/file-structure.html)

### There's so many files, what do they all do?!
Well, you're in for a treat. Although there is a lot of boilerplate, it's all there intentionally. You'll regularly find files less than 10 lines long this is because dogstack is heavily abstracted and focuses on everything being [obvious and readable.](https://github.com/reactjs/redux/issues/2295)

Hopefully everything is obvious and readable 😉, but lets walk you through some of the files anyway.

#### server.js - required by dogstack

dogstack is using the [feathers](https://feathersjs.com/) framework. Feathers makes creating and consuming a real-time API super easy. We'll get into more depth about Feathers and what it can do later, but for now, think of it as a way to interact with our API.

In this file you can list all of your [feathers services](https://docs.feathersjs.com/api/services.html)

We'll come back soon to create our first service.

#### routes.js - required by dogstack

dogstack is using [React routes](https://github.com/ReactTraining/react-router) for our routing

We're in the process of refactoring the routes and navigation into it's own module, but for now we've included these files into this boilerplate, so don't worry if you don't understand what's going on in these files:

- `app/getters.js` - This is used to map our routes to the Layout component as props.


---
dogstack expects `routes.js` to be an exported routes configuration in this format:

```js

export default  [
  {
    name: 'home',
    path: '/',
    exact: true,
    Component: Home,
    selector: getIsNotAuthenticated,
    navigation: {
      title: 'app.home',
      icon: 'fa fa-home'
    }
  },
  {
    name: 'signIn',
    path: '/sign-in',
    exact: true,
    Component: UserIsNotAuthenticated(SignIn),
    navigation: {
      title: 'agents.signIn',
      selector: getIsNotAuthenticated,
      icon: 'fa fa-sign-in'
    }
  }
]
```

You can read more into react-router and find the docs [here](https://reacttraining.com/react-router/web/guides/philosophy)

#### store.js - required by dogstack

This is for our [redux]() store config.

- [`updater`(add link to updater.js below)](): a function of shape `action => state => nextState`, combined from each topic using [`redux-fp.concat`](https://github.com/rvikmanis/redux-fp/blob/master/docs/API.md#concat)
- [`epic`(add link to epic below)](): a function of shape `(action$, store, { feathers }) => nextAction$`, combined from each topic using [`combineEpics`](https://redux-observable.js.org/docs/api/combineEpics.html)
- [`middlewares`](http://redux.js.org/docs/Glossary.html#middleware): an array of functions of shape `store => next => action`
- [`enhancers`](http://redux.js.org/docs/Glossary.html#store-enhancer): an array of functions of shape `createStore => createEnhancedStore

```js
// store.js
import updater from './updater'
import epic from './epic'

const middlewares = []
const enhancers = []

export default {
  updater,
  epic,
  middlewares,
  enhancers
}
```


#### updater.js

> An Updater is a type of function that takes an action and returns another function that takes the state and produces a result. Yes, they are just curried reducers with reversed argument order.

If you're used to the term `reducer`, you can see in our `updater.js` file we have a function that converts reducers to updaters

```js
function reducerToUpdater (reducer) {
  return action => state => reducer(state, action)
}
``` 


This file combines all of our apps `updaters` using [`redux-fp.concat`](https://github.com/rvikmanis/redux-fp/blob/master/docs/API.md#concat). Think of this as your 'root updater'. For any updaters to interact with the state, they must be in this file.

#### epic.js

This file exports a function of shape `(action$, store, { feathers }) => nextAction$`, combined from each topic using [`combineEpics`](https://redux-observable.js.org/docs/api/combineEpics.html)

> An Epic is a function which takes a stream of actions and returns a stream of actions. Actions in, actions out.

Need this looked at, still confusing to me after looking into `dogstack-agents`
The main way dogstack uses epics is with Feathers. We can subscribe to events, the epic will define what events we want to subscribe to, and what happens when the event is triggered.

Luckily [feathers-action](https://www.npmjs.com/package/feathers-action) does most of the heavy lifting for us and this will be the main way to create epics, but it helps to know the basics of [Observables and RxJs](http://reactivex.io/rxjs/manual/overview.html)

#### client.js - required by dogstack

This file is used for our [`feathers` client](https://docs.feathersjs.com/api/client.html) configuration(yes! feathers is also a client) so that know how to communicate with our feathers server and connect things like authentication.

This file is used to define what services we want the feathers client to use.

`services`: export an array of functions that will be run with [`client.configure(plugin)`](https://docs.feathersjs.com/api/application.html#configurecallback)

#### root.js

To ask Mikey - what's the purpose of styleNode in dogstack (in root.js)? I see where we use appNode, but don't see styleNode being used at all?

#### style.js required by dogstack

export configuration for [`fela`](https://github.com/rofrischmann/fela)

- `fontNode`: query selector string or dom node to render app fonts
- `theme`: object passsed to `<FelaThemeProvider theme={theme} />`
- `setup`: function of shape `(renderer) => {}`

We're using [`fela`](https://github.com/rofrischmann/fela) to style dogstack apps. Fela lets us create state-driven styling to create flexible and dynamic styes. We'll dive into styling with Fela later in this walkthrough.

#### intl.js - required by dogstack

dogstack is using [react-intl](https://github.com/yahoo/react-intl) to handle Internationalisation of our app. react-intl makes creating an app for users that read a number of different langagues super easy.

At the moment you can see the only langague we're using at the moment is English. By using react-intl it makes it incredibly easy at any point later in the lifecycle of your app you need to cater for another langague. You add an extra locale json file to `app/locales` and you're ready to go.

Even if you're currently not planning to cater for different langagues we reccoemdn you use this feature. For a small amount of effort you recive a large reward later on.
All it takes is to start using the [`FormattedMessage`](https://github.com/yahoo/react-intl/wiki/Components#string-formatting-components) component where you're using text.

[react-intl](https://github.com/yahoo/react-intl) also has a bunch of other features other than translation, feel free to [check out the docs](https://github.com/yahoo/react-intl) to see what other magic it does

#### layout.js - required by dogstack

And last, but not least `layout.js`

This is our main layout React component, which accepts `routes` as props.
We're also in the process of refactoring and abstracting how this layout works in the future.

## Show me the dogs 🐶
Now let's have a look at what we've got!

`npm run dev`

![step one](https://i.imgur.com/NkrJTPH.png)
That's not a dog...