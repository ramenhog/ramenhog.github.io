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

My CodePen is just a fun little code experiment with draggable elements, but draggable user interfaces actually pop up a lot in the tech world. You can find draggable elements in file uploads or interfaces that allow the user to drag and dropping colors to fill in the color of a certain element.

While it is possible to accomplish this with some [JavaScript plugins](https://jqueryui.com/draggable/){:target="_blank"}, learning how to do this with pure vanilla JS can allow you to demystify some of the _magic_ behind those plugins/libraries and become more comfortable with JS fundamentals you can apply to other code problems. So, without further ado, let's delve into the basic methodology behind creating a draggable element.


## Some Basic CSS
Before delving into the JS side of things, there’s some basic CSS needed for a draggable element to work: the **draggable element** must be **absolutely positioned** inside its **relatively positioned** parent container. This allows us to reposition the element as it’s being dragged by changing its `top` and `left` style properties with JS.

{% highlight css %}
  .draggable {
    position: absolute;
    top: 50px;
    left: 80px;
  }

  .container {
    position: relative;
  }
{% endhighlight %}

## The General Gist

### 1. Get **initial click position**

<img src="/assets/step_1.jpg" alt="Get initial click position" class="image image--small" />

Clicking within the draggable element will fire off the entire dragging process. 

We can detect this click by attaching an event handler to the **mousedown** event on the draggable element. This event handler allows us to be able to use the [MouseEvent object](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/MouseEvent) to determine the x and y coordinates of that initial mousedown event. Using the `clientY` and `clientX` properties on the event object will return coordinates will be relative to the upper left edge of the viewport.

In this step, we also want to be able to keep track of whether or not the mouse is being held down, which will come in handy in Step 3. We can do this by changing the state of our  `isMouseDown` boolean.

We will set `isMouseDown` to `true` within the `mouseDown` event handler, which is only triggered when our mouse is being held down.

We will set `isMouseDown` to `false` in the `mouseUp` event handler, which is only triggered when our mouse is up.

{% highlight js %}
var isMouseDown = false;

function mouseDown(event) {
  // set mousedown state
  isMouseDown = true;

  var mouseY = event.clientY;
  var mouseX = event.clientX;
}

function mouseUp(e) {
  isMouseDown = false;
}

elm.addEventListener("mousedown", mouseDown);
elm.addEventListener("mouseup", mouseUp);
{% endhighlight %}


### 2. Get distance from initial click to element's top left corner

<img src="/assets/step_2.jpg" alt="Get distance from initial click to element's top left corner" class="image image--small" />

Within that same **mousedown** event handler, we then want to get the top left position of the element relative to its parent container and calculate the distance between these top left coordinates to the initial click coordinates.

{% highlight js %}
// get elm top and left coordinates
var elmTop = elm.offsetTop;
var elmLeft = elm.offsetLeft;

// get diff from element top left to mousedown point
var diffY = mouseY - elmTop;
var diffX = mouseX - elmLeft;
{% endhighlight %}


### 3. As mouse moves, get new mouse position

<img src="/assets/step_3.jpg" alt="As mouse moves, get new mouse position" class="image image--small" />

Now we’ll attach another event listener for **mousemove**. We will use the event object associated with this **mousemove** event to calculate the new coordinates for mouse position relative to the viewport. 

However, we only want to perform those calculations when the mouse is **both moving and clicked**, so we can use our handy `isMouseDown` boolean to check the mouse state and `return` out of the function if the mouse is not currently down. 

{% highlight js %}
function mouseMove(e) {
  // if mouse is not clicked, return and do nothing
  if (!isMouseDown) return;

  // get new mouse coordinates
  var newMouseY = e.clientY;
  var newMouseX = e.clientX;
}

elm.addEventListener("mousemove", mouseMove);
{% endhighlight %}

### 4. Determine new top left coordinates for element

<img src="/assets/step_4.jpg" alt="Determine new top left coordinates for element" class="image image--small" />

Since we already have the **new coordinates for the mouse position** relative to the viewport and we know the **distance the mouse is from the top left corner of the element**, we can determine the **new coordinates for the top left corner of the element** relative to viewport.

{% highlight js %}
var newElmTop = newMouseY - diffY,
    newElmLeft = newMouseX - diffX;
{% endhighlight %}

### 5. Move element to new calculated coordinates

<img src="/assets/step_5.jpg" alt="Move element to new calculated coordinates" class="image image--small" />

Now that we’ve done all the math, all we need to do is to actually move the element to those coordinates by changing the `top` and `left` style properties on the element to these new coordinates:

{% highlight js %}
elm.style.top = newElmY + 'px';
elm.style.left = newElmX + 'px';
{% endhighlight %}


## So we’re done?!
…not so fast. With the code that we have so far we’ll be able to drag our element around, sure, but we’ll also be able to drag it outside of its parent container:

<img src="/assets/out_of_container.jpg" alt="Element dragged out of container" class="image image--small" />
_Not sure you need this illustration but here you go_

## So how do we keep the element from being dragged outside the parent container?
We can fix that with these few last steps:

### 1. Get parent container dimensions

In order to determine if our element is getting dragged outside its parent, we'll need to first get the parent container's dimensions:

{% highlight js %}
const container = elm.offsetParent;
containerWidth = container.offsetWidth;
containerHeight = container.offsetHeight;
{% endhighlight %}

### 2. Check if element is within top & left container bounds

We will know that our draggable element is outside of the parent container if the **new top or left element coordinate is less than 0**. 

If our element is being dragged off the top of the container, we should set the new top coordinate to 0 (the topmost coordinate possible). If our element is being dragged off the left of the container, we should also set the new left coordinate to 0. 

{% highlight js %}
// if elm is being dragged off top of the container...
if (newElmTop < 0) {
  newElmTop = 0;
}

// if elm is being dragged off left of the container...
if (newElmLeft < 0) {
  newElmLeft = 0;
}
{% endhighlight %}

### 3. Get new bottom and right element coordinates

We have the new top left coordinates, but we also need the bottom right element coordinates if we’re going to be able to determine if the element is within the bottom and right container bounds.

We can get these coordinates by **adding the height of the element to the new top element coordinate** and the **width of the element to the new left element coordinate**.

{% highlight js %}
var newElmRight = newElmTop + elmHeight,
    newElmBottom = newElmLeft + elmWidth;
{% endhighlight %}

### 4. Check if element is within bottom & right container bounds

Using the bottom and right element coordinates we just calculated, we will know that the element will be dragged off the bottom or right bounds of the container if the **right element coordinate is greater than the container height** or the **left element coordinate is greater than the container width**.

If the element is being dragged off the **bottom**, we want to set the new **top** element coordinate to the **topmost possible position** without the element going out of bounds. We can get this value by **subtracting the element height from the container height**.

Same thing goes for if the element is being dragged off the **right**. We will want to set the new **left** element coordinate to the **leftmost possible position** without the element going out of bounds. We can get this value by **subtracting the element width from the container width**.

{% highlight js %}
// if elm is being dragged off bottom of the container...
if (newElmRight > containerHeight) {
  newElmTop = containerHeight - elmHeight;
}

// if elm is being dragged off right of the container...
if (newElmBottom > containerWidth) {
  newElmLeft = containerWidth - elmWidth;
}
{% endhighlight %}

And now, you should be able to drag your element anywhere within the boundaries of your container.

## Extra credit
And that’s it! So those are the basic steps to creating a draggable element without it going out of the bounds of its parent container. But there are many ways you can expand on that. 

In my draggable buddy CodePen, his face also changes depending on whether he’s moving, still, or being bumped up against the side of the container. And when he’s bumped more than five times, he “faints” for 3 seconds before coming back to life. All of these states are achieved by just adding some conditional logic with the calculations we already walked through.

I hope you found this tutorial helpful and now it’s your turn to take these basic components and expand on them to create your own draggable buddy or some other draggable interface with vanilla JS! Enjoy!

<img src="/assets/world-is-your-oyster.gif" alt="The world is your oyster" class="image image--small" />
