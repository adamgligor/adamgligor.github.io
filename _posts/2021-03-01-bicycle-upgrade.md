---
layout: post
title: Bicycle upgrade, riverside 120
description: 
date: '2021-03-01'
tags: diy
---

Upgrading a bicycle ...

## The bike

A b-twin riverside 120, a cheap 8 speed bicycle from decathlon. I wanted to upgrade it with a front shifter to have 3x8 speeds on it.

After I bit of research I figured that I'll go with as 24-34-42 chain ring instead of the original single 36 and I found that the manufacturer of most of the mech on the bicycle to be `microshift` .


The shopping list.
- Shifter, microshift ts-39 [here](https://www.microshift.com/models/ts39-8/)
- Front draileur, microshift fd-m20 [here](https://www.microshift.com/models/fd-m20/)
- Crank, shimano ty-501 [here](https://bike.shimano.com/en-EU/product/component/tourney/FC-TY501.html)


The shifter and deraileur can be bought conveniently from decathlon, the parts are made by microshift just rebranded as btwin; the crank is the cheapest that shimano sells.
The shifter needs to be bottom swing and have clamp mount, the front deraileur needs to match the chain ring and be compatible with shimano 3x8 shifters, the crank needs to have square hole mount.

## The challenge 

Now onto the challenging part. In order to install the front mech the frame needs to have the appropriate cable mounts, and while most frames will have everything needed this one does not. The cable is supposed be routed below to bottom bracket same way as the rear shifter but then the cable stopper is missing. This is a part that's welded on the frame and instead of having two, this frame has only one. 


I decided to make the mount with the 3d printer. First I tried a few crew-less designs but that didn't work well.
Eventually I gave up and just drilled holes in the frame and used screws, this should provide the best mechanical support.


I managed to put everything back together in a weekend, then spent a few more days trying out various mounts for the cable stopper, but eventually everything worked out ! I spent about 50 euro on all the parts.


*Pic1.* Parts and the custom mount.

![placeholder](/public/2021/03/2021-03-01-bike1.jpg "bike1")


*Listing 1* Source code for the mount in openscad.

```
module arc(radius, angles, width = 1, fn = 24) {
    // todo ...
} 

module support() {
    // the piece that bolts on the frame tube

    DIA = 41; // the bicycle frame tube
    THK = 3; // the thickness of the piece 
    LEN = 34; // 37 the length of the piece
    WDT = 10; 
    
    difference() {  
        //the plate
        intersection() {
            //the arc
            translate([0,0,-DIA/2]) rotate([90,0,90]) 
            linear_extrude(LEN) arc(DIA/2, [75, 105], THK, fn=100);
            
            //the rounding
            translate([0,-WDT/2,-THK]) linear_extrude(height=THK*2) 
                offset(r=2,$fn=20) offset(r=-2,$fn=20) square([LEN, WDT]);
            
        }
        //the crew holes    
        R2 = 3.1; 
        R1 = 1.1;
        H = 3.1 ;
        OFS = 6; //offset of hole from the edge
    
        union(){
            //color("red") {
            translate([OFS,0, -0.1]) 
            cylinder(h=H,r1=R1,r2=R2, $fn=20);

            translate([LEN-OFS,0, -0.1])
            cylinder(h=H,r1=R1,r2=R2, $fn=20);
        }
    }
}


module stopper() {
    // the cable stopper

    LEN = 12+1; // mm length
    D1 = 5; // mm inner ring diam
    D2 = 10; // mm outer ring diam
    CCT = 1.5; // mm bowden cutout
    
    translate([0,0,0]) rotate([90,0,90])
    difference(){
    //color("red"){
        union() {
            //the outer ring
            cylinder(d=D2, h=LEN,$fn=50);
            // the lower stright part
            translate([-D2/2,-D2/2,0]) cube([D2,D2/2, LEN]);
        }
        // the inner ring 
        translate([0,0,3]) cylinder(d=D1, h=LEN, $fn=50);
        //the bowden cutout
        translate([-CCT/2,0,0]) linear_extrude(height=LEN) 
        offset(0.5) offset(-0.5) square([CCT,LEN]);
    }    
}


support();
translate([(34-13)/2,0,6])
stopper(); 

```

## Links

- The bicycle, still sold by decathlon [here](https://www.decathlon.fr/p/velo-tout-chemin-riverside-120/_/R-p-300806)