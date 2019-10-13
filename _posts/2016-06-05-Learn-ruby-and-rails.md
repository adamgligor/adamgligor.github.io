---
layout: post
title: Ruby lang learning resources
date: '2016-06-05'
tags: programming
---

Curious about ruby ? Here are some resources

## Learning ruby - the language

Codewars is a gamified way of learning a programming language. You complete small code challenges and gain points and level up. It was just what I was looking for, it also had a lot of hits in google search so I signed up.

What I like about it is that after completing a challenge you are presented with the best solutions for that challenge, thus a nice way of learning from others.

What I like less is that coding happens in the browser and it's not the best place to experiment while building up a solution, that plus I wanted to store my code on github for future reference.

Fortunately others thought of this as well and came up with a solution to code from your editor of choice.

[Codewars client](https://github.com/shime/codewars) is a client for the api exposed by codewars, it's written in node.

These are the steps to install the tools:

 - install ruby [see here](https://gorails.com/setup/ubuntu/14.04)
 - install nodejs [see here](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server)
 - install codewars client [see here](https://github.com/shime/codewars)


and then the way of working is as follows ...

 - codewars train -l ruby  - start a new kata
 - codewars print > kata.txt - save the description for reference
 - codewars verify kata.rb - test the solution
 - codewars submit kata.rb - submit the solution
 - commit & push to github
 - check the codewars website and review the best solutions

## Learning Rails - the web framework

For learning the rails framework I started with the [railstutorial](https://www.railstutorial.org/book). It's a book designed around learning by doing. The nice part is that by using [cloud9](https://c9.io) the infrastructure and tooling comes out of the box and you can focus on learning rails. You will be deploying a functioning app to heroku starting from chapter 1.

## Links

- ruby reference [link](http://ruby-doc.org/core-2.3.1/)
- codewars [link](http://www.codewars.com/)
- codewars api client [link](https://github.com/shime/codewars)
- rails tutorial (by Michael Hartl) [link](https://www.railstutorial.org/book)
- cloud9 [link](https://c9.io)
- heroku [link](https://www.heroku.com)
