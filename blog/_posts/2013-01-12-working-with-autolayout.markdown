---
layout: post
published: true
title: Working With Auto Layout on iOS
excerpt: After spending the last few months working on a new app using Cocoa Auto Layout for iOS I thought I'd share some things I learned along the way. 
---

After spending a large percentage of my time over the last few months building a new app using [Cocoa Auto Layout for iOS](http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/AutolayoutPG/Articles/Introduction.html) I thought I'd share some things I learned along the way. Auto Layout is pretty new, and I don't claim to be an expert, but I personally found the following discoveries useful.

## Pay Attention to Interface Builder (There's a Method to It's Madness)

Probably the greatest increase in my ability to use Auto Layout efficiently came from learning to predict, even poorly, what constraint changes Interface Builder would make when modifying a storyboard. The constraints Interface Builder generates are governed by rules and those rules can seem directly intended to spite you if you're not at all familiar with them. I've learned to view this as the software having "opinions" that while I may or may not agree with them, are valid in their own way. For example, align the bottom of one view to the bottom of  another's and Interface Builder will assume you want the bottom of those two views aligned come hell or high water. Even if you may _actually_ want them to resize independently when the layout is presented on different sized screens. By noticing these "opinions" you can quickly find the potentially offending constraints and fix them before they bite you in the ass.

## Promote Your Constraints

There are two types of constraints in Interface Builder, "User Constraints" and those added automatically by the software. User Constraints are shown in the sidebar with a blue icon and generated constraints are shown in purple (sorry color blind developers). It would appear (and would make sense) that when calculating the changes needed to keep a layout from becoming invalid while it's modified the software will attempt to preserve User Constraints over those it generated for you. Because of this I recommend "promoting" any generated constraints that you want to keep. This can be done by selecting the view the constraint is applied to, finding it under "Constraints" in the size inspector, clicking the gear menu on the constraint and selecting "Promote to User Constraint".

## Use the 'Apply Form Factor' Button in Interface Builder to Test Your Layouts

Even with a complex UI, Auto Layout can allow you to use a single storyboard for both the new 4 inch screen on the iPhone 5 as well as the older 3.5 inch screen size. A quick way to test your layouts during development is to toggle the form factor and see what breaks. You can do this either by selecting the "Apply Retina X Form Factor" menu item from the Editor menu or just clicking the [little button](http://cl.ly/image/2w1r2K42211w) in the bottom right. One word of warning though. Applying the new form factor will cause Interface Builder to modify your layout if it is no longer valid. For this reason I would recommend using undo, not re-applying the previous form factor, to revert the changes after noting what needs to be fixed.

## Remember 'translatesAutoresizingMaskIntoConstraints' For Views Created In Code

When adding a view to an existing hierarchy in code it's good to remember that by default, the runtime will translate the default autoresizing mask into a set of constraints. This my be fine if that's what you want or the view is never asked to resize in response to changes to it's superviews frame. Personally, this has been an issue enough times that I almost always set translatesAutoresizingMaskIntoConstraints to NO for views I initialize in code and replace the autoresize mask with an explicit set of constraints. Taking the time to learn the [ASCII-art formatting style](http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/AutolayoutPG/Articles/formatLanguage.html) helps _a lot_ with this.

## Use '_autoLayoutTrace' to Detect Ambiguous Layouts

Unfortunately there's no debugger for Auto Layout on iOS yet. If something is going wrong you can get a text dump of the current layout by pausing execution and entering the following into the debugger:

{% highlight objc %}

    po [[UIWindow keyWindow] _autolayoutTrace]

{% endhighlight %}

## Animate Changes Using 'layoutIfNeeded'

Switching to Auto Layout also means animating views by changing their layout constraints instead of their frame. This is aided by the fact that constraints can be saved as instance variables and even defined as outlets. Animating these changes is easy too. Just update the constraints and then call layoutIfNeeded on the superview within an animation block.

{% highlight objc %}

    self.myViewsHeightConstraint.constant = 42.0;
    
    [UIView animateWithDuration:0.3 animations:^{
        [self.view layoutIfNeeded];
    }]

{% endhighlight %}

## YMMV

Maybe something here will help someone else make the transition to using Auto Layout in their apps. If I've gotten anything totally wrong or you have a question (or your own tips to share) just let me know [on Twitter](http://www.twitter.com/jdriscoll).

