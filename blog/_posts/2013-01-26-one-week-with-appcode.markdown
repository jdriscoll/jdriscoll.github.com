---
layout: post
published: true
draft: true
title: One Week With AppCode
excerpt: On Monday I decided to try using AppCode for an entire day. Then one day turned into a week.
---

I managed to secure a license for [AppCode](http://www.jetbrains.com/objc/) during the recent one-day JetBrains sale that was plagued by server meltdowns and over-capacity payment gateways. I wasn't convinced I'd use it all that often but the price was low enough that I wagered it was worth it anyway. In fact, it wasn't until this last week, after installing the latest [AppCode EAP release](http://confluence.jetbrains.com/display/OBJC/AppCode+EAP), that I really gave it a chance. What started as a plan to use AppCode for "real" work, for just one whole day, turned into one week. 

## What's to Like?

Quite a bit actually. While AppCode can't replace Xcode entirely (this isn't a judgment of it's quality, AppCode requires Xcode to function) it's code generation tools, refactoring support and deep code inspections are impressive.

### Intentions

One of my favorite features is what JetBrains calls "Intentions". These are context sensitive (and language specific) actions that perform common tasks automatically. They can add missing imports, add definitions for existing methods in your interface, convert instance variables to properties and a hundred other things. All of this is pretty much as easy as typing option-return and sending AppCode off to do your bidding. I've found Intensions drastically reduce both the number of distractions and the amount of navigation I need to do while writing code. Instead of jumping up to the top of the file to add a property to the private interface I can just use the property and AppCode will see that it doesn't exist yet and offer to define it wherever I'd like. If I need to declare an existing method in the public interface it only takes two keystrokes and I don't have to leave my place the implementation file.

### Live Templates

While Intentions are powerful, they're not editable. Thankfully AppCode also provides "Live Templates", snippets basically, that you can create and customize. Live templates support contexts and placeholders and work well. Unfortunately, I've found myself using Live Templates more than I would normally as App Code doesn't seem to get along well with [TextExpander](http://smilesoftware.com/TextExpander/index.html). Sometimes it works fine, sometimes it seems to just inject the last thing you copied in App Code. While TextExpander snippets have no knowledge of code context they work everywhere and sync over dropbox.

__Quick Tip:__ I know this post is about AppCode but did you know that pasting "<#your_placeholder_name#>" will generate tab-able placeholders in Xcode? For example, you might have a TextExpander snippet like this for generating block typedefs (the syntax nearly always escapes me):

{% highlight objc %}

typedef <#void#> (^<#BlockTypeName#>)(<#args#>);  

{% endhighlight %}

(Thanks [Brian](http://twitter.com/bricooke))

### Refactoring

In addition to Intensions AppCode features much faster and more powerful refactoring support than Xcode. Simple rename operations are blazing fast. I've also found myself starting to write code inline that I would previously start a new method for because I can just "extract" it into a new method after the fact. Again, it saves typing, but it also eliminates a distraction and lets me focus more on the code. AppCode's refactoring support also makes it easy to convert all the magic numbers you might have laying around ("I'll fix that later...") to constants.

### Inspections

Like Xcode, AppCode constantly evaluates your project and warns you of problems in real time (I believe the latest EAP has support for Clang static analysis too) and can offer to fix many issue automatically. It also has great code reformatting support and can example your files, or your project, for unused imports and remove them. The import optimization is a little aggressive for my tastes (I often want to have an import in the source file even if it's in my pre-compiled header file) but it's never broken anything.

## There's Got to Be a Catch?

While there are a lot of things to like about AppCode it's not all sunshine and roses.

### Java!?

I imaging the first thing many developers get hung up on is the appearance of the app. It's not a native Cocoa app, and though I think the developers have done a pretty good job making it fit in on OS X, it's not going to win any design awards. The icon is not to my taste either. It's not Handbrake horrible but it's not great. It's Java, you just have to [deal with it](http://www.funnyordie.com/lists/62bea1fdd2/the-best-deal-with-it-gifs).

### Death by Customization

The documentation is another weak spot. There's virtually none. Digging into the massive number of preference dialogs can help but sometimes things are labeled in a way that makes no sense (to me) and there's no where to turn to for clarification. The preferences themselves are probably both a pro and a con. You can fiddle with and tweak a million things in the app but in my experience you end up _needing_ to fiddle with and tweak a million things before you get it even close to working the way _you_ want. Oh, and the preferences dialog is modal so at least at first you're probably going to get really tired of launching (and relaunching) the preference dialog. I really hope the preferences export option works well because if I ever had to go back in and start from scratch I might just give up and go back to Xcode.

### Speaking of Xcode...

As I mentioned earlier, AppCode _requires_ Xcode be installed. It needs Xcode to build and run the apps and for tools like Instruments and the simulator. It can't manage devices and provisioning profiles or any of the thousand other little things Xcode provides an interface for either. It also can't edit Core Data model files, nibs or storyboards. Double clicking a storyboard in App Code will open it in Xcode. Double clicking a Core Data model does nothing (I should file a bug for that). That said, AppCode does get along with Xcode pretty well. You can run them side by side, edit a storyboard in Interface Builder while you edit a class in AppCode. Any changes to the Xcode project file however, will cause AppCode to stop and reindex. You can't use the app while it's reindexing and it can take a minute even on a fast machine.

## Final Thoughts

I hope you were able to pick up AppCode while it was on sale. The normal price is not outrageous by any means but it was a steal at $25 and it's the kind of software you need to put some time into before the benefits really become apparent. I'd also recommend using the [latest EAP](http://confluence.jetbrains.com/display/OBJC/AppCode+EAP) (Early Access Program) release of version 2. It's _mostly_ stable and they've made a lot of improvements over the first version.

It's not perfect, in theory or execution, but it's earned a spot on my dock and I haven't written any _real_ code in Xcode since my little experiment started. In fact, as I write this, it's actually been two weeks since I started getting to know AppCode (I didn't have time to finish  this post last weekend) and I don't have any plans to go back to Xcode full-time.

