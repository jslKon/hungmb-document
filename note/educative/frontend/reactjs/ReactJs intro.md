# React intro

## render() method

Inform React how the component is displayed/rendered. Have a return value

Should put at the end of all methods.

Should return a React component in most cases.

## ReactDOM

Allow React interact with DOM (Document Object Model) -> allow to interact with browser

ReactDOM.render() -> enables us to render a React element onto a root node, so we can display it on the screen

root node is not replaced, but rather, the content is inserted into the container

Best practice: we normally do not call ReactDOM.render() more than once, and only use it to initialize our Single Page
App to load the page

React never changes the root node itself, only its contents


