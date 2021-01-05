# HTML/CSS

- [HTML/CSS](#htmlcss)
  - [HTML](#html)
    - [General structure](#general-structure)
    - [Snippets](#snippets)
      - [View/Hide sections](#viewhide-sections)
  - [CSS](#css)

## HTML

### General structure

```html
<!DOCTYPE html>
<html>
  <head>
    <title>This is the title!</title>
    <meta charset="UTF-8">

    <!-- External stylesheet file -->
    <link rel="stylesheet" type="text/css" href="my.css"/>

    <!-- External javascript file -->
    <script src="my.js"></script>

    <!-- Inline javascript -->
    <script>
      // My script!
    </script>
  </head>

  <!-- Inline styling -->
  <body style="background:blue">

  <!-- Divs are just page divisions -->
  <div id="gameArea">
    <canvas id="myCanvas" width="800" height="480"></canvas>
  </div>
  </body>
</html>
```

### Snippets

#### View/Hide sections

Usable on GitHub; additional tags like `target` and `rel` have been stripped when testing:

```html
<details>
  <summary>Chapter 15</summary>
  <a href="/assets/readme/chapter15_astronaut.png?raw=true">
    <img src="/assets/readme/chapter15_astronaut.png?raw=true" width="800" style="max-width:100%;">
  </a>
</details>
```

## CSS

```css
html, body {
  margin: 0;
}
```
