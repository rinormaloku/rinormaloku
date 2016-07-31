---
layout:            post
title:             "Polymer and Chrome Extensions"
menutitle:         "Polymer and Chrome Extensions"
category:          PolymerJS
author:            rinormaloku
tags:              PolymerJS Chrome Extension
---

{% raw %}

In this article I will go through the creation process of a simple Chrome Extension that contains PolymerJS elements.
It should be a reference point when starting a new project, as the most part is ceremonial as we will see. 

# Unlike - chrome extension
Our extension hates Liking. The process of clicking the same icon again and again by so many people around the world irritates it. So it developed two functions:
Function #1: Replaces Like icons with Dislike!
Function #2: If that didn’t discourage liking, by activating the Hate it will disable the possibility to click ‘Like’.

## 1. The Manifest file 
Manifest is a JSON file that is comprised of important information for chrome, which describes what the extension is going to do and what permissions it needs to.
Important links to get started: https://developer.chrome.com/extensions/getstarted
Manifest File format: https://developer.chrome.com/extensions/manifest
Important:
* **Page_action**: Our extension should be present only on 	current pages and is not applicable to all pages(if it would have been we would use browser_action).
* **Background**: specifies the scripts to be run in order to control chrome related functionality, like enabling and disabling the extension. 
* **Content_scripts**: Scripts to be run on pages. 
* **Permissions**: the permissions our extension needs.

## 2. Index.html
This file is a simple HTML file with the content need. Currently for test purposes we have a button with ID initConsoleMessage.

## 3. JS for the Index
We included the script applyOnExtension in Index.html so that we can add functionality to the page, like activating scripts on click of a button.

## 4. Apply JS on page
Using the **Content_scripts** property we defined that the script applyOnPage.js to be executed when we are on a page that matches: `https://www.facebook.com/*`

## 5. Even page JS
This file enables control of browser related functionalities. E.g. **show**-ing or **hide**-ing the extension.

## Communication messages between scripts
Communication between these scripts is done using messages. See Screen 1.
**Example:**
1.	When a page on the Facebook domain is opened `applyOnPage.js` is executed.
2.	applyOnPage sends a message with the action `show` and adds a listener for an `executeConsoleMessage` action, which we will use to print a console message.
3.	Message `show` is received (in eventPage.js) and plugin is enabled, using chrome api’s.

<aside>
   <figure class="right">
      <img src="/media/img/communication-messages.png" />
      <figcaption>Screen 1.</figcaption>
   </figure>
</aside>

## Executing scripts on the page on demand.
In order to execute a script when we interact with the plugin page (clicking buttons see fig.1.) we need to listen for the events from our extension JS (applyOnExtension.js) and then send messages to content Scripts (applyOnPage.js)
In this simplified example when the button "Print to console" is clicked:
1.	Action: `executeConsoleMessage` is send as a message.
2.	ApplyOnPage.js is listening for messages and if it is `executeConsoleMessage` it prints to the console.

<aside>
   <figure class="left">
      <img src="/media/img/printToConsole.png" />
      <figcaption>Screen 2.</figcaption>
   </figure>
</aside>

## Modifications to support the Unlike functionalities
* ‘paper-toggle-button‘ is a costum element and to use it we need bower installation. [For more information see the link](https://elements.polymer-project.org/guides/using-elements)
* * We imported webcomponentsjs and polymerjs
* *	Used the paper-toggle-button in index.html
* We added a new CSS content script that applies on a page of the ‘facebook.com/*’ domain. See file css/applyOnPage.css. This will replace all Like icons with dislike icons.
* applyOnPage.js – 
* *	We added functions and an observer that replace the ‘Like’ text with ‘Hate’. 
* * Replaced the code in the listener to listen for ‘turnHateOn’ and to disable or enable the ability to click on Likes according to the current state of ‘hateTogglerState’ (for storage api’s see this [video]()https://www.youtube.com/watch?v=tmzxMO9lrF0)
* * Direct call to ‘setUpLikeAbility’ is done when domain is matched.
* applyOnExtension.js:
* * When plugin is opened paper-toggle-button is set to the value of hateTogglerState from storage.
* * On each radio button click it stores the change in ‘hateTogglerState’ and sends a message ‘turnHateOn’

## Resolving Content Security Policy - Annoyance
Scripts in html code are hated. And that is the whole logic of webcomponents. Now if we load the chrome extension (how to load a chrome extension) we won't see the webcomponent.
**Vulcanize and crisper to the rescue:** 
Install them:
npm install -g vulcanize
npm install -g crisper

Execute: `vulcanize index.html --inline-script | crisper --html target/build.html --js target/build.js` 
and redirect manifest property `default_popup` to `target/build.html`
For more information on the magic see the linked [video](https://www.youtube.com/watch?v=VrajHIZZbE4).

Reload the unpacked extension, and Toggle the paper toggle button of HATE on!.
{% endraw %}    