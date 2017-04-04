---
layout: post
title: "Creating a draggable element with JavaScript"
date: 2017-03-20
description: Utilizing mouse events and math to create elements a user can drag with their mouse with vanilla JavaScript
image: 
  twitter: /assets/move-me.gif
  facebook: /assets/move-me.gif
---

Recently, I made this cute lil CodePen buddy in an attempt to learn more about creating draggable UI elements with vanilla JS. Sure, I could have used [jQuery UI](https://jqueryui.com/draggable/){:target="_blank"}, but what's the fun in that? 😬 Basically, you can drag him all around his container, but he hates getting bumped against the sides. If you do it more than five times, he even faints.

<!--more-->

<p data-height="471" data-theme-id="28475" data-slug-hash="gmGzRQ" data-default-tab="result" data-user="ramenhog" data-embed-version="2" data-pen-title="Move Me: draggable element without jQuery UI" class="codepen">See the Pen <a href="http://codepen.io/ramenhog/pen/gmGzRQ/">Move Me: draggable element without jQuery UI</a> by Stephanie (<a href="http://codepen.io/ramenhog">@ramenhog</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

This post is just going to be a general walkthrough on how I approached using vanilla JS to drag an element with a mouse. If you want to dive deeper into my code, feel free to [check out the code on CodePen](https://codepen.io/ramenhog/pen/gmGzRQ){:target=“_blank”}.

## The General Gist
### 1. Get initial mousedown coordinates

Clicking with an element will fire off the entire dragging process. By adding a **mousedown** event listener to the draggable element, we are able to use the event object to determine the x and y coordinates of that initial mousedown event. Using the `clientY` and `clientX` properties on the event object will return coordinates will be relative to the upper left edge of the viewport.

{% highlight js%}
function mouseDown(e) {
  var mouseY = e.clientY;
  var mouseX = e.clientX;
}
elm.addEventListener("mousedown", mouseDown);
{% endhighlight %}


### 2. Get distance from element’s top left corner to mousedown coordinates

Within that same **mousedown** event handler, we then want to calculate the distance between the draggable element’s top left corner to the coordinates we calculated in step 1.

{% highlight js%}
// get elm top and left coordinates
var elmY = elm.offsetTop;
var elmX = elm.offsetLeft;

// get diff from (0,0) to mousedown point
var diffY = mouseY - elmY;
var diffX = mouseX - elmX;
{% endhighlight %}

### 3. As the mouse moves while being clicked, get new mouse coordinates

We then want to attach another event listener for **mousemove**, but we’re placing it within the **mousedown** event listener because we only want the event handler to be fired when the mouse is **both moving and clicked**. We will use the event object associated with the **mousemove** event to calculate the new coordinates for mouse position relative to the viewport.

### 4. Determine new top left corner coordinates for the element

Since we already have the **new coordinates for the mouse position** relative to the viewport and we know the **distance the mouse is from the top left corner of the element**, we can determine the **new coordinates for the top left corner of the element** relative to viewport.

{% highlight js%}
var newElmY = newMouseY - diffY,
    newElmX = newMouseX - diffX;
{% endhighlight %}

### 5. Move element to new coordinates

Now that we’ve done all the math, all we need to do is to actually move the element to those coordinates:

{% highlight js%}
elm.style.top = newElmY + 'px';
elm.style.left = newElmX + 'px';
{% endhighlight %}


## Make it yours
Those are the five main steps to creating a basic draggable div. But how can we expand on that?

In my example, I chose to restrict the div from moving outside it’s container. His face also changes depending on whether he’s moving, still, or being bumped up against the side of the container. And when he’s bumped more than five times, he “faints” for 3 seconds before coming back to life.

That’s what  I chose to do with a draggable element, but the world is your oyster. Let me know what you come up with!

<img src="/assets/world-is-your-oyster.gif" alt="The world is your oyster" class="image image--small" />
