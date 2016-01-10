---
layout: post
title: Map-reduce with javascript
---

TODO

## Example 1

Return the length of the longest word in the provided sentence.
Example: input "May the force be with you", output 5

```javascript
function findLongestWord (str) {
 var maxLen = str
  .split(" ")
  .map(function (currentValue) {
    return currentValue.length;
  })
  .reduce(function (previousValue, currentValue) {
    return Math.max(previousValue, currentValue);
  });
  return maxLen;
}
```

## Example 2

Return Largest Numbers in Arrays

```javascript
function findLargestNumbers(arr) {
 var result = arr.map(function (currentArr) {
   return currentArr.reduce(function (previousValue, currentValue) {
     return Math.max(previousValue, currentValue);
   });
 });
 return result;
}
```

## Example 3

Repeat a string

```javascript
function repeatString(str, num) {
  if (num <= 0) return "";
  var arr = Array(num);
  for (var i = 0; i < num; i++) arr[i] = 0;

  return arr.map(function (currentValue) {
    return str;
  }).reduce(function (prev, current) {
    return prev + current;
  });  
}
```

## Links

[examples in codepen.io] (http://codepen.io/adamgligor/pen/BjReyo)

[javascript array reference] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
