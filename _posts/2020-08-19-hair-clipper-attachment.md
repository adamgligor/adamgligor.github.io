---
layout: post
title: DIY 3d printed hair clipper attachment
description: 
date: '2020-01-02'
tags: 3d-printing
---

DIY 3d printed hair clipper attachment.

## Description 

A hobby grade 3d printer can be useful around the house specially if you're into DIY stuff.

Saturday ... I needed a haircut, I could go to the hair dresser and be done with it, but I am lasy. I have hair clipper and I could cut it myself faster but it does not have a large enough attachment. I could order a propper hair cliper but then why make it easy. I have a 3d printer so i'm going to build the attachment myself. 


**The plan** 

![placeholder](/public/clipper-comb/cad.png "idea")

Some basic geometry and common sense, here's what I'm thinking. 

Red is the clipper plate, the pink point is where the cutting happens. I will draw a triangle touching that point and have its height be the cutting distance. Regardless how I will stick the clipper in my hair it won't get any closer than the distance shown by the pink line.


A web right angle triange calculator to calculate sides of the triangle for a 9mm height (cutting distance). Ex [here](https://www.calculator.net/right-triangle-calculator.html?av=&alphav=30&alphaunit=d&bv=&betav=&betaunit=d&cv=&hv=9&areav=&perimeterv=&x=57&y=22)

The rest is irrelevant I just draw another triangle on the back side. 


## Prototyping 

I use openscad for 3d modeling, that works fine for small things and it's programming. I also have a measuring caliper, the other tool you really need for 3d printing.

First step is build something that can attach to the clipper, just a u shape with two holes on the side.

![placeholder](/public/clipper-comb/openscad1.png "mount")

After I got the right size for the mount I just build on top of that, by adding triangles and rectangles according the the 2d sketch.

After a few more hours I get this. 

![placeholder](/public/clipper-comb/openscad2.png "mount")

and the code that draws that ...

```

module prism(l, w, h){
   polyhedron( 
    points=[[0,0,0], [l,0,0], [l,w,0], [0,w,0], [0,w,h], [l,w,h]],
    faces=[[0,1,2,3],[5,4,3,2],[0,4,5,1],[0,3,4],[5,2,1]]
   );
}

module spacer_9mm(wall){
    rear_length=30;
    rear_offset_z=2;
    translate([0,-wall,0]){
        //the front piece
        translate([18,0,-3])
            rotate([0,0,90])
                prism(wall,18,10.4);
        //the rear piece
        translate([-rear_length,wall,rear_offset_z])
            rotate([0,0,-90])
                prism(wall,rear_length,5.4);        
        //fill-in bit
        translate([-rear_length,0,0])
            cube(size=[rear_length,wall,rear_offset_z]);        
    }
}

module mount_plate_side() {
    height = 7;
    width = 10;
    wall = 2;
    offset_x = 30;
    
    translate([-offset_x,0,-height]){
        difference(){
            cube(size=[width,wall,height]);
            translate([3,0,2])
                cube(size=[4,wall,2]);
        }
    }
}

module mount_plate(clipper_width){
    center_width =20;
    wall = 2;
    offset_x = 30;

    //the center piece
    translate([-offset_x,-wall,0])
        cube(size=[center_width,clipper_width+2*wall,wall]);
    //one side piece
    translate([0, -wall, 0])
        mount_plate_side();
    //the other side piece
    translate([0, clipper_width,  0]) 
        mount_plate_side();    
}

module clipper_comb_9mm() {
    width = 36;
    // the complete mount plate
    mount_plate(width);
    // the two sides (thicker)
    spacer_9mm(2);
    translate([0, width + 2,  0])
        spacer_9mm(2);
    // 5 more sides (thinner)
    for (i=[1:5])
        translate([0, i*6.2, 0])
            spacer_9mm(1.2); 
}

clipper_comb_9mm();
```

On to 3d printing. Cura, slice, tweak and after 1 more hour I have my attachement.

![placeholder](/public/clipper-comb/clipper2.jpg "3d comb")

## Resources 

 - Openscad, 3d modeling [here](http://www.openscad.org/)
 - Cura, slicer [here](https://ultimaker.com/software/ultimaker-cura)
 - Librecad, 2s cad [here](https://librecad.org/)
