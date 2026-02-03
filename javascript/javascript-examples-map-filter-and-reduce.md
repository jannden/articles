# JavaScript Examples: Map, Filter, and Reduce

*Originally published on Apr 13, 2023 [here](https://medium.com/@jannden/javascript-examples-map-filter-and-reduce-b89726e7ad36).*

---

These examples serve as a cheatsheet for Map, Filter, and Reduce.

PS: If you are wondering what's happening behind the scenes of these methods, you will find custom implementations in the second half of this article.

### Extract object properties with Map

```javascript
// What you have
var cities = [
  { name: "Vizima", king: "Foltest" },
  { name: "Cintra", king: "Calanthe" },
]

// What you need: array of kings, such as ["Foltest", "Calanthe"]
console.log(cities.map((city) => city.king));
```

### Filter object properties with Filter

```javascript
// What you have
var cities = [
  { name: "Vizima", king: "Foltest" },
  { name: "Cintra", king: "Calanthe" },
]

// What you need: array of cities where Foltest is the king
console.log(cities.filter((city) => city.king === "Foltest"));
```

### Reverse a string with Reduce

```javascript
// What you have
const text = "abcde"

// What you need: "edcba"
console.log(
  [...text].reduce(
    (reversedAccumulator, char) => char + reversedAccumulator,
    ""
  )
)
```

### Sort string characters with Sort

```javascript
// What you have
const text = "bdcea"

// What you need: "abcde"
// Simple solution:
console.log( [...text].sort().join("") )
// More precise:
console.log( [...text].sort((a, b) => a.localeCompare(b)).join("") )
```

### Sum object properties with Reduce

```javascript
// What you have
var cities = [
  { name: "Vizima", king: "Foltest", population: 6000 },
  { name: "Cintra", king: "Calanthe", population: 4000 },
]

// You need to know the total population of all cities in the array
console.log(cities.reduce((totalPopulation, city) => totalPopulation + city.population, 0));

// You need the city with the highest population
console.log(cities.reduce((highestPopulation, city) => highestPopulation > city.population ? highestPopulation : city.population, 0));
```

### Readable method chain

```javascript
// What you have
var cities = [
  { name: "Vizima", king: "Foltest", population: 6000 },
  { name: "Cintra", king: "Calanthe", population: 4000 },
  { name: "Brugge", king: "Foltest", population: 1000 },
]

// You want the total population of cities ruled by Foltest
const ruledByFoltest = (city) => city.king === "Foltest"
const getPopulation = (city) => city.population
const sumItems = (totalPopulation, item) => totalPopulation + item
console.log(
  cities
    .filter(ruledByFoltest)
    .map(getPopulation)
    .reduce(sumItems, 0)
)
```

## Custom Implementations

Re-creating these methods from scratch is the best way to truly understand what they do.

### Custom Map implementation

```javascript
function myMap(dataset, callback) {
  let cleanDataset = [];
  for (let element of dataset) {
    cleanDataset.push(callback(element));
  }
  return cleanDataset;
}
```

### Custom Filter implementation

```javascript
function myFilter(dataset, callback) {
  let cleanDataset = [];
  for (let element of dataset) {
    if(callback(element)) {
      cleanDataset.push(element);
    }
  }
  return cleanDataset;
}
```

### Custom Reduce implementation

```javascript
function myReduce(dataset, callback, accumulator) {
  for (let element of dataset) {
    accumulator = callback(accumulator, element)
  }
  return accumulator;
}
```

## Conclusion

This article showed examples and custom implementations of Map, Filter, and Reduce.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
