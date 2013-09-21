D3 for Dummies
==============

In this tutorial we will cover basic and advanced techniques of working with D3 library. Let's go!

## What we need

For each lesson there are two html files in relative folders. One of them is offered to you as a playground where you can learn new technique following the chapter. The other file contains the final product, which can be compared with what you have.

For each lesson open playground.html file, follow instructions and try to implement the code yourself. Solving problems after each lesson is recommended.

## Intro

You've probably met a lot of tutorials for D3. There was a big problem in ones I had a chance to observe. They were explaining how to build some sort of a plot with D3, but were not giving a systematic understanding of what was actually going on.

In 2005 Leland Wilkinson introduced the World an abstraction, making reasoning and communicating graphics easier. He described this concept in his book "The Grammar of Graphics" (according to its name).

In my opinion, following this abstraction allows to understand D3 much easier. We should definetly observe Grammar of Graphics in more details. We will follow it's modification called layered grammar of graphics, which is different from the original (presented by Wilkinson), but not that much.

### Grammar of Graphics

Consider the tech IPOs dataset, with information about companies valuations since 1980. This dataset is taken from New York Times right after Facebook public offering (http://www.nytimes.com/interactive/2012/05/17/business/dealbook/how-the-facebook-offering-compares.html).

The data allows to answer, which companies in Internet and telecom industries had highest valuations in history. It's easy to understand when we look at the following scatterplot.

![image](https://raw.github.com/mac-r/d3-for-dummies/master/intro_scatter_plot.png)

What is going underneath the surface? How did Michael Bostock build this plot with D3?

#### Mapping Aesthetics to Data

Mapping aesthetics to data is a fundamental part of working with D3. Let's think what is a precise way of illustrating data above? The scatterplot represents each company as a point, positioned according to its valuation and year of going public. As well as horizontal and vertical position, each point also has a size and a colour. We could build things in a different way, by using other geometric forms mapped to data.

In fact "aesthetics" is a definition I took from from ggplot, but there is nothing better, which could describe this.

#### Scaling

We understand that a company valuation is defined by the amount of money market is willing to pay for the companies equity. We may know even more about valuations, but D3 doesn't know anything about it until we communicate this to it. The best way to achieve our goal is to convert data units (years and valuations) to physical units that D3 can display.

This conversion process is called scaling and performed by scales.

(this part should be continued)


## Lesson 1 :: Static Bar Chart

By the end of this lesson you'll know how to build a plot like this:

![image](https://raw.github.com/mac-r/d3-for-dummies/master/lesson_1/lesson_1.png)

D3 is a JavaScript package, but we'll built our plots using CoffeeScript.

This will be our dataset:

```coffeescript
  dataset = [ {name: "Tom",   age: 60},
              {name: "Susan", age: 50},
              {name: "Fiona", age: 40},
              {name: "Mike",  age: 30},
              {name: "Merry", age: 20},
              {name: "Pat",   age: 10} ]
```

Plots in D3 are built layer by layer. First of all, we need to specify parameters of the canvas:

#### Canvas

```coffeescript
  margin  = { top: 60, right: 60, bottom: 60, left: 60 }
  width   = 640
  height  = 480
```

Then we create the canvas with the following code:

```coffeescript
  svg = d3.select("body") # specifying, where we append our plot
          .append("svg")  # svg object, which is a canvas
          .attr("style" , "background-color: yellow") # making canvas visible
          .attr("width" , width)
          .attr("height", height)
```

#### Scales

Scales are fundamental, they translate data into pixels. We build plots in D3 by mapping aesthetics to data (according to Grammar of Graphics). If you want to show something on the plot - don't forget about scales.

We build scales this way:

```coffeescript
  xScale  = d3.scale.ordinal() # used for descrete/categorical data (ex: histogram buckets)
              .domain(dataset.map((d)-> d.name)) # input values from the dataset
              .rangeRoundBands([margin.left, width - margin.right], 0.15) # output pixels
  yScale  = d3.scale.linear() # used for continuous data (ex: temperature, age, valuations)
              .domain([0, d3.max(dataset, (d) -> d.age)]) # input values from the dataset
              .range([height - margin.bottom, margin.top]) # output pixels
```

#### Axis

X and Y Axis are described like this:

```coffeescript
  xAxis = d3.svg.axis().scale(xScale).orient("bottom")
  yAxis = d3.svg.axis().scale(yScale).orient("left").ticks(3)
```

Then they are added to the plot:

```coffeescript
  svg.append("g")
     .attr("class", "x axis")
     .attr("transform", "translate(0,#{height-margin.bottom})")
     .attr("style", "font-family: sans-serif; font-size: 25px;")
     .call(xAxis)

  svg.append("g")
     .attr("class", "y axis")
     .attr("transform", "translate(#{margin.left},6)")
     .attr("style", "font-family: sans-serif; font-size: 25px;")
     .call(yAxis)
```

#### Bars

Now we are ready to build bars. To do this we use a chain of functions `data -> enter -> rect`. This allows to use magic function in each further `attr` (attribute): `(d) -> d.attribute`. The function allows to iterate through each entered object we want to create on the plot.

```coffeescript
  rect = svg.selectAll("rect")
     .data(dataset).enter().append("rect")
     .attr("fill", (d) -> "rgb(20,#{d.age*3}, 10)")
     .attr("x", (d) -> xScale(d.name))
     .attr("y", (d) -> yScale(d.age))
     .attr("height", (d) -> height - yScale(d.age) - margin.bottom)
     .attr("width", xScale.rangeBand())
```

#### Labels

Finally we add labels:

```coffeescript
  svg.selectAll("label")
     .data(dataset).enter()
     .append("text")
     .attr("class", "label")
     .text((d) -> d.age)
     .attr("style", "font-family: sans-serif; font-size: 17px; fill:white;")
     .attr("x", (d) -> xScale(d.name) + xScale.rangeBand()/2) # place text in the middle
     .attr("y", (d) -> yScale(d.age))
     .attr("dy", 30)
     .attr("text-anchor", "middle")
```

