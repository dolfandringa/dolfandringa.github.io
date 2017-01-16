---
layout: post
title: Drawing images and interactively change text on a canvas using Angular JS.
category: blog
date: "2017-01-16"
tags: 
    - angularjs
    - web-application
    - website
    - development
    - canvas
    - html5
---
So I love [AngularJS](https://angularjs.org/) (or Angular as [you're supposed to call it now](https://www.infoq.com/news/2016/12/angular-4)) for creating the interfaces of web-applications. I have really started to decouple the interfaces I make from the backend, by using Angular for the front-end and [flask](http://flask.pocoo.org) to create a RESTful backend with which Angular communicates. It's awesome. Right now I am making a web-interface that draws an image and dynamically changes numbers in that image depending on data it fetches from the backend. It is used in this case to show an image of a Solar Power system and dynamically change numbers in the image, showing the current state (battery capacity, panel output, power consumption, etc) of their system. I am using Angular for this and the [HTML5 Canvas](http://www.w3schools.com/html/html5_canvas.asp) object.

What I'm trying to do is load a static png image in the canvas, and draw text in that canvas as well that is stored in the scope of the controller. This is why I like Angular. You just define a variable in your controller, and whenever that value changes in the controller, automatically the web-interface changes according to the changing data. No complicated event handling necessary (well, Angular does that for you). This is a screenshot of the finished version of what I was trying to do:

* ![Dynamically changing text in solar system image](/images/solar_state_canvas_angular.png)

All the numbers in the drawing change dynamically with changing data from a web-service, but are located within the canvas that the png image is in too, so there is not html/css positioning nightmare.

When writing this interface, I ran into a number of problems that I thought would be useful to share the solution about. All of my solutions were the result of Google magic, especially due to [StackOverflow](http://stackoverflow.com), so major credit really goes to the community, and be sure to hone your google skills before asking others to help you.

Before I go over the specific problems I had and how I solved them, here is a complete working example, including comments, of the final solutions that I came up with. The result isn't particularly beautiful because I wanted to keep the code and markup to a minimum as not to obfuscate the core. If you are looking into making a nice looking interface with angular, I suggest you look at [Angular Material](https://material.angularjs.org/).

<p data-height="477" data-theme-id="0" data-slug-hash="EZgyXv" data-default-tab="js,result" data-user="dolfandringa" data-embed-version="2" data-pen-title="Drawing an image on a canvas with Angular" class="codepen">See the Pen <a href="https://codepen.io/dolfandringa/pen/EZgyXv/">Drawing an image on a canvas with Angular</a> by Dolf Andringa (<a href="http://codepen.io/dolfandringa">@dolfandringa</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

The code creates a custom [angular directive](https://docs.angularjs.org/guide/directive) (called stateGraph/state-graph) that can be set as an attribute on a canvas tag. When you do, angular kicks in and handles the canvas. The angular code creates an additional attribute for the canvas tag called "numbers" where you set the variable (an object in this case) that contains the data that should be drawn (and updated) in the canvas. The stateGraph directive uses an isolated scope for this. This means that the scope inside the stateGraph directive is not the same scope as the scope of the controller. Instead, the "numbers" attribute of the canvas tag is where the variable from the controller ($scope.ctrlnumbers) is assigned to the scope of the directive (scope.numbers). This way, the controller can change the $scope.ctrlnumbers on the scope, and those same numbers are watched by the directive scope.numbers, which updates the canvas when they change.

Ok, so this way I can create a function (now just a function that is triggered by a timeout and adds/multiplies numbers, but this can be changed to something that fetches values from a web-service) inside the controller that changes numbers, and the canvas directive is changed when those numbers change. The problem I still had (which is canvas related, not angular related) is how to *change* text in the canvas instead of just adding numbers on top of each other. The canvas object doesn't have any concept of variables. I is just a piece of digital paper on which you can draw and erase. And there lies the trick. To change text, you need to erase the old text and then add new text. This is what the drawNumber function does. It first clips the canvas to the circle where the text is. This ensure that everything that is done in the canvas from that point on, only affects the clipped area. I then do a clearRect() for clear the area where the text is. As the canvas is clipped, this will only erase whatever is in the clipped area, nothing outside. Then I restore the canvas (undo the clipping) and set the new text in the canvas.

The last thing I want to explain is how I fixed that my png image wasn't actually showing up on my canvas. I could see in the "Network" tab of the developer tools of my browser that the image was loading, it just didn't show up. The trick was to first create an onload function for the image object in which the image is drawn on the canvas, and only then set the src attribute. That way, the image starts loading as soon as you set the src attribute for it, and when it finishes, the onload method is called, which puts it on the canvas.

I hope this code and explanation helps someone out there with some mad google skills.
