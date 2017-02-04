+++
title = "jupyter notebook with interactive charts"
draft = false
date = "2017-01-30T11:06:01-06:00"
tags = ["python", "javascript", "jupyter"]
categories = ["post"]
readtime = "10"

+++

<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/modules/exporting.js"></script>
<script src="https://code.jquery.com/jquery-3.1.1.min.js"></script>

### The Problem

> I want beautiful, interactive charts in my Jupyter notebook.

Just yesterday I was playing with some data in a Jupyter notebook and making some charts with matplotlib when this thought came to me: "Wouldn't it be nice if I could generate interactieve charts in my notebook? Like those beautiful Javascript charts you see on every admin template.."

It is possible. Let me show you a whole new beautiful world!

### Highcharts

Highcharts is a Javascript library that makes attractive, interactive charts from minimal code. Example:

<div id="container" style="min-width: 310px; height: 400px; margin: 0 auto"></div>

<script>
$(function () {
    Highcharts.chart('container', {
        title: {
            text: 'Monthly Average Temperature',
            x: -20 //center
        },
        subtitle: {
            text: 'Source: WorldClimate.com',
            x: -20
        },
        xAxis: {
            categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
                'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
        },
        yAxis: {
            title: {
                text: 'Temperature (°C)'
            },
            plotLines: [{
                value: 0,
                width: 1,
                color: '#808080'
            }]
        },
        tooltip: {
            valueSuffix: '°C'
        },
        legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle',
            borderWidth: 0
        },
        series: [{
            name: 'Tokyo',
            data: [7.0, 6.9, 9.5, 14.5, 18.2, 21.5, 25.2, 26.5, 23.3, 18.3, 13.9, 9.6]
        }, {
            name: 'New York',
            data: [-0.2, 0.8, 5.7, 11.3, 17.0, 22.0, 24.8, 24.1, 20.1, 14.1, 8.6, 2.5]
        }, {
            name: 'Berlin',
            data: [-0.9, 0.6, 3.5, 8.4, 13.5, 17.0, 18.6, 17.9, 14.3, 9.0, 3.9, 1.0]
        }, {
            name: 'London',
            data: [3.9, 4.2, 5.7, 8.5, 11.9, 15.2, 17.0, 16.6, 14.2, 10.3, 6.6, 4.8]
        }]
    });
});
</script>

Isn't it great? Animations, tooltips on mouse hover, and exporting to image are just a few features of this great library. Its only dependency is jQuery which Jupyter loads by default. The chart above is Highchart's [Line Chart](http://jsfiddle.net/gh/get/jquery/3.1.1/highslide-software/highcharts.com/tree/master/samples/highcharts/demo/line-basic/) example which is the same chart I will be creating in my Jupyter notebook.

### Jupyter and JS

Jupyter uses [requirejs](http://requirejs.org/) to manage its Javascript libraries, which means we can "require" Highcharts into the notebook by adding the following code into a cell:

```javascript
%%javascript
require.config({
  paths: {
    highcharts: "http://code.highcharts.com/highcharts",
    highcharts_exports: "http://code.highcharts.com/modules/exporting",
  },
  shim: {
    highcharts: {
      exports: "Highcharts",
      deps: ["jquery"]
    },
    highcharts_exports: {
      exports: "Highcharts",
      deps: ["highcharts"]
    }
  }
});
```

Since the Highcharts library does not expose itself according to the requirejs standard, I have made it into a shim. The feature to export charts as images is loaded from a separate module in the Highcharts [source](http://code.highcharts.com/), so I had to load it as a separate shim and make the main library a dependency. 

### Python to Javascript

To translate the data from the Python realm into the Javascript realm felt a little "hacky", so bear with me and include the following code in a new cell:

```python
import json
from IPython.display import Javascript

chart_data = [
  {
    'name': 'Tokyo',
    'data': [7.0, 6.9, 9.5, 14.5, 18.2, 21.5, 25.2, 26.5, 23.3, 18.3, 13.9, 9.6]
  }, 
  {
    'name': 'New York',
    'data': [-0.2, 0.8, 5.7, 11.3, 17.0, 22.0, 24.8, 24.1, 20.1, 14.1, 8.6, 2.5]
  }, 
  {
    'name': 'Berlin',
    'data': [-0.9, 0.6, 3.5, 8.4, 13.5, 17.0, 18.6, 17.9, 14.3, 9.0, 3.9, 1.0]
  }, 
  {
    'name': 'London',
    'data': [3.9, 4.2, 5.7, 8.5, 11.9, 15.2, 17.0, 16.6, 14.2, 10.3, 6.6, 4.8]
  }
]

Javascript("window.chartData={};".format(json.dumps(chart_data)))
```

This takes the Python variable `chart_data` and binds it to Javascript's global `window` variable as `window.chartData`. This feels hacky -- manually loading Javascript and binding to a global -- but it works. If anyone has a prettier method please tweet me ([@codeweavr](http://twitter.com/codeweavr)) and I will edit this post with your recommendations.

# The Graph

To display the graph, I used the %%Javascript cell magic method and used the Highcharts demo code:

```javascript
%%javascript
// Since I append the div later, sometimes there are multiple divs.
$("#container").remove();

// Make the cdiv to contain the chart.
element.append('<div id="container" style="min-width: 310px; height: 400px; margin: 0 auto"></div>');

// Require highcarts and make the chart.
require(['highcharts_exports'], function(Highcharts) {
    $('#container').highcharts({
        title: {
            text: 'Monthly Average Temperature',
            x: -20 //center
        },
        subtitle: {
            text: 'Source: WorldClimate.com',
            x: -20
        },
        xAxis: {
            categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
                'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
        },
        yAxis: {
            title: {
                text: 'Temperature (°C)'
            },
            plotLines: [{
                value: 0,
                width: 1,
                color: '#808080'
            }]
        },
        tooltip: {
            valueSuffix: '°C'
        },
        legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle',
            borderWidth: 0
        },
        // This is where I used the chart_data from Python
        series: window.chartData
    });
});
```

## External References

You can see my Juypter notebook as a [gist](https://gist.github.com/jacobbridges/52b88708f9757ef799083528ebcf433b) and as a [rendered notebook](https://nbviewer.jupyter.org/urls/gist.githubusercontent.com/jacobbridges/52b88708f9757ef799083528ebcf433b/raw/2ba0d1405c204f3c9eb07664f65a9889d200c4c3/highcharts_example_with_jquery.ipynb). (The nbviewer project does not load all the libraries and css modules that Jupyter loads, so I had to include jquery as well in the require js cell. You may also notice the chart looks small or oddly shaped -- this won't happen on your local instance of jupyter.)

#### Articles

- [StackOverflow - Loading Highcharts with Requirejs](http://stackoverflow.com/questions/8186027/loading-highcharts-with-require-js)
- [The Data Incubator - Embedding d3.js in a iPython Notebook](http://blog.thedataincubator.com/2015/08/embedding-d3-in-an-ipython-notebook/)

#### Docs

- [Jupyter Docs](https://jupyter.readthedocs.io/en/latest/)
- [Highcharts Docs](http://www.highcharts.com/docs)
