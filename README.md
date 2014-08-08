Tree.js
=======

Tree.js is a JavaScript library to build and manipulate hookable trees.

# Installation

It is available with bower:

```
bower install tree.js
```

Then add the retrieved files to your HTML layout:

```html
<script type="text/javascript" src="/path/to/bower_components/tree.js/tree.min.js"></script>
<!-- If you want to build hookable tree (see below) -->
<script type="text/javascript" src="/path/to/bower_components/q/q.js"></script>
```

You can also use it with [RequireJS](http://requirejs.org/)  as an AMD module.

Usage
-----

### Simple tree

#### Create a tree

```javascript

var myTree = Tree.tree({
    children: [
        {
            name: 'dupuis',
            children: [
                {
                    name: 'prunelle'
                    children: [
                        {
                            name: 'lebrac',
                            job: 'designer'
                        },
                        {
                            name: 'lagaffe',
                            firstname: 'gaston',
                            job: 'sleeper'
                        },
                    ]
                }
            ]
        }
    }
    ]
});
```

#### Find a node

```javascript
var lebrac = myTree.find('/dupuis/prunelle/lebrac');

// or

var lebrac = myTree.find('/dupuis').find('/prunelle/lebrac');

// or

var lebrac = myTree.find('/dupuis').find('/prunelle').find('/lebrac');
```

#### Get the raw data of a node

```javascript
lebrac.data() // { name: 'lebrac', job: 'designer' }
```

#### Get an attribute

```javascript
lebrac.attr('job'); // designer
```

#### Set an attribute

```javascript
lebrac.attr('job', 'director');
lebrac
    .attr('location', 'here')
    .attr('hobby', 'design');
```

#### Get the path of a node

```javascript
lebrac.path(); // /dupuis/prunelle/lebrac
```

#### Get the parent of a node

```javascript
var dupuis = lebrac.parent();
dupuis.name(); // dupuis
dupuis.parent(); // undefined
```

#### Append a child node

```javascript
lebrac.append(Tree.tree({
    name: 'paper',
    children: [{ name: 'pen' }]
}));

lebrac.find('/paper/pen');
lebrac.find('/paper').parent().parent().parent().name(); // dupuis
```

#### Remove a node

```javascript
lebrac.remove();
myTree.find('/dupuis/prunelle/lebrac'); // undefined
```

#### Move a node

```javascript
var lagaffe = myTree.find('/dupuis/prunelle/lagaffe').moveTo(myTree.find('/dupuis'));
lagaffe.path(); // /dupuis/lagaffe
```

#### Get the children of a node

```javascript
var children = myTree.find('/dupuis').children();
children[0].name(); // prunelle
```

### Work with hooks

To work with hooks, you first need to add hook capacities to your tree:

```javascript
var hookableTree = Tree.hookable(myTree);
```

Everything explained above is still true but the hookable operations will now return promises!

Because of that you need to include into your page `Q` library. Otherwise you can specify another promises library by calling: `hookableTree.promiseFactory(YOUR_FACTORY)`.

#### Register a hook listener

There are 12 hooks available:

| Hook              | Description                                                        |
| ----------------: |:-------------------------------------------------------------------|
| HOOK_PRE_APPEND   | Trigger when `append` is called and before applying it on the tree |
| HOOK_POST_APPEND  | Trigger when `append` is called and after applying it on the tree  |
| HOOK_ERROR_APPEND | Trigger when `append` is called and an error occured               |
| HOOK_PRE_REMOVE   | Trigger when `remove` is called and before applying it on the tree |
| HOOK_POST_REMOVE  | Trigger when `remove` is called and after applying it on the tree  |
| HOOK_ERROR_REMOVE | Trigger when `remove` is called and an error occured               |
| HOOK_PRE_MOVE     | Trigger when `moveTo` is called and before applying it on the tree |
| HOOK_POST_MOVE    | Trigger when `moveTo` is called and after applying it on the tree  |
| HOOK_ERROR_MOVE   | Trigger when `moveTo` is called and an error occured               |
| HOOK_PRE_CLONE    | Trigger when `clone` is called and before applying it on the tree  |
| HOOK_POST_CLONE   | Trigger when `clone` is called and after applying it on the tree   |
| HOOK_ERROR_CLONE  | Trigger when `clone` is called and an error occured                |

To register a hook you need to call `registerListener(string hook, function listener)`:

```
hookableTree.registerListener(hookableTree.HOOK_PRE_APPEND, function(next, newNode) {
    // I am a hook listener, I will be triggered before any append operation.

    // The arguments other than the next callback are not always the same depending on the hook.
    // Hook context is the tree.

    this; // will be our tree

    // When I am done I MUST call the next callback:
    //   - with no argument if everything is OK,
    //   - with an error if something goes wrong and I want to prevent the append to be completed: next('error').
    next();
});
```

Because of hooks, `append`, `remove`, `move`, `clone` will return promise, as show in this example:

```
hookableTree.append(Tree.tree({ name: 'spirou'})).then(function(childNode) {
    // Everything is ok, it worked!
    // When you append a node, you can either give as argument a tree or hookable tree.
    // If it is a hookable tree, its hook listeners will added to the parent tree.
}, function(err) {
   // An error occured or a hook listener failed
});
```

```
hookableTree.find('/dupuis').remove().then(function(dupuisParent) {
    // Everything is ok, it worked!
}, function(err) {
    // An error occured or a hook listener failed
});
```

```
hookableTree.find('/dupuis/prunelle').moveTo(hookableTree).then(function(prunelle) {
    // Everything is ok, it worked!
}, function(err) {
    // An error occured or a hook listener failed
});
```

```
hookableTree.find('/dupuis/prunelle').clone().then(function(clonedPrunelle) {
    // Everything is ok, it worked!
}, function(err) {
    // An error occured or a hook listener failed
});
```

Build
------

To rebuild the minified JavaScript you must run: `make build`.

Tests
-----
Install dependencies and run the unit tests:

```
make install
make test-spec
```

Contributing
------------

All contributions are welcome and must pass the tests. If you add a new feature, please write tests for it.

License
-------

This application is available under the MIT License, courtesy of [marmelab](http://marmelab.com).
