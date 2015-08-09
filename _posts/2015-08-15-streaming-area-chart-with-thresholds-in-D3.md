---
layout: post
title: "D3: A Flowing Area Chart With Thresholds."
tags: "JavaScript, D3, D3.js, visualization, chart, area-chart, thresholds, code"
summary: "A flowing area chart using D3.js. It has two thresholds and is updated with random data every 500ms to simulate live streaming data. It adapts its width and height to the element it is applied to, but it isn't responsive (yet)."
---

<script src="http://blog.nicohaemhouts.com/D3-Transitioning-Area-Chart-With-Thresholds/d3.min.js"></script>  
<script src="http://blog.nicohaemhouts.com/D3-Transitioning-Area-Chart-With-Thresholds/transitioning-area-chart.js"></script>  
<style>    
    #myChart .minThreshold {
      fill: #ee5b33;
    }
    #myChart .area {
      fill: url(#area-gradient);
    }
    #myChart .thresholdLine {
      stroke: #ee5b33;
      stroke-width: 1px;
      stroke-linecap: round;
      stroke-dasharray: 1,7;
    }
    #myChart .yAxis {
      fill: none;
      stroke-width: 0;
    }
    #myChart .yAxis text {
      fill: #ee5b33;
      font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
    }
</style>
<div id="myChart"></div>
<script>
    (function () {
        d3.select('#myChart').call(transitioningAreaChart);
    }());
</script>

[Demo](http://blog.nicohaemhouts.com/D3-Transitioning-Area-Chart-With-Thresholds/) and [Github Repo](https://github.com/nicohaemhouts/D3-Transitioning-Area-Chart-With-Thresholds)

At work I'm building a system dashboard that visualizes metrics from a variety of systems. It polls different endpoints and
stores the metrics in a database as a way to keep a history of what the systems have been up to over time. 

The system is built using Node.js, Express, MongoDB, Angular.js and D3.js. I could have probably used something like PostgreSQL 
instead of MongoDB as I have in no way the amount of data that would make MongoDB shine. Luckily the data isn't all that relational so
I don't really need a relational database either. Also, opting for the MEAN stack I was able to hit the ground running by 
using the excellent [angular-fullstack](https://github.com/DaftMonk/generator-angular-fullstack) (Yeoman generator).

The frontend uses both ajax and websockets to retrieve and receive data. Initially when a dashboard is loaded in the webbrowser, an Angular controller
uses ajax to fetch the data of the last *n* hours. Once Angular has the data it's then bound to a visualization directivie implemented in D3. 
Form then on Angular starts listening to events on a websocket for new datapoints to be published by the backend. Every time a datapoint is received on the
socket the visualization is updated. 
  
It all works quiet nicely except for one little frustration. There's a transition in the D3 chart to make it appear as if the data is scrolling to the left.
In reality, a new datapoint is added to the right (out of view), which is then transitioned into view after which the left-most element is dropped from the 
data array. This transition is set to take the same amount of time as the polling interval used by the server. That way, I hoped, the transition would finish 
just as a new datapoint arrived and it could start all over. Thus creating the illusion of flowing data as in the chart above this post. 
Unfortunately this doesn't really work very well and the flowing illusion is often interrupted because the transition has finished before a new datapoint has arrived.
I've tried making the transition longer, but that didn't fix it. I also tried working with a buffer of datapoints, but after a couple of delays you run out of buffer 
and the flow is interrupted. 

Luckily the real-world polling intervals for this dashboard are pretty long and the problem occurs mostly with shorter intervals (less than four or five seconds). 

So, how does the chart above flow so nicely? Simple, it's all fake :) It doesn't listen to a websocket for data. It uses random data that is added to the data array
at the beginning of the transition and once the transition is finished it simply starts again by adding another datapoint and so on. 
 
{% highlight js %} 
var transition = d3.select({})
                   .transition()
                   .duration(500)
                   .ease("linear");

function update() {
    transition = transition.each(function () {
      
      data.push(random());
    
      path.attr("d", area)
          .attr("transform", null)
          .transition()
          .attr("transform", "translate(" + x(-1) + ")");
    
      data.shift();
    
    }).transition().each('start', update);
}

update(); 
{% endhighlight %}

A full explanation can be found on [http://bost.ocks.org/mike/path/](http://bost.ocks.org/mike/path/) Which is also where I got the idea from. 

