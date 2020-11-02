---
title: "Interpreting Index as Trend"
author: "Marek Fajkus & Zdenko Nevřala"
repository: "https://github.com/GlobalWebIndex/diverging-bar-chart"
created: "Tue Jan 10 2017 14:26:30 GMT+0100 (CET)"
show_footer: true
---

Please see this document in compiled [html version](http://globalwebindex.github.io/diverging-bar-chart).

# tl;dr

One of the most confusing things in both Chart Builder and Dashboards is readability of index metric.
From technical perspective there is nothing wrong with that number it self but it seems to be hard to interpret especially from charts we have.
Index suppose to be showing audience trends. In other words it shows diference between
particular audience and average internet users (or base audience when applied).
For instance if you create audience of **"XBOX ONE owners"** and you open question
**"Weekly hours spent playing video games"** with data-points like **"0 - 4 hours"** and **"more than 4 hours"** you should probably see
that your audience has lower index in **0 - 4 h** (probably lower than 100) and higher for **"more than 4 h"**.
This trend is expected but there can be patterns which are not so obvious and therefore index
brings important insight to audience behaviour.

This is (almost) how index is calculated:

```ruby
def index
    (audience_datapoint_percentage / audience_base_datapoint_percentage) * 100
end
```

# Understanding Index

This equation is linear. Let's assume our audience has **base data-point result equal 50%**
and result for **audience data-point percentage equal 75%**. When we pass these numbers to our function we will get this.

```ruby
(75/50) * 100 // => 150
```

So 150 is index of data-point in audience in this example.

**NOTE:** *audience base data-point percentage means "data-point percentage in set of users which are used as base for audience".
Understanding this concept is not necessary in order to understand index therefore we can say that base are all internet users for our purposes.*

We can port this calculation to java-script:

```javascript; auto
return window.index = function (audienceDatapointPerc, baseDatapointPerc) {
    return (audienceDatapointPerc / baseDatapointPerc) * 100;
}
```

Now we can check results for few data:

**TREND:** Audience trend match base trend (same percentage) - *is exactly average*

```javascript; auto
return index(50, 50);
```

**TREND:** Audience trend is half (* 1/2) base trend - *two times less than average*

```javascript; auto
return index(25, 50);
```

**TREND:** Audience trend is double (* 2/1) base trend. - *two times more than average*

```javascript; auto
return index(50, 25);
```

As you can see even though we used simple numbers trend itself is not so obvious from given results.
When trend is exact opposite (2 times less likely vs. 2 times more likely) it's hard to interpret this when looking ad index number.

We can think about index as about trend coefficient. In that sense index `100` is 1 since `1*n = n` (This is called identity function in Math).
Index `50` is `0.5` (half of base) and index `200` is `2` (two times more).

## Reading Index in Pro

It's really simple to find where this number come from once you understand what it means.
Let's have a brief look at how we can get those numbers directly from PRO platform.

![base](images/base.png)

![audience](images/audience.png)

# Showing Index as Trend

I think current charts do really terrible job in bringing this type of insight to index metric. This is due to linear nature of equation we are using.
Scale reflect respondents count (even though percentage is used) instead of showing ration.

In order to show trend itself `1/2` should be exactly opposite from `2/1`. This means that `50` and `200` should be represented
with bar of exactly same width but in opposite direction from index `100`.


## How it Looks

**I think we can use logarithmic scale for bar chart to show trend rather then count.**

**NOTE:** *Please be aware of fact that we are de-facto working with count even though we are using percentage.
100% - 50% = 50% where 25% - 50% = -25%. Even though trend is exact opposite numbers are not.*

### This is How We Can Translate Index Using Ln10

![ln10](images/ln10.png)

### Implementation

This is super simple implementation of diverging chart which renders 2 bars per data-point (in linear and logarithmic scale):

```javascript; auto
return window.bar = function (graphElement, audienceDatapointPerc, baseDatapointPerc) {

    // settings
    var width = 772;
    var height = 10;

    // calculate index
    var data = index(audienceDatapointPerc, baseDatapointPerc);

    // add SVG
    var $svg = d3.select(graphElement)
        .append('svg')
        .style('background', '#e0e0e0')
        .style('width', width)
        .style("height", height);

    var $chart = $svg.append('g')
        .attr('transform', 'translate(' + width/2 + ', 0)');

    // diverging bar chart with LINEAR scale
    var translate = data > 100 ? 0 : (100 - data) * (width/200);
    var barWidth = data > 100 ? ((data - 100) / 50) * (width/2) : translate;
    $chart.append('rect')
        .style('fill', '#f57474')
        .attr('transform', 'translate(' + -1 * translate + ', 0)')
        .attr('width', barWidth)
        .attr('height', height/2);

    // LOGARITHMIC SCALE

    // scale
    var scale = d3.scale.log();

    if (data <= 0) {
        return 'No match between audience and base';
    }

    // calculate
    var barWidth = Math.abs(scale(data/100)) * width/2;
    var translate = data > 100 ? 0 : -barWidth;

    // render rect
    $chart.append('rect')
        .style('fill', '#3fccff')
        .attr('transform', 'translate(' + translate + ', ' + height/2 + ')')
        .attr('width', barWidth)
        .attr('height', height/2);

   return 'index is: ' + data;
}
```

The important part is that we use logarithmic scale in case of blue bars: `var scale = d3.scale.log();`

## Results

As you can see bellow with this implementation trend visualization is quite obvious.

You can compare linear vs logarithmic scale yourself:

- Red bar is scaled linearly (according to design)
- Blue bar is scale logarithmically (suggested)

```javascript; auto
return bar(graphElement, 100, 50);
```

```javascript; auto
return bar(graphElement, 25, 50);
```

```javascript; auto
return bar(graphElement, 40, 60);
```

```javascript; auto
return bar(graphElement, 60, 90);
```

```javascript; auto
return bar(graphElement, 90, 60);
```

```javascript; auto
return bar(graphElement, 50, 50);
```

```javascript; auto
return bar(graphElement, 1, 10);
```

```javascript; auto
return bar(graphElement, 10, 1);
```

```javascript; auto
return bar(graphElement, 1, 100);
```

```javascript; auto
return bar(graphElement, 100, 1);
```

```javascript; auto
return bar(graphElement, 0, 30);
```

### Real world

Examples above are easy to understand. However they doesn't reflect real world in most cases since audience
with behaviour 100 times different from base is not expected to see too often.
These examples show more how readability changes with real data:

```javascript; auto
return bar(graphElement, 55, 50);
```

```javascript; auto
return bar(graphElement, 50, 55);
```

```javascript; auto
return bar(graphElement, 10, 11);
```

```javascript; auto
return bar(graphElement, 11, 10);
```

As you can see bars are pretty small. This is because implementation above scales:

- from 0 to 150 in case of linear
- from 10 times less to 10 times more for logarithmic.

We don't expect to have results of this scale (10 times more than base) in real world much.
Anyway with logarithmic scale we can deal with it by changing scale per "orders of magnitude"
or perhaps calculating biggest difference from `1` to logarithm `index/100`.
Than data will be still easy to compare even with dynamic scale.
Anyway this is scope of this example since we are rendering only one bar each time and dynamic scale will make it imposible to compare
different results.

# Working with This Project

- Clone repository
- run `npm install`
- run `npm run build`
- that's it

Tested with node 6.9.4 & npm 3.10.10.
