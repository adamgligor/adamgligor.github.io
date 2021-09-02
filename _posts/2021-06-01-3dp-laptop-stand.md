---
layout: post
title: 3d printed laptop stand
description: 
date: '2021-06-01'
tags: 3d-printing
---

A a customizable laptop stand for 3d printing.


## The stand 

![placeholder](/public/2021/06/2021-06-01-3dp-stand.png "stand")

Source code on thingiverse [here](https://www.thingiverse.com/thing:4827958)

## Things I learned 


1) padding using offset.

```
square([10,20]);
color("red") offset(r=-2) square([10,20]);
```

2) rounding by applying positive and negative offset

```
$fn=20;
offset(2) offset(-2) square([10,20]);
```

3) 90 degree rounding with boolean operators

```
intersection(){
  circle(r=10);
  square([10,10]);
}

difference(){
    square([10,10]);
    translate([10,10]) circle(r=10);
}

```

4) pyramids using intersection 

```
intersection()
{
linear_extrude(height=10)
    polygon([[0,0],[10,0], [8,10], [2,10]]);

translate([0,0,10]) rotate([0,90,0])
linear_extrude(height=10)
    polygon([[0,0],[10,0], [8,10], [2,10]]);
}
```

5) more 3d rounding ... 

```
$fn=20;
difference(){
  linear_extrude(height=10)
    offset(2) offset(-2) square([10,20]);
  linear_extrude(height=10)
   offset(1) offset(-2) square([10,20]);
}
```

![placeholder](/public/2021/06/2021-06-01-3dp-new.png "new")