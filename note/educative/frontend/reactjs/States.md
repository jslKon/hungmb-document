# State

Components can:

- Hold
- Manage
- Change their own state.

But if the state within a component changes, it always triggers a re-render of the component

## Lifecycle methods

![img.png](Lifecycle%20Methods.png)

Mount phase : methods are only called once when the component is first rendered (meaning, when they’re added to the DOM)

Update phase : If new props are passed to a component, or if state changes within the component, update methods
will be called

Unmount phase: This phase only has one matching method, which is called as soon as the component is removed from the
DOM. It can be useful to tidy up event listeners and setTimeOut() or setInterval() calls that were added during mounting
of the component

Error Handling: This method can be called to catch errors that occur during the rendering process in a lifecycle method
or in the constructor of a child component

![img.png](Lifecycle%20Method%20Diagram.png)

## Define States

State inside a class component can be accessed with the instance property this.state. However, it is encapsulated in the
component, and neither parent nor child components can access it

Define states using constructor

```js
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            counter: props.counter,
        };
    }

    render() {
        // ...
    }
}
```

With babel plugins

```js
class MyComponent extends React.Component {
    state = {
        counter: this.props.counter,
    };

    render() {
        // ...
    }
}
```

## Change States

To change states, use

```js
this.setSate(updatedState)
```

If change state directly -> nothing change since re-render is not triggered

If you happen to have the same property name in your object as you do in the state, the state value will be overwritten
while all other properties will remain the same. To reset properties in the state, their values need to be explicitly
set to null or undefined. The new state that is being passed is thus never replaced but merged with the existing state

To guarantee that only the most current state is always accessed, we should use an updater function with the current
state as a parameter. Many developers have already made the mistake of trying to access this.state right after calling
setState() only to realize that their state is still old

```
Note: React collects many sequential setState() calls and does not immediately invoke them in order to avoid an unnecessary amount of re-renders. 
Sequential setState() calls, which are called right after each other, are executed as a batch process. 
This is important to keep in mind as we cannot reliably access new states with this.state, just after a setState() call.
```

If want to access state right after update -> use callback as 2nd params of setState()