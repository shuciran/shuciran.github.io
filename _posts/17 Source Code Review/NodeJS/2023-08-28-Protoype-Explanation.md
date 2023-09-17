---
description: >-
  NodeJS Prototype Explanation
title:  NodeJS Prototype Hunting             # Add title here
date: 2023-08-28 08:00:00 -0600                           # Change the date to match completion date
categories: [17 SCR, NodeJS Prototype Explanation]                     # Change Templates to Writeup
tags: [scr, nodejs, prototype ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

Nearly everything in JavaScript is an object. This includes arrays, Browser APIs, and functions. The only exceptions are null, undefined, strings, numbers, booleans, and symbols.

Unlike other object-oriented programming languages, JavaScript is not considered a class-based language. In JavaScript the class keyword is a helper function that makes existing JavaScript implementations more familiar to users of class-based programming.

We'll demonstrate this by creating a class and checking the type.

```javascript
class Student {
    constructor() {
    this.id = 1;
    this.enrolled = true
	const newObject = {};
  }
    isActive() {
	
            console.log("Checking if active")
            return this.enrolled
    }
	
}

> s.isActive()
Checking if active
false

> s.isActive = function(){
... console.log("New isActive");
... return true;
... }
[Function (anonymous)]

> s.isActive()
New isActive
true

> s.__proto__.isActive()
Checking if active
undefined

> delete s.isActive
true

> s.isActive()
Checking if active
false

```

According to the documentation, JavaScript's *new* keyword will first create an empty object. Within that object, it will set the `__proto__` value to the constructor function's prototype (where we set isActive). With `__proto__` set, the new keyword ensures that this refers to the context of the newly created object. Previous commands shows that this.id and this.enrolled of the new object are set to the respective values. Finally, this is returned (unless the function returns its own object).

```javascript

newObject = {
	this.__proto__ = constructor() {this.id = 1; this.enrolled = true
	const newObject = {};} isActive() { console.log("Checking if active")
            return this.enrolled} 
};

```
