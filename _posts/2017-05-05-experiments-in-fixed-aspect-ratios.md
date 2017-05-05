---
layout: post
title: "Experiments in fixed aspect ratios"
date: 2017-05-04
description: Utilizing mouse events and math to create elements a user can drag with their mouse with vanilla JavaScript
image: 
  twitter: /assets/move-me.gif
  facebook: /assets/move-me.gif
---

When I was TA-ing a Responsive Web Design class for Girl Develop It a few weeks ago, a question was raised as to how to keep video embeds from losing their aspect ratio responsively, so I thought I'd do a quick write up on maintaining embed aspect ratios in a responsive site.

<!--more-->

Responsive video embeds are tricky because unlike images which are easy to scale with `height: auto`, iframe embeds must have a specified height to avoid it from collapsing since it's unable to base its height on its contents. However, a specified height combined with a responsive percentage-based width leads to awkward iframe responsive resizing, especially when it comes to videos. 

We can get around this problem by wrapping our embed in a parent container with the proper aspect ratio and then stretching our embed to fill those container dimensions. Typically the most accepted technique to do this involves a nifty trick with the parent container padding.

### The Tried &amp; True Method

There's a few variations of this method floating around the internet, but they all involve using padding to create a parent container with the desired aspect ratio. 

The cool thing about using padding to dictate height instead of utilizing the `height` property itself is that unlike percentage heights where the [percentage is calculated with respect to the height of the generated box's containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/height){:target="_blank"}, padding percentages [refer to the width of the containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/padding?v=control){:target="_blank"}. This way we can now make the height relative to the width of the element. 

So, knowing that, we can achieve a container element with the proper aspect ratio by setting `height` to 0 and setting `padding-top` to `height/width * 100%`. So, for a container with an aspect ratio of 16:9 (the standard Youtube video aspect ratio), where the markup is like this:

{% highlight html %}
<div class="container">
  <iframe class="embed"></iframe>
</div>
{% endhighlight %}

The CSS for the parent container would look like so:

{% highlight css %}
.container {
  position: relative;
  width: 100%;
  height: 0;
  padding-top: 56.25% // 9 / 16 * 100%
}
{% endhighlight %}

Now we'll simply need to stretch the embed to take up the space of its parent with absolute positioning:

{% highlight css %}
.embed {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
{% endhighlight %}

So as the viewport shrinks responsively, the embed will always retain the proper aspect ratio that is defined in its container. You can see the final product of this technique in action here:

<p data-height="510" data-theme-id="28475" data-slug-hash="65d6ea9b7fa0a3f5d33d2f9fed067a3d" data-default-tab="css,result" data-user="ramenhog" data-embed-version="2" data-pen-title="Fixed aspect ratio video embeds - padding trick" class="codepen">See the Pen <a href="http://codepen.io/ramenhog/pen/65d6ea9b7fa0a3f5d33d2f9fed067a3d/">Fixed aspect ratio video embeds - padding trick</a> by Stephanie (<a href="http://codepen.io/ramenhog">@ramenhog</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

A variation to this approach is to define a psuedo-element on the container and dictate the aspect ratio there instead of on the container element itself:

{% highlight css %}
.container {
  position: relative;
  width: 75%;

  &:before {
    content: "";
    display: block;
    width: 100%;
    padding-top: 56.25%;
  }
}
{% endhighlight %}

This variation comes in handy when you don't want the embed container to be 100% width of it's containing block. By dictating the aspect ratio on the psuedo-element, you don't need to worry about recalculating the padding-top percentage whenever the container width changed. One thing to keep in mind, however, is that if the container has `display: flex`, this pseudo-element approach doesn't play as well with Firefox.

## Well, what about CSS Grid?

So, while I was halfway done with creating that first CodePen demo, a little thought popped into my head: "Is there a way to do this with CSS Grid?" One of the great things about Grid is that it allows you to two-dimensional control over the grid's child elements, which is precisely what we need to control aspect ratio. So I decided to do a little experimenting and came up with this demo:

<p data-height="381" data-theme-id="28475" data-slug-hash="196e49797107763ba27a91490948ac81" data-default-tab="css,result" data-user="ramenhog" data-embed-version="2" data-pen-title="Fixed aspect ratio video embed experiment with CSS Grid" class="codepen">See the Pen <a href="http://codepen.io/ramenhog/pen/196e49797107763ba27a91490948ac81/">Fixed aspect ratio video embed experiment with CSS Grid</a> by Stephanie (<a href="http://codepen.io/ramenhog">@ramenhog</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

The experimental approach I took was to create a container grid with the number of columns dictated by the **width** number in the aspect ratio. In the demo, I'm working with a **16:9** aspect ratio, so my grid container has 16 columns.

I utilized the `vw` unit for these column widths, making each column a percentage of the viewport width which allows the columns to adjust with the viewport width making them automatically responsive.

I also set `grid-auto-row` to the same size as the columns, making each grid cell a square with a 1:1 aspect ratio. 

Then, on the child element, which is the video embed, I spanned the element across 16 cells horizontally and 9 cells vertically. This allows us to directly give our video embed the desired aspect ratio.

