# GraphScape

GraphScape([paper](http://idl.cs.washington.edu/papers/graphscape/)) is a directed graph model of the visualization design space that supports automated reasoning about visualization similarity and sequencing. It uses the [Vega-Lite](https://vega.github.io/vega-lite) language to model individual charts. This repository contains source code for building GraphScape models and automatically recommending sequences of charts.

- [Sequence Recommender API](#sequence-recommender-api)
- [Sequence Recommender Web Application](#sequence-recommender-web-application)
- [Development Instructions](#development-instructions)
- [Cite Us!](#cite-us)

## Sequence Recommender API

<a name="sequence" href="#sequence">#</a>
graphscape.<b>sequence</b>(<i>charts</i>, <i>options</i>[, <i>editOpSet</i>, <i>callback</i>])
[<>](https://github.com/uwdata/graphscape/blob/master/src/sequence/sequence.js "Source")

Generate recommended sequence orders for a collection of Vega-Lite *charts*. The return value is a ranked array of potential sequences and associated metadata.

### Input

| Parameter  | Type          | Description    |
| :-------- |:-------------:| :------------- |
| charts | Array | An array of [Vega-Lite](https://vega.github.io/vega-lite/) unit charts. |
| options | Object | `{ "fixFirst": true / false }` <br> *fixFirst*: indicates whether the first chart in *charts* should be pinned as the first chart of the recommended sequence (`true`) or not (`false`).|
| editOpSet | Object | (*Optional*) Specifies custom rules for calculating sequence costss |
| callback | Function | (*Optional*) `function(result) { ... }` <br> A callback function to invoke with the results. |


### Output

The output is a ranked array of objects, each containing a sequence ordering and related metadata.

| Property  | Type          | Description    |
| :-------- |:-------------:| :------------- |
| charts | Array | The given input charts. <br> If `options.fixFirst` was `false`, a *null specification* for an empty chart is included as the first entry. |
| sequence | Array | Order of indexes of input charts.   |
| transitions | Array | Transitions between each pair of two adjacent charts with `id`. |
| sequenceCost | Number| Final GraphScape sequence cost. |
| sumOfTransitionCosts | Number | Sum of transition costs. |
| patterns | Array | Observed patterns of the sequence. <br> Each pattern is consist of `pattern`, `appear`, `coverage`, and `patternScore`. <br> `pattern` : An array of transition `id`s composing the pattern.<br> `appear` : An array of indexes of `transitions` where the pattern appears in the sequence. <br>`coverage` : How much the pattern cover the sequence.<br>`patternScore` : Final pattern score, which is the same as coverage now. |
| globalWeightingTerm | Number | Global weighting term. |
| filterSequenceCost | Number | Filter sequence cost. |
| filterSequenceCostReasons | Array | Sum of filter value change score <br> Increment of value : +1 <br> Decrement of value : -1 <br> Otherwise : 0|


### Sample Code (node.js)

```js
var gs = require('./graphscape.js')
var charts = []; // an array of Vega-Lite charts
charts.push({
  "data": {"url": "data/cars.json"},
  "mark": "point",
  "encoding": {
    "x": {"field": "Horsepower","type": "quantitative"},
  }
});
charts.push({
  "data": {"url": "data/cars.json"},
  "mark": "point",
  "encoding": {
    "x": {"field": "Horsepower","type": "quantitative"},
    "y": {"field": "Miles_per_Gallon","type": "quantitative"}
  }
});
var options = { "fixFirst": false };
console.log(gs.sequence(charts, options));
```


<a name="transition" href="#transition">#</a>
graphscape.<b>transition</b>(<i>source chart</i>, <i>target chart</i>)
[<>](https://github.com/uwdata/graphscape/blob/master/src/transition/trans.js "Source")

Generate a transition from a source Vega-Lite chart to a target Vega-Lite chart. The transition has the minimum edit operation costs.

### Input

| Parameter  | Type          | Description    |
| :-------- |:-------------:| :------------- |
| source chart | Object | A [Vega-Lite](https://vega.github.io/vega-lite/) unit chart. |
| target chart | Object | A [Vega-Lite](https://vega.github.io/vega-lite/) unit chart. |

### Output

The output is a ranked array of objects, each containing a sequence ordering and related metadata.

| Property  | Type          | Description    |
| :-------- |:-------------:| :------------- |
| mark | Array | Edit operations in *mark* category. |
| transform | Array | Edit operations in *transform* category.   |
| encoding | Array | Edit operations in *encoding* category.   |
| cost | Number | Sum of all costs of edit operations in this transition.   |


### Sample Code (node.js)

```js
var gs = require('./graphscape.js')
var source = {
  "data": {"url": "data/cars.json"},
  "mark": "point",
  "encoding": {
    "x": {"field": "Horsepower","type": "quantitative"},
  }
};
var target = {
  "data": {"url": "data/cars.json"},
  "mark": "point",
  "encoding": {
    "x": {"field": "Horsepower","type": "quantitative"},
    "y": {"field": "Miles_per_Gallon","type": "quantitative"}
  }
};

console.log(gs.transition(source, target));
```

## Sequence Recommender Web Application

The `app/` folder contains a *sequence recommender* web application. Given a set of input [Vega-Lite](https://vega.github.io/vega-lite/) specifications, it produces a recommended sequence intended to improve chart reading and comprehension. To run this app, first you should install bower components:
```console
$ cd app
$ bower install
```
Next, launch a local webserver to run the application. For example:

```console
$ python -m SimpleHTTPServer 9000 # for Python 2
$ python -m http.server 9000 # for Python 3
```

To use a custom build of `graphscape.js`, copy your new `graphscape.js` file and paste it into the `app/js` folder.

## Development Instructions

1. MATLAB is required to solve `lp.m`.
2. Install npm dependencies via `npm install`.
3. You can customize rankings of edit operations by modifying `lp.js` and running the following commands:

```console
$ cd src/rule
$ node lp.js
$ matlab < lp.m
$ node genEditOpSet.js # This will generate editOpSet.js.

# After creating your rankings, you must re-build `graphscape.js` to apply changes.
$ cd
$ npm run test
$ npm run build
```

## Cite us!

If you use GraphScpae in published research, please cite [this paper](http://idl.cs.washington.edu/papers/graphscape/).
