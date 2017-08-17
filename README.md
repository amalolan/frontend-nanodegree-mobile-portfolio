# Website Performance Optimization portfolio project

In this project, I tried to optimize an extremely unoptimized website to achieve
a PageSpeed Score of above 90 and 60fps consistently on a page.


## Installation

Either visit https://amalolan.github.io/frontend-nanodegree-mobile-portfolio/index.html and https://amalolan.github.io/frontend-nanodegree-mobile-portfolio/views/pizza.html
or clone the repository and open index.html and views/pizza.html

## Optimizations and Changes

### index.html


The index page originally had a [Google PageSpeed](https://developers.google.com/speed/pagespeed/insights/) score of 35/100 for mobile and 47/100 for desktop.

I optimized it and brought the score up to [94/100 for mobile and 95/100 for desktop.](https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Famalolan.github.io%2Ffrontend-nanodegree-mobile-portfolio%2F)

#### Optimizations for index.html

1. The web font was taking making the site too slow so I was forced to remove it.

2. I in-lined the style.css file since the file wasn't too Large

3. I made the print.css file only load when the page was to be printed

4. I made the google analytics script run asynchronously by adding the `async` attribute

5. I moved the in-line script which ran the analytics to the bottom to run after the document loaded.

6. Finally, I optimized the pizzeria picture by compressing and reducing its size

After all those optimizations, here's how the head looks without any of the comments:

```
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content=" Very Nice ">
  <meta name="author" content=" amalolan ">
  <title>Cameron Pittman: Portfolio</title>

  <style>
    <!-- The inlined css goes here -->
  </style>
  <link href="css/print.css" rel="stylesheet" media="print">
  <script async src="js/perfmatters.js"></script>
  <script async src="https://www.google-analytics.com/analytics.js"></script>
</head>
```

And here is the google analytics script which runs after the document is loaded.

```
<script>
  document.addEventListener("DOMContentLoaded", function(event) {
    (function(w,g){w['GoogleAnalyticsObject']=g;
    w[g]=w[g]||function(){(w[g].q=w[g].q||[]).push(arguments)};w[g].l=1*new Date();})(window,'ga');

    ga('create', 'UA-XXXX-Y');
    ga('send', 'pageview');
  });
</script>
```

### Pizza slider (views/pizza.html)

The pizza page contains a pizza slider which displays pizzas based on the size set by the slider (Small, Medium, or Large). Initially, the time taken for the pizzas to switch from one size to another was very large (around 100ms). I optimized it so that it only took an average of 1ms while retaining all the same functionality.

#### Optimizations for Pizza Slider
1. To achieve this, I modified the resizePizzas() function in views/js/main.js

2. I moved the sizeSwitcher() function out of the determineDx() function's body.

3. I removed the determineDx() function as it was pretty much useless.

4. I multiplied each returning value in the sizeSwitcher() to express everything as percents

5. I reduced code repetition changePizzaSizes() and optimized it.

Here is the resizePizzas() function after all the modifications without the comments:
```
var resizePizzas = function(size) {
  window.performance.mark("mark_start_resize");
  function changeSliderLabel(size) {
    switch(size) {
      case "1":
        document.querySelector("#pizzaSize").innerHTML = "Small";
        return;
      case "2":
        document.querySelector("#pizzaSize").innerHTML = "Medium";
        return;
      case "3":
        document.querySelector("#pizzaSize").innerHTML = "Large";
        return;
      default:
        console.log("bug in changeSliderLabel");
    }
  }

  changeSliderLabel(size);


  function sizeSwitcher (size) {
    switch(size) {
      case "1":
        return 25;
      case "2":
        return 33.33;
      case "3":
        return 50;
      default:
        console.log("bug in sizeSwitcher");
    }
  }

  function changePizzaSizes(size) {
    var randomPizzas = document.querySelectorAll(".randomPizzaContainer");
    var newwidth = (sizeSwitcher(size)) + '%';
    for (var i = 0; i < randomPizzas.length; i++) {
      randomPizzas[i].style.width = newwidth;
    }
  }

  changePizzaSizes(size);

  window.performance.mark("mark_end_resize");
  window.performance.measure("measure_pizza_resize", "mark_start_resize", "mark_end_resize");
  var timeToResize = window.performance.getEntriesByName("measure_pizza_resize");
  console.log("Time to resize pizzas: " + timeToResize[timeToResize.length-1].duration + "ms");
};
```


### Making the scrolling in pizza.html a constant 60fps

In the beginning, scrolling in pizza.html felt very painful. The scripting required for
loading took around 18ms for 10 frames on average. I reduced this amount to under 1ms for 10 frames and made the experience very smooth at 60fps constantly

#### Optimizations for smooth scrolling in pizza.html

1. In the updatePositions() function in views/js/main.js, I optimized the for loop and removed the line in the for loop which didn't depend on the iteration and hence could be moved outside.

2. In the function executed after the document is ready in views/js/main.js at line 519, I reduced the number of pizzas from 200 to 50 since 50 was enough to cover the viewport.

3. In views/css/style.css I modified the `.mover`'s attribute by elevating it to another layer

Here is the updatePositions() function after modifications without the comments:
```
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");
  var scrollTop = (document.body.scrollTop / 1250);
  var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin(scrollTop + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}
```

Here is the modified function at line 519
```
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  for (var i = 0; i < 50; i++) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }
  updatePositions();
});
```

And finally, here is the .mover from style.css

```
.mover {
  position: fixed;
  width: 256px;
  z-index: -1;

  transform: translateZ(0);
  will-change: transform;
}
```
<br>

##### And that's it!! Those were all the optimizations I made to make the website better.

# The End
