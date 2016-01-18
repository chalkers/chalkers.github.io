---
title: Hacking the Kinect with Processing
date: 28 Feb 2013
tags:
  - Reverse Engineering
  - Kinect
  - Processing
---
I bought an Xbox Kinect late last month as I wanted to use it to scan in my daughter to get printed out at [Shapeways](http://shapeways.com/) or a similar service for a present for my Gran's birthday.

I had seen software like [ReconstructMe](http://reconstructme.net/) and others like [Skanect](http://skanect.manctl.com/) on YouTube which have the ability to scan using a Kinect.

I then found myself wanting to experiment with the live data coming in from the Kinect. And I didn't really know where to start.

Since I've been playing around with [Arduino](http://forefront.io/a/beginners-guide-to-arduino) I thought I'd see if [Processing](http://processing.org/) had a way to interface with it. The Arduino programming environment editor is a derivative of the Processing editor. So I thought there would be some familiarity there.

I then Googled _processing with kinect_ and came across [this post](http://www.shiffman.net/p5/kinect/) by Daniel Shiffman which provided a library to interface Kinect with Processing.

<iframe src="http://player.vimeo.com/video/18058700" width="500" height="281" frameborder="0" webkitAllowFullScreen="webkitAllowFullScreen" mozallowfullscreen="mozallowfullscreen" allowFullScreen="allowFullScreen"></iframe> <p><a href="http://vimeo.com/18058700">Kinect Point Cloud demo in Processing</a> from <a href="http://vimeo.com/shiffman">shiffman</a> on <a href="http://vimeo.com">Vimeo</a>.</p>

I then opened up the point cloud example (shown above) and proceeded to hack for the evening, tweaking, reverse engineering Daniel's source code and learning keyboard inputs with Processing and after a little while I came up with this matrixy style hologram of myself with keyboard controlled rotation. (There's no audio.)

<iframe src="https://www.facebook.com/video/embed?video_id=10151459745267214" width="500" height="278" frameborder="0"></iframe>

If this has peeked your curiosity in to Processing check out the __Hello World! Processing__ documentary below:

<iframe src="http://player.vimeo.com/video/60735314" width="500" height="281" frameborder="0" webkitAllowFullScreen="webkitAllowFullScreen" mozallowfullscreen="mozallowfullscreen" allowFullScreen="allowFullScreen"></iframe> <p><a href="http://vimeo.com/60735314">Hello World! Processing</a> from <a href="http://vimeo.com/ultralab">Ultra_Lab</a> on <a href="http://vimeo.com">Vimeo</a>.</p>