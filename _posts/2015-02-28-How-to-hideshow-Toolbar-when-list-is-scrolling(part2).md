---
layout: post
title: How to hide/show Toolbar when list is scrolling (part 2)
tags: [android, material design, ui, animations, Toolbar, Tabs]
---

This is a second (and the last) post of this series. If you haven't read [part 1] I recommend you to do it. In [previous part] we've learned how to achieve an effect of hiding Toolbar like Google+. Today we will see how we can make it to behave like Google Play Store Toolbar.  Let's begin!

Before we start, I would like to say that I've refactored this [project] a little bit - it's split in two `Activities`: `PartOneActivity` and `PartTwoActivity` that are started from `MainActivity`. Code is in the packages `partone` and `parttwo` so you can easily find classes that interest you.

This is how our final effect will look compared to Play Store Toolbar:
![Working example gif](/images/2/goal.gif "Working example gif") ![Play Store gif](/images/2/playstore.gif "Play Store gif")

## First things first
I won't show `build.gradle` file because it's the same as in [part 1] so we will start from creating a layout for our `Activity`:
{% gist /mzgreen/54faa3e44ae3de8a682c %}

It's just a `RecyclerView` and a `Toolbar` (we will add `Tabs` later). Notice that I'm using the second method (with adding padding to `RecyclerView`) described in [previous post].

I will omit a layout file of our list item because it's exactly the same as [before]. `RecyclerAdapter` was also described so we can skip it ([here] is how it should look - simple adapter, without header).

Let's jump to our `PartTwoActivity` code:
{% gist /mzgreen/72ef88c35fae6e979f15 %}

It's a basic initialization of `RecyclerView` and `Toolbar`, notice setting `OnScrollListener` in line 27.

The most interesting part is `HidingScrollListener`, so let's create it!
{% gist /mzgreen/95f93c615456c1d72104 %}

If you've read [previous part] it should look familiar (actually it's even simpler right now). What do we have here?
There is only one important variable so far - `mToolbarOffset` which holds a scrolled offset relative to `Toolbar's` height. To put it simply, we want to track only values between 0 and `Toolbar` height so thanks to this:
{% gist /mzgreen/5610dfaaf9756b6d456a %}
it will be increased if we scroll up (but we don't want it to be bigger than `Toolbar's` height) and decreased if we scroll down (again we don't want it to be lower than 0). You will see why do we restrict these values soon.
We are also clipping `mToolbarOffset` because it's possible that it will have some value out of our range for a small period of time (e.g. during fling) and it would cause flickering.
We've also defined  `onMoved()` - an abstract method which we call during scroll.
It may surprise you but this is it for now!

We have to get back to our `PartTwoActivity` and implement `onMoved()` method inside our scroll listener:
{% gist /mzgreen/e108819d31a9e675452b %}

Yup, that's all. We can run our app and see what do we have:
![No snap gif](/images/2/nosnap.gif "No snap gif")

Pretty good, the `Toolbar` is moving along with the list and getting back just as we expect it to. This is thanks to the restrictions that we put on the `mToolbarOffset` variable. If we would omit checking if it's bigger than 0 and lower than `mToolbarHeight` then when we would scroll up our list, the `Toolbar` would move along far away off the screen, so to show it back you would have to scroll the list down to 0. Right now it just scrolls up to `mToolbarHeight` position and not more so it's "sitting" right above the list all of the time and if we start scrolling down, we can see it immediately showing.

It works pretty well, but this is not what we want. It feels weird that you can stop it in the middle of the scroll and the `Toolbar` will stay half visible. Actually this is how it's done in Google Play Games app which I consider as a bug.

##Snapping the Toolbar
I think that views should smoothly snap to the position like Logo/SearchBar in Chrome app or `Toolbar` in Play Store app. I'm pretty sure that I saw it somewhere in Material guidelines/checklist or heard in one of Google I/O presentations.

Let's revisit out `HidingScrollListener` code:
{% gist /mzgreen/3e867d8db87aa23aa376 %}

It got a little bit more complicated but there is nothing scary in there. We've just overrided the second method of the `RecyclerView.OnScrollListener` class which is `onScrollStateChanged()`. This is what we're doing in this method:
- we are checking if the list is in `RecyclerView.SCROLL_STATE_IDLE` state so it's not scrolling nor flinging (because if it is, we'are translating Y position of the `Toolbar` manually - like before).
- if we lift up our finger and list has stopped (it's in `RecyclerView.SCROLL_STATE_IDLE` state) we have to check if it's visible and if it is, then this means that we have to hide it if `mToolbarOffset` is bigger than `HIDE_THRESHOLD` or we have to show it again if `mToolbarOffset` is lower than `SHOW_THRESHOLD`:
{% gist /mzgreen/a4f4fb043bc902e386c5 %}

and if it's not visible then we have to do the opposite - if `mToolbarOffset` (which now is calculated from top position so it's `mToolbarHeight - mToolbarOffset`) is bigger than `SHOW_THRESHOLD` then we are showing it and if it's lower than `HIDE_THRESHOLD` then we are hiding it again:
{% gist /mzgreen/fa7bb444c05315258dd3 %}

`onScrolled()` stays the same as it was, and we don't have to change anything else here. The last thing that we need to do is to implement our two new abstract methods in `PartTwoActivity` class:
{% gist /mzgreen/29359ccda3b5e1c68093 %}

It's time to build our project and see the effect:
![Snap no tabs gif](/images/2/snapnotabs.gif "Snap no tabs gif")

Looks like it's working! :)

You may think that adding the tabs will complicate the code, so let me show you that it's not the case.

##Adding Tabs

We need to modify our `Activity's` layout:
{% gist /mzgreen/0f8111db8b780efeb372 %}

and `tabs.xml`:
{% gist /mzgreen/d77ce91f21e9ceace291 %}

As you can see I'm not adding real `Tabs`, just a layout that mimics their look. It won't change anything in the implementation. You can put any view here. There are some implementations of the `Tabs` for Material Design available on github or you can create them yourself :)

Adding the `Tabs` means that they will cover our list a little, so we need to increase the padding. To make it  flexible we won't be setting this in xml (notice removed padding from `RecyclerView` in `part_two_activity.xml`), because `Toolbar` can have different height depending on orientation or device (e.g on tablets), so we would have to create a bunch of xml to cover all the cases. Instead we will set the padding in code:
{% gist /mzgreen/d3197b0faeb0f8103b71 %}
It's pretty simple, we're setting the padding to be the sum of `Toolbar's` height and `Tabs'` height.
If we run this now we will see something like this:
![Screen with tabs](/images/2/withtabs.png "Screen with tabs")

It's all good, our first list item is perfectly visible, so we can move along. Actually we won't change anything in our `HidingScrollListener` class. The only change that needs to be done is in `PartTwoActivity`:
{% gist /mzgreen/28e057f34b7e51bb1592 %}

Can you see what has changed? We are getting `mToolbarContainer` reference which is a `LinearLayout` instead of `Toolbar`, and in `onMove()`, `onHide()` and `onShow()` methods we are translating and animating this view instead of `Toolbar`. This will move a whole container that contains the `Toolbar` and `Tabs` and this is exactly what we need to do.

If we run it we can see that it seems to be working as expected, but if you look closely you will see that there is a little bug in there. Sometimes there is a white line visible between `Tabs` and `Toolbar` for the fraction of a second. It's probably because they are not perfectly synchronized when they are animating. Fortunately it's not something that we couldn't fix:)

The fix is very simple, just put the background of the `Toolbar` and `Tabs` to their parent layout:
{% gist /mzgreen/ce503aeba273c322abbb %}

Now even if views are not perfectly synchronized during animation, it won't be visible.
There is one more bug, the same that we had in [part 1]. If we are at the top of the list, we can scroll a little bit up and if the `HIDE_THRESHOLD` small enough, the `Toolbar` will hide and there will be an empty space(padding) visible above the list. Again - fix is really simple:
{% gist /mzgreen/09ef2f24440fbef5f17a %}

We've just added one more variable that holds total scroll offset of the list, and when we are about to check if we should show or hide the `Toolbar`, we first check if we scrolled more than `Toolbar's` height (if not, we show the `Toolbar` again).

This is it, let's run our app!
![Working example gif](/images/2/goal.gif "Working example gif")

It's working very well now :) And it even works with other LayoutManagers without changing anything else:
{% gist /mzgreen/892c394d6db47eb3708b %}

![Grid layout](/images/2/grid.png "Grid layout")

There was a question in the comment about saving scroll state, and it truly is a problem a little bit. If our item's text would be long enough to be 2 lines in portrait mode and 1 line in landscape mode then our items heights will be different in portrait and landscape. So if we would scroll to position let's say 100 in portrait and rotate the device with saving `totalScrollOffset` and after rotation we would scroll up the list to the top, then `totalScrollOffset` could be different than 0. There is no simple fix for that but for our usecase it doesn't matter. And if you really want to do something about it, I would reset `totalScrollOffset` to 0 and show the `Toolbar` after rotation.

So this is the end of this series of posts. I'm glad that some of you have learned something from the previous part. Thanks for the kind words :) I will continue writing this blog, but I don;t know what will be in the next posts yet:)

I also want to say that I think that methods described in this and previous(link) posts work pretty good but I didn't test them well enough, so I'm not sure if they are ready to use in production code (see for example the saving state problem described above). The goal of this series was to show that you can achieve effects that looks difficult using only simple methods and standard APIs. I've also found out that this method can be used for many other things (e.g. creating sticking tabs with parallax background - like in Google+ profile screen).
Happy coding!

###Code
Source code of the full project described in this post is available on GitHub [repo].

- Michał Z.
[part 1]: http://mzgreen.github.io/2015/02/15/How-to-hideshow-Toolbar-when-list-is-scroling%28part1%29/
[previous part]: http://mzgreen.github.io/2015/02/15/How-to-hideshow-Toolbar-when-list-is-scroling%28part1%29/
[project]: https://github.com/mzgreen/HideOnScrollExample
[before]: https://gist.github.com/mzgreen/b44afa284a0c3c4e0b9c#file-recycler_item-xml
[here]: https://gist.github.com/mzgreen/f0dc97062bb5f1c534b1#file-recycleradapter-java
[repo]:https://github.com/mzgreen/HideOnScrollExample


