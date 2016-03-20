---
layout: post
title: Mastering Visual Studio keyboard shortcuts
date: '2016-03-19'
tags: .net
---

I've been using Resharper since it's early days, it's a great productivity booster but it also comes with a few annoyances. Major releases are often plagued by issues, it eats up a lot of resources and it requires a license. I decided to switch to using only the visual studio capabilities because visual studio 2015 now offers a much improved experience. I've also set myself a goal of becoming better with the keyboard and use mouse less. if you're happy with Resharper, I won't try to convince you to ditch it, you can skip the rest of the article. If you're interested what shortcuts I learned and use read on. In this guide I will focuses only on navigation and editing.  

## Level 0 - the basics

Probably one of the most used features in Resharper is the context actions menu (Alt+Enter usually). Visual studio has a similar function, the default shortcut to access it  **Ctrl+.**. Different options are available based on the cursor's location. The most common ones that I've discovered:

 - inside the using statements, removes the unnecessary using statements
 - on a method or constructor, remove or re-arrange parameters
 - on an unknown type (defined in another namespace), imports the namespace
 - on an undeclared variable, create a variable or property
 - on a class name, extract an interface

For example to create fields from the constructor parameters. Type the ctor signature, **Ctrl + Space** (autocomplete or suggestion) will help, then assign the ctor parameters to field names that are not yet defined, then use context action on the unknown symbol to create a field or property. The context action might even hold surprises occasionally, I've seen suggestion like "replace temp variable with inline call".


Next, to search for a file when you know a partial of full name, use the search of the solution explorer. The shortcut is **Ctrl + ;**. To have only files in the result there is a trick, on the right side of the search box access the dropdown "search options" and untick "search within file content and external items". Beware that when there is not a one to one match between class names and file names (which is a bad thing) don't turn these search options off. There is an alternative search window accessed with **Ctrl + ,**  this looks through everything, class names, variables, files. I find this less useful specially on big solutions as it returns more result than you'll want.  


Go to the definition of a class or method use **F12** on its name (this is Resharper's ctrl + click), this will open the class in a new tab.
Find usages of a method use **Shift + F12**, the results will be shown in a search window at the bottom with focus on the window. Use arrow keys to select an option or **Esc** to close the window and move focus back to the active tab.  There is also a "go to implementation" that you can call on interfaces, this does not have a key binding and it's only accessible from the right click context menu.


Tab pages are both good and bad. Having too many tab pages open will creates confusion, i'll go through a few ways of combating this issue in the next section, but the first thing to learn is that you have a navigation history and you can move back and forth between the places you visited. Navigate backward **Ctrl + -**, navigate forward **Ctrl + Shift + -**.
Scenario: while editing a file, you want to lookup a method definition found in another file, press F12 opens the method in a new tab, to return press Ctrl+-, and to go back to the method definition again Ctrl+Shift+-.   
Other basic commands related to tabs are, switching between tabs **Ctrl + Tab**, close current tab **Ctrl + F4**, and close all tabs but the current one for when you're totally lost. There is no default binding for this last one but one can be set, look for "File.CloseAllButThis" I use Ctrl+Alt+Shift+0  


Renaming things is done with **F2**, one thing to watch for here is that renaming a class applies only to the class name and not the file that contains it. To rename both the file and the class at the same time use rename file from solution explorer. Parameters info is obtained with **Ctrl + Shift + Space** after an open round brace. Use up down arrow keys to navigate between overloads. To search for plain text use the find window **Ctrl + F**


To improve selection and navigation within he current file. Use **Ctrl + Arrow Left/Right** to move the pointer one word at the time left or right (as opposed to one character at the time by using only the arro keys). To extend the selection one word at the time add a shift **Ctrl + Shift + Arrow Left/Right**.  Cut line with **Ctrl + X**, Copy/paste with **Ctrl + C** , **Ctrl + V**. These shortcuts works in most windows apps by the way.


Finally the peek definition window. It's an inline popup, the idea is the most often you only want to just quickly look at something so it's not worth opening a new tab. Use **Alt + F12** with the pointer on a class or method name to open the peek window. The focus moves in the peek window, you can use navigate with the arrow keys. When done use **Esc** to close it and return focus in the open tab.


## Level 1 - Intermediate

Bookmarks are an alternative way of navigating between locations of interest. The shortcuts are easy to remember. Bookmarks can successfully replace the need to use split panes or to drag tab pages out to a secondary monitor.

 - Clear all bookmarks **Ctrl+B, Ctrl+C**
 - Go to next bookmark **Ctrl+B, Ctrl+N**
 - Go to previous Bookmark **Ctrl+B, Ctrl+P**
 - Toggle bookmark on the current line **Ctrl+B, Ctrl+T**
 - Open bookmarks window **Ctrl+W, Ctrl+B**

Example. You have to edit one class and want to write a unit test, place a bookmark in the test class and one bookmark in the class file being developed, cycle back an forth with bookmark shortcuts. When you're done clear out the bookmarks.


Mastering the peek window. It won't take long to figure out that the peek window can do much more, you can have more peek windows open at the same time, cycle between them and the active tab, edit the file from the peek and navigate back and forth just like with tabs. Another advantage of using the peek window is that you'll have less tabs open. With only two or three tabs open you can quickly cycle through with ctrl+tab.  

 - Open Peek Definition **Alt+F12**
 - Navigate between multiple peek windows **Ctrl+Alt+-** and **Ctrl+Alt+=**
 - Toggle between editor window and the peek definition window **Shift+Esc-**
 - Close a peek window **Esc** (with focus on the peek window)
 - Promote the peek definition window to a regular document tab **Ctrl+Alt+Home**
 - Peek Forward **Ctrl+Alt+=**
 - Peek Backward **Ctrl+Alt+-**

Example. While editing a file you want to see the implementation details of the method A that's in another file. Open the peek window, look through, then say while in the peek window you want to see the details of a method B further down in another file. Alt+F12 moves the peek window to method B, then to go back to method A use peek backward Ctrl+Alt+-, finally to close the peek window press Esc. Peek forward and backward is similar to navigate forward and backward for tab pages.  


More useful shortcuts to help with navigation

 - expand all regions **Ctrl + M + P**
 - expand current region **Ctrl + M + E**
 - collapse current region **Ctrl + M + S**
 - navigate through IDE windows **Alt + F7**


More useful shortcuts to help with editing

- comment selected section **Ctrl + K + C**
- uncomment selected section **Ctrl + K + U**
- Move current line up or down **Alt + Arrow Up/Down**
- Select current word **Ctrl + Shift + W** (identical to double click)

## Level 2 - Wizard

First time you'll try these you might feel you brain is about to explode. Keep exercising and soon you'll be jumping back and forth code bits and files like a pro.

Clipboard ring. **Ctrl+Shift+V**, **Ctrl+Shift+Ins**. The clipboard retains more than just the last value, it's like a stack. You can move up and down the stack with the shortcuts and recall older clipboard data. One way to use this is when you need to copy bits from one file to another, just copy ctrl+c all the bits in the clipboard, then you can recall them when editing the other file just press ctrl+shift+v a couple of times to get the bits back.     

Refactor menu. You get good aid from visual studio even when performing more complex refactoring. Here's what I use more often.

 - Extract interface **Ctrl + R + I**
 - Extract method **Ctrl + R + M**
 - Encapsulate field **Ctrl + R + E**

Advanced features for navigation and editing

- Go to matching curly brace **Ctrl+]** , to extend selection use **Ctrl+Shift+]**. Use this on an open or closed curly brace to travel to the matching brace or select a method body.  
- Go to next error **Ctrl + Shift + F12**. Quickly cycle through build error
- Go to highlighted references  **Ctrl+Shift+Down Arrow**. Pause on a variable name, it will get highlighted then use the shortcut to visit all usages of that variable.
- select and edit vertically **Shift + Alt + Arrow keys**. This came as a surprise to me, you can select vertically not just horizontally.
