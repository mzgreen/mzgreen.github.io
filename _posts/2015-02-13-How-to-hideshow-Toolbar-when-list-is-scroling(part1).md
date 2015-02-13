---
layout: post
title: How to hide/show Toolbar when list is scroling (part 1)
tags: [android, material design, ui, animations, Toolbar, FAB]
---

In this post we will see how to achieve an effect that you can observe in Google+ app - hiding Toolbar and FAB (and any other views) when list is scrolling down and showing it again when it’s scrolling up. This behavior is mentioned in [Material Design Checklist].
> "Where appropriate, upon scrolling down, the app bar can scroll off the screen, leaving more vertical space for content. Upon scrolling back up, the app bar should be shown again.”

This is how our goal should look:
![Working example](/images/1/demo_gif.gif "Working example")


We will be using `RecyclerView` for our list but it’s possible to implement it in any other scrolling container (with a little more work in some cases i.e. `ListView`).
There are two ways that come to my mind on how to achieve this:
1. With adding a padding to the list. 
2. With adding a header to the list.
I decided to implement only the second one because I saw multiple questions on how to add a header to `RecyclerView` and this is a good opportunity to cover this but I will also briefly descripe the first one.

#Let’s get started!

We will begin from creating our project and adding necesarry libraries:
{% gist mzgreen/129542ec1164deca459b build.gradle %}

Now we should define `styles.xml` so that our app will use Material Theme but without `ActionBar` (we will be using `Toolbar`):
{% highlight xml %}
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  <item name="colorPrimary">@color/color_primary</item>
  <item name="colorPrimaryDark">@color/color_primary_dark</item>
</style>
 {% endhighlight %}

The next thing is to create our Activity layout:
[xml z main activity]

It’s a simple layout with RecyclerView, Toolbar and ImageButton which will be our FAB. We need to put them in a FrameLayout because Toolbar needs to be overlayed on RecyclerView. If we don’t do this, there will be an empty space visible above the list when we hide the Toolbar.

Let’s jump into the code of our MainActivity:
[main activity class]

As you can see it’s a relatively small class. It only implements onCreate which does the following things:
1. Initializing Toolbar (lines xx-xx)
2. Getting a reference to our FAB (xx)
3. Initializing RecyclerView (xx-xx)

Now we will create an adapter for our RecyclerView. But first, we have to add a layout for our list items:
[item xml]

Our list will be showing only text items so this is it - easy!

Now we can jump to the adapter code:
[recycler code]

It’s a basic RecyclerView.Adapter implementation. There is nothing special about it. If you want to find about more about RecyclerView, I recommend you reading Mark Allison’s great series of posts: https://blog.stylingandroid.com/material-part-4/

We got all the pieces in place so let’s run it!
[screen z zakrywaniem]

Oh, what is this?! The toolbar hides our list items and as you have probably noticed it’s because we are using FrameLayout in our main activity xml. This is the moment when we have two options that I mentioned at te beginning.
First option will be to add a paddingTop to our RecyclerView and set it to Toolbar height. But we have to be careful because RecyclerView will clip it’s chilren to padding by default so we have to turn it off. Our layout would look like this:
[recycler z paddingiem xml]

And it would do the thing. But as I said, I wanted to show you another way - maybe a little more complicated which involves adding a header to the list.

#Adding a header to the RecyclerView:

First we need to modify our adapter a little:
[adapter z headerem]

Here is how it works:
1. We need to define types of items that the Recycler will display. RecyclerView is a very flexible component. Item types are used when you want to have different layout for some of your list items. And this is exactly what we want to do - our first item will be a header view, so it will be different from the rest of items.
2. We need to tell the recycler how many item types we want it to display.
3. We need to modify bindViewHolder method to bind a normal item if it’s type is TYPE_ITEM and a header item if it’s type is TYPE_HEADER.
4. We need to modify getCount() - we return a count+1 because we have also a header.

Now let’s create a layout and ViewHolder for the header view:
[layout headera xml]

The layout is very simple. Important thing to notice is it’s height to be equal to our Toolbar height. And RecyclerView is also pretty straightforward:
[view holder]

Ok, it’s done so let’s run it!

[naprwiony screen]

Much better, right?
So to sum up, we have added a header to our RecyclerView that has the same height as Toolbar, so now our Toolbar hides header view (which is an empty view) and all of our list items are perfectly visible.
And finally we can implement showing/hiding views when list is scrolling.

#Showing/hiding views when list is scrolling.

To achieve this we will create only one more class - OnScrollListener for RecyclerView.
[on scroll listener source]

As you can see there is only one method where all the magic happens:
[onScroll source zepsuty jeszcze]

It’s parameters - dx, dy are the amounts of horizontal and vertical scrolls. Actually they are deltas, so it’s the amount between two events.
You can imagine this like that:
Let’s assume that you will walk along the street and you will measure the passed distance every 5 seconds and reset the counter. Let’s say that you started at position 0 and after 5 seconds you have passed 8 meters. So your delta is 8-0 = 8 meters. Now you continue walking from that position at 8 meter and after 5 seconds you have passed 10 meters. You started at 8 and now you are at 18 meter so your delta is 18-8 = 10 meters. I hope you now understand what delta is and for those who already knew I’m sorry for explaining such things :)

Let’s get back to our code. Basically an algorithm works like this:
1. We calculate total scroll amount (sum of deltas) but only if views are hidden and we are scrolling up or if views are visible and we are scrolling down because these are the cases that we care about.
[kod sumowania]
2. Now if this total scroll amount exceeds some threshold (that you can adjust - the bigger it is, the more you have to scroll to show/hide views) we show/hide views depending on the direction (dy>0 means that we are scrolling up, dy<0 means that we are scrolling down).
3. We don’t actualli show/hide views in our scroll listener class, instead we make it abstract and call show()/hide() methods, so the caller can implement them as he wants.

Now we need to add this listener to our RecyclerView:
[main activity z podpieciem listenera]

And here are the methods where we animate our views:
[show animation]

[hide animation]

We have to take margins into account, otherwise fab would’t fully hide.

It’s time to run our app!
[zepsute ukrywanie]

It looks almost good! Almost because there is a little bug - if you are at the top of the list and threshold is small, you can hide the toolbar and have empty space at the top of the list visible. Fortunately there is an easy fix for this. All we need to do is to detect if the first item of the list is visible and trigger our show/hide logic only if it’s not.
[fixed on scroll]

After this change if the first item is visible and views are hidder, we are showing them, otherwise it works as before. Let’s run our project again and see if it helped.
[running app]

Yup! It seems like everything is working like a charm now :)

It was the first blog post in my life so forgive me if it was boring or if I have made some mistakes. I will improve in the future.

Oh and if you don’t want to use this method, you can still use the second one with adding padding to the RecyclerView. There will be a need to change the logic in our scroll listener a little bit (only the part with detecting if first list item is visible) but it’s so simple that I will leave it to you as a homework ;)

In the next part I will show you how to make it to behave like scrolling in Google Play Store app.

If you have any questions feel free to ask them in the comments below.

##Code
Source code of the full project described in this post is available on GitHub repository(klik).

 *- Michał Z.*

[Material Design Checklist]:http://android-developers.blogspot.com/2014/10/material-design-on-android-checklist.html
