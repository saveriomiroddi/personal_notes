# Javascript

- [Javascript](#javascript)
  - [Basic example](#basic-example)
  - [Data types](#data-types)
  - [Functions](#functions)
  - [Composite types/OO design](#composite-typesoo-design)
  - [Base APIs](#base-apis)
  - [Events](#events)
  - [Canvas](#canvas)
    - [Input](#input)
    - [Image (sprites)](#image-sprites)
    - [Audio](#audio)
  - [Snippets](#snippets)
    - [Random (color)](#random-color)

## Basic example

```js
// Important. Avoids some nasty stuff.
//
"use strict";

var backgroundColor, otherVar = 123;

start = function () {
  backgroundColor = "blue";
  changeBackgroundColor();
}

changeBackgroundColor = function () {
  document.body.style.background = backgroundColor;
}

document.addEventListener('DOMContentLoaded', start);
```

## Data types

Javascript is **weakly** typed.

```js
var uninit1, uninit2 = undefined;
uninit1 == uninit2;
uninit1 / 2 == undefined;

// automatic casting to float
//
10 / 20 == 0.5;

// boolan typecast to int (!)
//
true + 2 == 3;
false + 2 == 2;

// math ops don't generate errors! they return [-]Infinity or NaN
//
1 / 0 == Infinity
-1 / 0 == -Infinity
"not a number" / 2 // -> NaN (not equal to itself)

"abc" == 'abc'     // strings can be single- or double-quoted
`2 + 2 = ${2 + 2}` // template literals; they use backticks
```

## Functions

JS runtimes performs two passes; during the first, they interpret constructs unassigned functions:

```js
// valid
//
fx1();
function fx1() { }

// invalid!
fx2();
var fx2 = function () { }
```

Methods can be dynamically added to objects (!!):

```js
Canvas2D.clear = function () {
  Canvas2D.canvasContext.clearRect(0, 0, this.canvas.width, this.canvas.height);
};
```

## Composite types/OO design

```js
var object = {
  member1: "abc",
  member2: 1,
};

// Members can also be defined outside the variable definition.

object.member3 = false;

object.fx1 = function () {
  object.member2 = 2;
  alert(object.member2 + object.member3);
};

object.fx1();
```

## Base APIs

```js
document // web page; owns all the object

document.getElementById("id")
document.addEventListener("event", funktion)

alert("message");

var date = new Date;
d.getTime(); // time in ms since 1970

// Math
//
sin(radians); cos(radians); tan(radians); // they have `a-` versions
atan2(opposite, adjacent); // arctangent; it takes vertexes, so that if one is zero, it doesn't err
```

## Events

| Name               | Notes       |
| ------------------ | ----------- |
| `click`            | Mouse click |
| `DOMContentLoaded` |             |

## Canvas

```js
// This does not automatically update the canvas!
//
changeCanvasColor = function () {
  var canvas = document.getElementById("myCanvas");

  // Context is needed to draw; can be 2d or 3d.
  var context = canvas.getContext("2d");

  context.fillStyle = "blue";
  context.fillRect(0, 0, canvas.width, canvas.height);

  // Clear the defined rectangle of anything that was drawn on it.
  context.clearRect(0, 0, canvas.width, canvas.height);
}

document.addEventListener('DOMContentLoaded', changeCanvasColor);
```

### Input

Input is best handled via events.

```js
// Mouse move event.
//
document.onmousemove = function (event) {
  // `pageX/Y`: typically used, since they take into account the scrolling; essentially, it doesn't
  // count the invisible, scrolled (e.g. values are lower)
  //
  alert(`x: ${event.pageX}, y: ${event.pageY}`);

  // `clientX/Y`: less useful; they take into account scrolling
  //
  alert(`x: ${event.clientX}, y: ${event.clientY}`);
}
```

### Image (sprites)

```js
var sprite = new Image();

// Starts loading immediately (best to wait with events). Dimensions are set automatically.
//
sprite.src = "spring.png";
```

Each instantiated `Image` will load the source again. For performance reasons, it's best to load all the `Image`s ahead, and put them (for example) in an object.

Drawing:

```js
// There is no background/foreground concept; any new sprite overwrites the previous ones.
//
function drawImage(sprite, x, y) {
  // First, save the context...
  //
  canvasContext.save();

  // ... the transformations are applied to the images drawn before restore().
  //
  canvasContext.translate(x, y);
  canvasContext.rotate(rotation)

  // Origin is the relative point where the drawing starts. In this example, origin is a variable with
  // values equal to half width/height, so that the drawn sprite is centered.
  //
  canvasContext.drawImage(
    sprite, 0, 0, sprite.width, sprite.height,
    -origin.x, -origin.y, sprite.width, sprite.height
  );

  canvasContext.restore();
};
```

### Audio

```js
music = new Audio();
music.src = "snd_music.mp3";
music.volume = 0.4;

music.play();
```

## Snippets

### Random (color)

```js
// cheap way to get a random
new Date().getTime() % (1 << 24)

'#' + Math.floor(Math.random() * (1 << 24 - 1)).toString(16).padStart(6, '0');
```
