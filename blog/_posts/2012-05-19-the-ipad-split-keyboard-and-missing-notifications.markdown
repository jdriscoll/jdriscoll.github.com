---
layout: post
published: true
title: The iPad Split-Keyboard and (Missing) Notifications
excerpt: When the iPad keyboard is "un-docked" or "split" your app won't get the same UI events. If your app needs to accurately track the visibility of the keyboard on iPad you need to do a little more work.
---

Developing an app that people actually use is a great way to discover holes in your knowlege of new APIs. I recently released a side project of mine for sale on the App Store under the name [OnePAD](http://www.onepadapp.com). It's an electronic pocket notebook that syncs using your iCloud account. It's also a universal app, the first I've built, and even with a relatively simple app like this there are subtle differences in the two platforms that can catch you off-guard if you're not looking for them.

During the last round of beta testing, one of my testers reported that the interface wasn't updating properly on the iPad in response to keyboard events. This is not the kind of person who would make frivolous claims but even following his steps everything worked fine for me. Luckily, after I let him know of my inability to reproduce the issue, he was kind enough to send me a quick video of the bug occuring on his device and about 1 second into the video it was clear what the problem was and why I couldn't reproduce it.

*He was using the split keyboard.*

This new feature, added in iOS 5, allows you to either "undock" the full size keyboard from the bottom of the screen, or slide it up and "split it" into two halves to make thumb typing easier. It was also a feature that I never used, and to be honest, had forgotten even existed soon after the initial, "hey, that's kind of neat" reaction wore off.

It turns out that when the keyboard is "undocked", as they say, your app no longer receives the standard keyboardWill/DidHide/Show events. Because I was relying on these events to let me know when and how to configure the UI based on the presence of the keyboard, things were getting out of sync.

With the cause of the problem established, I had only to figure out how to replicate these events when using the undocked keyboard. Reviewing the [UIWindow class reference](http://developer.apple.com/library/ios/#DOCUMENTATION/UIKit/Reference/UIWindow_Class/UIWindowClassReference/UIWindowClassReference.html) It was clear that the undocked keyboard handled UI events a little differently. Instead of separate show/hide events it offered separate will/didChangeFrame events triggered by both user interaction and other application events. This was a start but there was no direct way to map these events to the show/hide events my application relied on.

The solution I settled on, and one that seems to work well, is to compare the destination frame from the keyboardDidChangeFrame event's notification to the view's frame and check for an intersection. If the keyboard is visible, it should intersect with the views frame right?

First we need to register for the event:

Test: {% post_url 2009-12-30-iphone-imap %}

{% highlight objc %}
[[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(keyboardDidChangeFrame:)
                                                 name:UIKeyboardDidChangeFrameNotification
                                               object:nil];
{% endhighlight %}

Then we just need to pull out the destination frame of the keyboard, convert it's coordinates to our own, and look for an intersection:

    - (void)keyboardDidChangeFrame:(NSNotification *)notification
    {
        CGRect keyboardEndFrame;
        [[notification.userInfo objectForKey:UIKeyboardFrameEndUserInfoKey] getValue:&keyboardEndFrame];
        CGRect keyboardFrame = [self.view convertRect:keyboardEndFrame fromView:nil];

        if (CGRectIntersectsRect(keyboardFrame, self.view.frame)) {
            // Keyboard is visible
        } else {
            // Keyboard is hidden
        }
    }


So if your app supports the iPad and relies on keyboard events, be sure to test it with the keyboard undocked. It's one of those problems that's far easier to fix than to discover.

And if you're in the market for a unique notebook/journaling app for iPhone, iPad and iPod Touch, I hope you'll take a look at [OnePAD](http://www.onepadapp.com).
