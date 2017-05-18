# Infinite Tree [![build status](https://travis-ci.org/cheton/infinite-tree.svg?branch=master)](https://travis-ci.org/cheton/infinite-tree) [![Coverage Status](https://coveralls.io/repos/github/cheton/infinite-tree/badge.svg?branch=master)](https://coveralls.io/github/cheton/infinite-tree?branch=master)
[![NPM](https://nodei.co/npm/infinite-tree.png?downloads=true&stars=true)](https://www.npmjs.com/package/infinite-tree)

A browser-ready tree library that can efficiently display a large tree with smooth scrolling.

Powered by [FlatTree](https://github.com/cheton/flattree) and [Clusterize.js](https://github.com/NeXTs/Clusterize.js).

Demo: http://cheton.github.io/infinite-tree

[![infinite-tree](https://raw.githubusercontent.com/cheton/infinite-tree/master/media/infinite-tree.gif)](http://cheton.github.io/infinite-tree)

## Features
* High performance infinite scroll with large data set
* [Customizable renderer](https://github.com/cheton/infinite-tree/wiki/Options#rowrenderer) to render the tree in any form
* [Load nodes on demand](https://github.com/cheton/infinite-tree/wiki/Options#loadnodes)
* Native HTML5 drag and drop API
* A rich set of [APIs](https://github.com/cheton/infinite-tree#api-documentation)
* No jQuery

## Browser Support
![Chrome](https://github.com/alrra/browser-logos/raw/master/src/chrome/chrome_48x48.png)<br>Chrome | ![Edge](https://github.com/alrra/browser-logos/raw/master/src/edge/edge_48x48.png)<br>Edge | ![Firefox](https://github.com/alrra/browser-logos/raw/master/src/firefox/firefox_48x48.png)<br>Firefox | ![IE](https://github.com/alrra/browser-logos/raw/master/src/archive/internet-explorer_9-11/internet-explorer_9-11_48x48.png)<br>IE | ![Opera](https://github.com/alrra/browser-logos/raw/master/src/opera/opera_48x48.png)<br>Opera | ![Safari](https://github.com/alrra/browser-logos/raw/master/src/safari/safari_48x48.png)<br>Safari
--- | --- | --- | --- | --- | --- |
 Yes | Yes | Yes| 8+ | Yes | Yes | 
Need to include [es5-shim](https://github.com/es-shims/es5-shim#example-of-applying-es-compatability-shims-in-a-browser-project) polyfill for IE8

## React Support
Check out <b>react-infinite-tree</b> at https://github.com/cheton/react-infinite-tree.

## Installation
```bash
npm install --save infinite-tree
```

## Usage
```js
var InfiniteTree = require('infinite-tree');

// when using webpack and browserify
require('infinite-tree/dist/infinite-tree.css');

var data = {
    id: 'fruit',
    name: 'Fruit',
    children: [{
        id: 'apple',
        name: 'Apple'
    }, {
        id: 'banana',
        name: 'Banana',
        children: [{
            id: 'cherry',
            name: 'Cherry',
            loadOnDemand: true
        }]
    }]
};

var tree = new InfiniteTree({
    el: document.querySelector('#tree'),
    data: data,
    autoOpen: true, // Defaults to false
    droppable: { // Defaults to false
        hoverClass: 'infinite-tree-droppable-hover',
        accept: function(event, options) {
            return true;
        },
        drop: function(event, options) {
        }
    },
    loadNodes: function(parentNode, done) { // Load node on demand
        var nodes = [];
        setTimeout(function() { // Loading...
            done(null, nodes);
        }, 1000);
    },
    nodeIdAttr: 'data-id', // the node id attribute
    rowRenderer: function(node, treeOptions) { // Customizable renderer
        return '<div data-id="<node-id>" class="infinite-tree-item">' + node.name + '</div>';
    },
    shouldSelectNode: function(node) { // Determine if the node is selectable
        if (!node || (node === tree.getSelectedNode())) {
            return false; // Prevent from deselecting the current node
        }
        return true;
    }
});
```

#### Functions Usage
Learn more: [Tree](https://github.com/cheton/infinite-tree/wiki/Functions:-Tree) /  [Node](https://github.com/cheton/infinite-tree/wiki/Functions:-Node)
```js
var node = tree.getNodeById('fruit');
// → Node { id: 'fruit', ... }
tree.selectNode(node);
// → true
console.log(node.getFirstChild());
// → Node { id: 'apple', ... }
console.log(node.getFirstChild().getNextSibling());
// → Node { id: 'banana', ... }
console.log(node.getFirstChild().getPreviousSibling());
// → null
```

#### Events Usage
Learn more: [Events](https://github.com/cheton/infinite-tree/wiki/Events)
```js
tree.on('click', function(event) {});
tree.on('doubleClick', function(event) {});
tree.on('keyDown', function(event) {});
tree.on('keyUp', function(event) {});
tree.on('clusterWillChange', function() {});
tree.on('clusterDidChange', function() {});
tree.on('contentWillUpdate', function() {});
tree.on('contentDidUpdate', function() {});
tree.on('openNode', function(node) {});
tree.on('closeNode', function(node) {});
tree.on('selectNode', function(node) {});
tree.on('willOpenNode', function(node) {});
tree.on('willCloseNode', function(node) {});
tree.on('willSelectNode', function(node) {});
```

## API Documentation
* [Options](https://github.com/cheton/infinite-tree/wiki/Options)
* [Functions: Tree](https://github.com/cheton/infinite-tree/wiki/Functions:-Tree)
* [Functions: Node](https://github.com/cheton/infinite-tree/wiki/Functions:-Node)
* [Events](https://github.com/cheton/infinite-tree/wiki/Events)

## FAQ

### Index
* [Creating tree nodes with checkboxes](#creating-tree-nodes-with-checkboxes)
* [How to attach click event listeners to nodes?](#how-to-attach-click-event-listeners-to-nodes)
* [How to use keyboard shortcuts to navigate through nodes?](#how-to-use-keyboard-shortcuts-to-navigate-through-nodes)
* [How to select multiple nodes using the ctrl key (or meta key)?](#how-to-select-multiple-nodes-using-the-ctrl-key-or-meta-key)

#### Creating tree nodes with checkboxes

```js
tree.on('willSelectNode', function(node) {
    node.props.checked = !node.props.checked;
    tree.updateNode(node);
});
```

Sets the checked attribute in your rowRenderer:

```js
const tag = require('html5-tag');

const checkbox = tag('input', {
  type: 'checkbox',
  checked: node.props.checked
}, '');
```

#### How to attach click event listeners to nodes?

Use <b>event delegation</b> <sup>[[1](http://javascript.info/tutorial/event-delegation), [2](http://davidwalsh.name/event-delegate)]</sup>

```js
var el = document.querySelector('#tree');
var tree = new InfiniteTree(el, { /* options */ });

tree.on('click', function(event) {
    var target = event.target || event.srcElement; // IE8
    var nodeTarget = target;

    while (nodeTarget && nodeTarget.parentElement !== tree.contentElement) {
        nodeTarget = nodeTarget.parentElement;
    }

    // Call event.stopPropagation() if you want to prevent the execution of
    // default tree operations like selectNode, openNode, and closeNode.
    event.stopPropagation(); // [optional]
    
    // Matches the specified group of selectors.
    var selectors = '.dropdown .btn';
    if (nodeTarget.querySelector(selectors) !== target) {
        return;
    }

    // do stuff with the target element.
    console.log(target);
};

```

Event delegation with jQuery:
```js
var el = document.querySelector('#tree');
var tree = new InfiniteTree(el, { /* options */ });

// jQuery
$(tree.contentElement).on('click', '.dropdown .btn', function(event) {
    // Call event.stopPropagation() if you want to prevent the execution of
    // default tree operations like selectNode, openNode, and closeNode.
    event.stopPropagation();
    
    // do stuff with the target element.
    console.log(event.target);
});
```

#### How to use keyboard shortcuts to navigate through nodes?

```js
tree.on('keyDown', (event) => {
    // Prevent the default scroll
    event.preventDefault();

    const node = tree.getSelectedNode();
    const nodeIndex = tree.getSelectedIndex();

    if (event.keyCode === 37) { // Left
        tree.closeNode(node);
    } else if (event.keyCode === 38) { // Up
        const prevNode = tree.nodes[nodeIndex - 1] || node;
        tree.selectNode(prevNode);
    } else if (event.keyCode === 39) { // Right
        tree.openNode(node);
    } else if (event.keyCode === 40) { // Down
        const nextNode = tree.nodes[nodeIndex + 1] || node;
        tree.selectNode(nextNode);
    }
});
```

#### How to select multiple nodes using the ctrl key (or meta key)?

You need to maintain an array of selected nodes by yourself. See below for details:

```js
let selectedNodes = [];
tree.on('click', (event) => {
    // Return the node at the specified point
    const currentNode = tree.getNodeFromPoint(event.x, event.y);
    if (!currentNode) {
        return;
    }

    const multipleSelectionMode = event.ctrlKey || event.metaKey;

    if (!multipleSelectionMode) {
        if (selectedNodes.length > 0) {
            // Call event.stopPropagation() to stop event bubbling
            event.stopPropagation();

            // Empty an array of selected nodes
            selectedNodes.forEach(selectedNode => {
                selectedNode.state.selected = false;
                tree.updateNode(selectedNode, {}, { shallowRendering: true });
            });
            selectedNodes = [];

            // Select current node
            tree.state.selectedNode = currentNode;
            currentNode.state.selected = true;
            tree.updateNode(currentNode, {}, { shallowRendering: true });
        }
        return;
    }

    // Call event.stopPropagation() to stop event bubbling
    event.stopPropagation();

    const selectedNode = tree.getSelectedNode();
    if (selectedNodes.length === 0 && selectedNode) {
        selectedNodes.push(selectedNode);
        tree.state.selectedNode = null;
    }

    const index = selectedNodes.indexOf(currentNode);

    // Remove current node if the array length of selected nodes is greater than 1
    if (index >= 0 && selectedNodes.length > 1) {
        currentNode.state.selected = false;
        selectedNodes.splice(index, 1);
        tree.updateNode(currentNode, {}, { shallowRendering: true });
    }

    // Add current node to the selected nodes
    if (index < 0) {
        currentNode.state.selected = true;
        selectedNodes.push(currentNode);
        tree.updateNode(currentNode, {}, { shallowRendering: true });
    }
});
```

## License

Copyright (c) 2016-2017 Cheton Wu

Licensed under the [MIT License](LICENSE).
