---
layout: post
title: Javascript array map and reduce functions
---

Map and reduce are functions defined on the Array class that are usefull for all kinds of processing.
In this post we'll provide some simple use cases. 

## Example 1

Return the length of the longest word in the provided sentence.

for input "May the force be with you"  return 5

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

Return Largest Numbers in Arrays.

for input [[13, 18, 26], [4, 5], [32,  37, 39]]  return [26,5,39]

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

for input '\*' and 8  return ********

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

Want more higher level functions? Check out the javascript array reference link below. 

## Links

examples in codepen.io [link] (http://codepen.io/adamgligor/pen/BjReyo)

javascript array reference [link] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

higher order functions wiki [link] (https://en.wikipedia.org/wiki/Higher-order_function)