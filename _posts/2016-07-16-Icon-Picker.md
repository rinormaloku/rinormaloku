---
layout:            post
title:             "Icon Picker - Introduction to Web Components"
menutitle:         "Icon Picker - Introduction to Web Components"
category:          Programming
author:            rinormaloku
tags:              PolymerJS WebComponents
---

{% raw %}
Lately PolymerJS is making rounds around front end developers and it got me interested. To understand it I started working on a simple project. One of the simpler components used in 
those projects was `<icon-picker>` It's purpose is to enable selection of one icon, from a bunch that are listed.

<aside>
   <figure class="right">
      <img src="/media/img/1Iron-Icon.png" />
      <figcaption>Screen 1.</figcaption>
   </figure>
</aside>

Icon-picker tries to adhere to the **Ten Principles for Great General Purpose Web Components** [[Link]](https://github.com/basic-web-components/components-dev/wiki/Ten-Principles-for-Great-General-Purpose-Web-Components), but there are still possible improvements, which are at the current level of my understanding hard to distinguish.


Complete code [^1].

<aside>
   <figure class="right">
      <img src="/media/img/2icon-picker.png" />
      <figcaption>Screen 2.</figcaption>
   </figure>
</aside>

### **Icon Picker**


1. In the first screen we see an element that when clicked opens the icon-picker component.
2. The second screen is the Icon Picker component which displayes a couple icons for selection.
3. Icon is selected and an event is fired. which is listened for and done whatever functionality is desired, in this case we wanted to apply the selected icon to one of the Iron Icons.
<aside>
   <figure class="right">
      <img src="/media/img/3selected-icon.png" />
      <figcaption>The result: Screen 3.</figcaption>
   </figure>
</aside>


### **Web Components and PolymerJS**

WebComponents enable creation of custom elements, with the purpose of increasing reusability and mimicking object oriented programming by applying separation of concerns where Style, Markup and Logic is in one place.


PolymerJS is a platform that enables creation of custom elements, that we will be using to create the Icon Picker.


### Step 1. The Element Stub 


```html
<link rel="import" href="../polymer/polymer.html">

<dom-module id="icon-picker">
  <template>
    <style>
      :host {
        display: block;
      }
    </style>
    <h2>Hello [[prop1]]</h2>
  </template>

  <script>
    Polymer({

      is: 'icon-picker',

      properties: {
        prop1: {
          type: String,
          value: 'icon-picker',
        },
      },

    });
  </script>
</dom-module>
```
This code is automatically generated using Polymer CLI, [check out this link on how to generate a custom element](https://www.polymer-project.org/1.0/docs/tools/polymer-cli). 

The functionality currently is that each time `<icon-picker></icon-picker>` is used on the page we get a simple `<h2>Hello icon-picker</h2>`

### Step 2. Updating the Stub HTML
1. The icon picker needs to be called on demand. To do so we make use of the `paper-dialog` element, which has methods open() and close().
2. When `paper-icon-button` is clicked we call **showIconSelection()** which opens the `paper-dialog`, more on that later.
3. A `div` to contain a title and some styling for it.
4. Template **dom-repeat** that displays all Icons to be selected.

```html
<paper-icon-button id="inputIcon" icon="editor:insert-emoticon"
				   on-click="showIconSelection" prefix></paper-icon-button>

<paper-dialog id="theIconDialog">
	<div class="titleCont">
		<div>{{title}}</div>
	</div>
	<div id="repositionImages">
		<template is="dom-repeat" items="{{iconNames}}">
			<iron-icon icon="{{item}}" prefix></iron-icon>
		</template>
	</div>
</paper-dialog>
```


### Step 3. Adding the element Properties
* **_selectedIcon** : This property is set when one of the items is selected and it is reflected back to the attribute. [Information on properites](https://www.polymer-project.org/1.0/docs/devguide/properties)
* **iconNames** : The names which will populate the iron-icon elements when repeated. *template dom-repeat will repeat it's contents N times, where N is the number of elements in iconNames (Step 2: Line 7)*.
* **title** : The title that you want to display on the `paper-dialog`

```html
properties: {
	_selectedIcon: {
		type: String,
		value: '',
		reflectToAttribute: true
	},
	iconNames: {
		type: Array,
		value: [],
		notify: true
	},
	title:{
	        type: String,
		value: 'placeholder'
	}
},
```

**Example of populating the properties of this element:**

```html
<icon-picker id="icondialog" 
             icon-names='["android", "announcement"]' 
             title="Select one icon"></icon-picker>
```

### Step 4. Adding event Listeners:

When one of the Icons is clicked we want to set the **_selectedIcon** property to it's value. 
In the below code the listener listens for a tap event on theIconDialog. It would trigger even when empty space is clicked. (There must be a better solution but not with my current understanding of JS)

```json
listeners: {
	'theIconDialog.tap': 'selectionMade'
},
```    

### Step 5. Adding the instance methods:
* **ready**: Will execute after property values are set. If user didn't add his desired icons he will have 4 placeholders.
* **open**: carries the open call over to the paper-dialog. It is being accessed by it's ID.
* **selectionMade**: when a tap event is fired this function checks if it was on an `iron-icon` element. Then sets _selectedIcon to it's _iconName attribute, fires an event 'event-icon-selected' and closes the paper-dialog.
* **showIconSelection**: opens the paper-dialog.
* **_reset**: resets icon back to *editor:insert-emoticon*

```javascript
ready: function () {
	if (this.iconNames.length == 0) {
		this.iconNames = [
			'android',
			'book',
			'announcement',
			'3d-rotation',
		];
	}
},
open: function () {
	this.$.theIconDialog.open();
},
selectionMade: function (iconClicked) {
    console.debug(iconClicked);
    if (iconClicked.target.nodeName == 'IRON-ICON') {
        this._selectedIcon = '' + iconClicked.target._iconName;
        this.theIconDialog = this._selectedIcon;
        this.$.inputIcon.setAttribute('icon', this._selectedIcon);
        this.fire('event-icon-selected', this._selectedIcon);
        this.$.theIconDialog.close();
    }
},
showIconSelection: function () {
    this.$.theIconDialog.open();
},
_reset: function () {
    this.$.inputIcon.setAttribute('icon', 'editor:insert-emoticon');
}
```


### Step 6. Some styling :)
```css
:host {
	display: block;
}
#theIconDialog {
	max-width: 585px;
}
iron-icon {
	opacity: 0.7;
	padding: 15px;
}
iron-icon:hover {
	opacity: 0.6;
	color: #00c853;
}
iron-icon:active {
	opacity: 1;
}
#repositionImages {
	display:inline-block;
	text-align:center;
}
.titleCont{
	width: auto;
	background-color: #0d47a1;
	margin-top: -20px;
	padding: 25px 25px 25px 45px;
}
.titleCont div{
	color: white;
	font-size: large;
}
```


## Final notes:
Our element is finished. Anytime we need an icon-picker in any project all we have to do is bower install `rinormaloku/icon-picker` and import it:

```html
    <link rel="import" href="../icon-picker.html">
```

Read this [article on how to enable 'bower install'](https://www.polymer-project.org/1.0/docs/tools/reusable-elements)

[^1]: [Complete code for the component](https://github.com/rinormaloku/icon-picker)

{% endraw %}
