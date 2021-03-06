# ApiGroup

> [Home](../README.md) &gt; [Classes](README.md) &gt; ApiGroup
 
## Intro

The `ApiGroup` class extends [`ApiCore`](ApiCore.md) and is the base building block for named actions.

It provides functionality for:

- configuration of actions via a map of `<name>: <config>` pairs
- a variety of shorthand ways to configure the endpoints themselves
- population of URL `:placeholders` with payload variables
- calling of actions via `call('<action>')` or `<action>()`
- handling call completion via `when()`

## API

The `ApiGroup` class has the following API:

```
+- ApiCore
+- ApiGroup ( axios: Axios, actions: any = null )
    |
    +-  actions : ActionMap
    |    |
    |    +- map           : object<Action>
    |    |   +- config    : object : AxiosRequestObject
    |    |   +- handlers  : Set<Function>
    |    |
    |    +- get           ( name: string )
    |    +- add           ( name: string, url: string, method: string = 'get', handler: Function = null )
    |    +- remove        ( name: string )
    |    +- addHandler    ( name: string, handler: Function )
    |    +- removeHandler ( name: string, handler: Function )
    |
    +-  add  ( action: string, url : string, method: string = 'get', handler: Function = null ) : ApiGroup
    +-  when (name: string, callback: Function) : ApiGroup
    +-  call ( action: string, data?: any, options?: any ) : Promise
```


## Configuration

You set it up with an axios instance and a configuration block:

```js
export const widgets = new ApiGroup(axios, {
  view: 'products/widgets?category=:category',
  load: 'products/widgets/:id',
  save: 'POST products/widgets/:id',
})
```

Note that:

- HTTP methods can be specified directly in the URL format
- placeholders can be used anywhere in the string
- you can use `:value` or `{value}` style placeholders
- placeholder variables are automatically filled-in using payload data
- values can be passed as function arguments, an array, or object


## Usage

To use the endpoint, import it, then either `call()` the action by name, or execute it directly:

```js
import { widgets } from '../api'

// call by name
widgets.call('load', 1).then(onLoad)

// execute directly
widgets.load(1).then(onLoad)
```

Note that `ApiGroup` will **not** override existing properties or methods with new ones.


## Adding actions

`ApiGroup` and any classes which extend from it can have actions added to them at any point.

The signature is an `action`, `config` and optional `handler`:

```js
widgets.add(action, config, handler)
```

There are 3 signatures you can use:

```js
// url
widgets.add('search', 'products/search?product=widgets&text=:text')

// method + url
widgets.add('upload', 'POST products/widgets/:id/files')

// config
widgets.add('create', { method: 'post', url: 'products/widgets'})
```

The optional `handler` function will be added and called when the request completes.

Execute the request via `call()` or the automatically-created method:

```js
widgets
  .search(form)
  .then(onSearch)
```


Actions can be added to any `ApiGroup` instances, or if greater functionality is required, you can [extend from a base class](../extension/classes.md) and add your own custom methods.


## Handling events

Events (call success or failure) can be handled in three ways:

- per instance
- per action
- per call

### Per-instance

To set up instance-level event handling (via [`ApiCore`](ApiCore.md#handling-events)) use `done()` and `fail()` on the Api instance:

```js
const posts = new ApiEndpoint(axios, 'posts/:id')
  .done(onLoad)
  .fail(onError)
```
```js
posts.index()
```

### Per-action

To set up action-level event handling, use `when()` with a string of `action` names on any `ApiGroup` (or subclass) instance:

```js
function onAction (res, action) {
    console.log(`action: ${action}`, res)
    posts.index() // reload
}

const posts = new ApiEndpoint(axios, 'posts/:id')
  .when('create update delete', onAction)
```

```js
posts.create({title: 'new post', body: 'this is a new post'})
```

Note that the handler will only be called for successful calls.

### Per-call

To set up call-level event handling, use `then()` and `catch()` on the call's returned Promise:

```js
const posts = new ApiEndpoint('posts/:id')
```
```js
posts
  .index()
  .then(res => this.data = res.data.map(post => new Post(post)))
```

## Next steps 

- Docs: [ApiEndpoint](ApiEndpoint.md)
- Code: [`src/classes/ApiGroup.ts`](https://github.com/davestewart/axios-actions/blob/master/src/classes/ApiGroup.ts)

