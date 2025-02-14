# RxReact

>  [ReactJS](http://facebook.github.io/react/) bindings for [RxJS](https://github.com/Reactive-Extensions/RxJS)


# Installation

Install this module with npm: 

```
npm install rx-react
```

# Usage: 

RxReact provides a set of utilities to work with RxJS and React : 

* The `StateStreamMixin`
* The `LifecycleMixin`
* The `PropsMixin`
* The `RxReact.Component` base class
* The `FuncSubject` helper



## StateStreamMixin

The `StateStreamMixin` allows to bind a component state to an RxJS `Observable` stream. 
The way to achieve the binding is to provide a `getStateStream` method on your component that returns an RxJS `Observable`, the `StateStream` mixin will automatically merge the state of your component with the values published by the returned observable. The subscription will be automaticly cleaned on component unmount.

Example: 

```javascript
var StateStreamMixin = require('rx-react').StateStreamMixin;
var React = require('react');
var Rx = require('rx');


var Timer = React.createClass({
  mixins: [StateStreamMixin],
  getStateStream: function () {
    return Rx.Observable.interval(1000).map(function (interval) {
      return {
        secondsElapsed: interval
      };
    });
  },
  render: function () {
    var secondsElapsed = this.state ? this.state.secondsElapsed : 0;
    return (
      <div>Seconds Elapsed: {secondsElapsed}</div>
    );
  }
});

React.render(<Timer />, document.getElementById('timer-holder'));
```


## LifecycleMixin

The `LifecycleMixin` allows you to consume React components lifecycle events as RxJS `Observable`.
The `LifecycleMixin` will inject a property `lifecycle` to the component, that property contains an observable for each lifecycle events.

Example : 

```javascript
var LifecycleMixin = require('rx-react').LifecycleMixin;
var React = require('react');
var Rx = require('rx');


var Component = React.createClass({
  mixins: [LifecycleMixin],
  componentWillMount: function () {
    this.lifecycle.componentDidMount.subscribe(function () {
      console.log('componentDidMount');
    });
    
    this.lifecycle.componentWillReceiveProps.subscribe(function (props) {
      console.log('componentWillReceiveProps : ' JSON.stringify(props));
    });
    
    this.lifecycle.componentWillUpdate.subscribe(function ({nextProps, nextState}) {
      console.log('componentWillUpdate : ' JSON.stringify({nextProps, nextState}));
    });
    
    this.lifecycle.componentDidUpdate.subscribe(function ({prevProps, prevState}) {
      console.log('componentDidUpdate : ' JSON.stringify({prevProps, prevState}));
    });
    this.lifecycle.componentWillUnmount.subscribe(function () {
      console.log('componentWillUnmount');
    });
  },
  render: function() {
    //...
  }
});
```

## PropsMixin

The `PropsMixin` allows to obtain a stream of props as RxJS `Observable` for your component.
Example : 

```javascript
var PropsMixin = require('rx-react').PropsMixin;
var React = require('react');


var Component = React.createClass({
  mixins: [PropsMixin],
  componentWillMount: function () {
    this.propsStream.subscribe(function (props) {
      console.log(props.message);
    }
  },
  render: function() {
    //...
  }
});

var comp = React.render(<Component message='Hello World!' />, domNode); // log 'Hello World!'
comp.setProps({message: 'Hello John'}); // log 'Hello John'
```
This is particulary useful in combination with the `StateStreamMixin` when your component states depends on Props.


## Component

The `RxReact.Component` is a base class combining the behavior of the `PropsStreamMixin` and the `StateStreamMixin`.
It extends `React.Component`.
Example: 

```javascript
var RxReact = require('rx-react');
var Rx = require('rx');

class MyComponent extends RxReact.Component {
  getStateStream() {
    return Rx.Observable.interval(1000).map(function (interval) {
      return {
        secondsElapsed: interval
      };
    });
  }
  
  render() {
    var secondsElapsed = this.state ? this.state.secondsElapsed : 0;
    return (
      <div>Seconds Elapsed: {secondsElapsed}</div>
    );
  }
}
```
Note that when you extend lifecycle methods: `componentWillMount` `componentWillReceiveProps` and `componentWillUnMount`, You must call the `super` method.

> Before the 0.3.x versions `RxReact.Component` also implemented lifecyle mixin behavior, for some perf reasons and because most of the time it's unnecessary this has been removed. 
> If you want reenable this behavior  use `FuncSubject` as lifecycle method, or manually apply the `LifecycleMixin` on your class.

## FuncSubject

The `FuncSubject` helper allows to create an RxJS `Observable` that can be injected as callback for React event handlers, refs, etc...
To create an handler use the `create` function of `FuncSubject`

```javascript
var myHandler = FuncSubject.create()
```

Example: 

```javascript
var FuncSubject = require('rx-react').FuncSubject;
var React = require('react');
var Rx = require('rx');


var Button = React.createClass({
  componentWillMount: function () {
    this.buttonClicked = FuncSubject.create();
    
    this.buttonClicked.subscribe(function (event) {
      alert('button clicked');
    })
  },
  render: function() {
    return <button onClick={this.buttonClicked} />
  }
});
```

`FuncSubject` also accept a function as argument,  if provided this funtion will be used to map the value of each elements.
This function will *always* be called even if the `FuncSubject` has no subscription.

```javascript
var FuncSubject = require('rx-react').FuncSubject;
var React = require('react');
var Rx = require('rx');


var MyComponent = React.createClass({
  componentWillMount: function () {
    this.inputValue = FuncSubject.create(function (event) {
      return event.target.value
    });
    
    this.inputValue.subscribe(function (value) {
      alert('inputValue changed :' + value);
    })
  },
  render: function() {
    return <input onChange={this.inputValue} />
  }
});
```

### FuncSubject.behavior

You can also create a `FuncSubject` that extends [`BehaviorSubject`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/behaviorsubject.md). simply use the `behavior` function exposed by `FuncSubject`: 


```javascript
var myHandler = FuncSubject.behavior(intialValue, mapFunction)
```
