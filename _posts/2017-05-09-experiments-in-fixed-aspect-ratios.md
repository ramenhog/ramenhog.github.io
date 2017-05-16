---
layout: post
title: "Experiments in fixed aspect ratios"
date: 2017-05-09
description: Revisiting responsive aspect ratios techniques with CSS Grid
image: 
  twitter: /assets/responsive-aspect-ratio.gif
  facebook: /assets/responsive-aspect-ratio.gif
---

When I was TA-ing a Responsive Web Design class for [Girl Develop It](https://www.girldevelopit.com/){:target="_blank"} several weeks ago, a great question was raised as to how to keep video embeds from losing their aspect ratio responsively. So I thought I'd do a quick post on how to maintain embed aspect ratios in a responsive site. And maybe I could even brush some dust off the accepted solution to this problem, especially now that we have CSS Grid ✨.

<!--more-->

Responsive video embeds are tricky because, unlike images which are easy to scale with `height: auto`, iframe embeds must have a specified height to avoid it from collapsing since it's unable to base its height on its contents. However, a specified height combined with a responsive percentage-based width leads to awkward iframe responsive resizing, especially when it comes to videos. 

We can get around this problem by wrapping our embed in a parent container with the proper aspect ratio and then stretching our embed to fill those container dimensions. Typically the most accepted technique to achieve this involves a nifty trick with the parent container padding.

### The Tried &amp; True Method

There's a few variations of this method floating around the internet, but they all involve using padding to create a parent container with the desired aspect ratio. 

The cool thing about using padding to dictate height instead of utilizing the `height` property itself is that padding percentages [refer to the width of the containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/padding?v=control){:target="_blank"} unlike percentage heights where the [percentage is calculated with respect to the height of the generated box's containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/height){:target="_blank"}, . This way we can now make the height relative to the width of the element. 

Knowing that, we can achieve a container element with the proper aspect ratio by setting `height` to 0 and setting `padding-top` to `height/width * 100%`. So, for a container with an aspect ratio of 16:9 (the standard Youtube video aspect ratio), the markup will look like this:

{% highlight html %}
<div class="container">
  <iframe class="embed"></iframe>
</div>
{% endhighlight %}

And the CSS for the parent container would look like this:

{% highlight css %}
.container {
  position: relative;
  width: 100%;
  height: 0;
  padding-top: 56.25% /* 9 / 16 * 100% */
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

<p data-height="565" data-theme-id="28441" data-slug-hash="65d6ea9b7fa0a3f5d33d2f9fed067a3d" data-default-tab="css,result" data-user="ramenhog" data-embed-version="2" data-pen-title="Fixed aspect ratio video embeds - padding trick" class="codepen">See the Pen <a href="http://codepen.io/ramenhog/pen/65d6ea9b7fa0a3f5d33d2f9fed067a3d/">Fixed aspect ratio video embeds - padding trick</a> by Stephanie (<a href="http://codepen.io/ramenhog">@ramenhog</a>) on <a href="http://codepen.io">CodePen</a>.</p>
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

So, while I was halfway done with creating that first CodePen demo, a little thought popped into my head: "Is there a way to do this with CSS Grid?" One of the great things about Grid is that it allows you to two-dimensional control over the grid's child elements, which is precisely what we need to control aspect ratio. So I decided to conduct my own experiment.

<img src="/assets/secret-lab.gif" alt="To the secret lab!" class="image image--small" />

## Caution. CSS Grid Experiment in Progress

The approach I took was to create a container grid with the number of columns dictated by the **width** number in the aspect ratio. In the demo, I'm working with a **16:9** aspect ratio, so my grid container has 16 columns.

I utilized the **vw** unit for these column widths, making each column a percentage of the viewport width which allows the columns to adjust with the viewport width. To get the size of the columns, I took the **container to viewport percentage divided by the number of columns** to get the actual size of the columns in **vw**.

I also set our grid row size to the same size I calculated for the columns, making each grid cell a square with a 1:1 aspect ratio. Instead of implicitely setting the number of rows, I used `grid-auto-rows` so our grid would be smart and automatically create new rows with our set size as needed.

{% highlight css %}
.container {
  display: grid;
  grid-template-columns: repeat(16, 5.625vw);
  grid-auto-rows: 5.625vw;
}
{% endhighlight %}

Now all I have to do, on the child video embed grid element, is to span the element across the number of grid cells horizontally and vertically according to our desired aspect ratio. And since iframe embeds need a declared height and width property, we can set those to 100%, to fill that grid area.

{% highlight css %}
.embed {
  grid-column: span 16;
  grid-row: span 9;
  width: 100%;
  height: 100%;
}
{% endhighlight %}

I actually really ended up liking how intuitive this approach was for me. Here, instead of creating a container with the appropriate aspect ratio, you set the aspect ratio on the embed itself.

<p data-height="370" data-theme-id="28441" data-slug-hash="196e49797107763ba27a91490948ac81" data-default-tab="css,result" data-user="ramenhog" data-embed-version="2" data-pen-title="CSS Grid Experiment: fixed aspect ratio video embed" class="codepen">See the Pen <a href="http://codepen.io/ramenhog/pen/196e49797107763ba27a91490948ac81/">CSS Grid Experiment: fixed aspect ratio video embed</a> by Stephanie (<a href="http://codepen.io/ramenhog">@ramenhog</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

The funkiest part of this approach is the grid cell size calculations. Since the size must be based on the viewport width to create responsive 1:1 grid cells, the math may get a little opaque if your video embed is nested within several ancestor elements.

You would also have to recalculate the cell size if you would like to change the overall width of that container at different break points. I used CSS custom properties to make this a bit less painless in my demo and simply set the `--cellSize` property to a new value at different breakpoints. I then used that custom property to set my grid column and row sizes.

Also, to limit your video embed at a max size, you would have to set your cell sizes to a fixed **pixel** value at a certain breakpoint, which can also be kind of a headache.

Still, I appreciate CSS Grid for giving us some new approaches to age old problems like this one. Dictating the aspect ratios on the embed itself is actually how I would intuitively expect aspect ratios to be handled in CSS. And it seems like I'm not the only one with CSS Grid aspect ratio expectations. In my research, I came across this CSS spec request from one of my CSS Grid gurus: [Jen Simmons](https://twitter.com/jensimmons?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor){:target="_blank"}.

> I wish big time there were a way to transfer a calculated size from one dimension to the other. AKA… I define columns to be 1fr each. And I want the cells to be squares (or always have a 16x9 ratio, or whatever — but to maintain a fixed aspect ratio). How can I define the rows to be 1fr-from-the-column-calculation?? I want that.
_Jen Simmons, [https://github.com/w3c/csswg-drafts/issues/333](https://github.com/w3c/csswg-drafts/issues/333){:target="_blank"}_

As powerful as CSS Grid already is now, having a spec like that would be a game changer. CSS Grid containers would almost resemble graph paper, which would give us even more control over sizing grid elements two dimensionally. Maintaining responsive aspect ratios would be a breeze.

But, for now, especially taking [CSS Grid browser support](http://caniuse.com/#feat=css-grid){:target="_blank"} into consideration, the "tried and true" method, is still your best bet for most situations that involve maintaining aspect ratios. It hasn't failed me yet! But as CSS Grid begins to mature and expand its capabilities, here's to hoping for more exciting CSS developments relating to aspect ratios in the future!

<img src="/assets/fingers-crossed.gif" alt="Fingers crossed!" class="image image--small" />

P.S. If you're in need of a quick break, check out that [short film](https://www.youtube.com/watch?v=upPCohrJcbw){:target="_blank"} in my CodePens. It's 🔥!



