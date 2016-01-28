---
layout: post
title: Javascript simple pocket calculator
---

In this post i'll describe how to make a simple calculator using html css and javascript. 

## Requirements

The calculator should do the following: 

 * Add, subtract, multiply and divide two numbers
 * Clear the input
 * Chain multiple mathematical operations together

## Research 

A calculator is simple right? but it's good to start with a reasearch regardless of the urndertaking.

I wanted to compare a couple of calculators to find out if their behavior is identical or not. Pulled out my old genuine calculator, the calculator from my mobile phone, plus a couple of web calculators (plenty of those).
Straight away I noticed that each was doing its own thing, for example the mobile phone calc displayed the complete expression like 1 + 2 + 3  while the old calculator wiped out the screen after every operator. 

Next, finding the suitable algorithm. I first searched for information about pocket calculator architecture and algorithms, havent found much on that. 
Then tried to recalled algorithms I learned back in highscool for solving arthmetical operations, came up with reverse polish notation (quite proud I still remember that).
That would have worked but required a lot of work, so I came up with something nicer. 

Javascript has a built in expression evaluator, combine that with the ability to evaluate strings dynamically using the eval method and you can make the calculator without a sweat.

example. executing arithement expressions with eval()
 
```
Example eval("1 + 2 * 3"); //prints 7
```

## Implementation

### The caculator 

Let's start of with the javascript. I decided to put everything in one class so it can be tested and easily hooked up to a ui later.
It's modeled after the real thing. A method call simulates pressing a key, the screen redout is another method and a callback is also provided when the screen changes.
 
example. using the the calculation oject

```javascript 
var calc = new Calculator ()
calc.pressKey("3");
calc.pressKey("+");
calc.pressKey("1");
calc.pressKey("eval");
calc.value(); //prints 4
```

### Html & css 

Nothing complicated here. A button grid 4 by 5. The highlights are use of inline-block display to align the buttons on one row. 
One cavet with this is that the boxes don't stick to each other by default so you need to fix that with some tricks (i used font-size 0). 
Then box-sizing border-box to avoid zooming issues and use of data- attibute to store the keycodes of the buttons in the html

example. html for one row

```html
<div class="row">
    <div class="btn" data-key="7">7</div>
    <div class="btn" data-key="8">8</div>
    <div class="btn" data-key="9">9</div>
    <div class="btn  btn-highlight" data-key="*">x</div>
</div>
```

example. css for one row

```css
.btn {
    background-color:#dedede;
    color:#333;
    display: inline-block;
    border:1px solid #fff;
    padding:10px;
    width:80px;    
    height: 80px;
    line-height: 60px;    
    font-size: 25px;
    text-align: center;
    cursor: pointer;
}
```
 
## The result

Check out the final result on codepen and github.

[codepen link] (http://codepen.io/adamgligor/pen/rxJQoG)
[github] (https://github.com/adam-gligor)

 
## Bonus 

Unit testing with jasmine. To get started with jasmine simply download the archive from their website, edit the included SpecRunner.html file 
to include the paths to the tests and other resources. The tests get executed by opening the SpecRunner.html in the browser, and you'll get a nice visual report of the outcome.

example. sample jasmine tests
 
```javascript
describe("can add, subtract, multiply and divide two numbers.", function () {
    it(" add two numbers", function () {
        calc.pressKey("1");
        calc.pressKey("+");
        calc.pressKey("2");
        calc.pressKey("eval");
        expect(calc.value()).toBe("3");
    });
});
```

## Usefull links

[css tricks - border box] (https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/)
[css tricks - inline block] (https://css-tricks.com/fighting-the-space-between-inline-block-elements/)
[jasmine] (http://jasmine.github.io/)

