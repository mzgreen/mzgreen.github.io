---
layout: post
title: How to hide/show Toolbar when list is scrolling (part 2)
tags: [android, material design, ui, animations, Toolbar, Tabs]
---

This is a second (and the last) post of this series. If you haven’t read [part 1] I recommend you to do it. In [previous part] we've learned how to achieve an effect of hiding Toolbar like Google+. Today we will see how we can make it to behave like Google Play Store Toolbar.  Let’s begin!

Before we start, I would like to say that I’ve refactored this [project] a little bit - it’s split in two `Activities`: `PartOneActivity` and `PartTwoActivity` that are started from `MainActivity`. Code is in the packages `partone` and `parttwo` so you can easily find classes that interest you.

This is how our final effect will look compared to Play Store Toolbar:
![Play Store gif](/images/2/playstore.gif "Play Store gif")
![Working example gif](/images/2/goal.gif "Working example gif")

## First things first
I won’t show `build.gradle` file because it’s the same as in [part 1] so we will start from creating a layout for our `Activity`:
[layout bez zakladek]

It’s just a `RecyclerView` and a `Toolbar` (we will add tabs later). Notice that I’m using the second method (with adding padding to `RecyclerView`) described in [previous post]:
[recycler z paddingiem]

I will omit a layout file of our list item because it’s exactly the same as [before]. `RecyclerAdapter` was also described so we can skip it ([here] is how it should look - simple adapter, without header).

Let’s jump to our `PartTwoActivity` code:
[kod activity]

It’s a basic initialization of `RecyclerView` and `Toolbar`, notice setting `OnScrollListener` in line XXX.

The most interesting part is `HidingScrollListener`, so let’s create it!
[listener]

If you’ve read [previous part] it should look familiar (actually it’s even simpler right now). What do we have here?
There is only one variable so far - `toolbarOffset` which holds a scrolled offset relative to `Toolbar’s` height. To put it simply, we want to track only values between 0 and `Toolbar` height so thanks to this:
[watrnek]
it will be increased if we scroll up (but we don’t want it to be bigger than `Toolbar’s` height) and decreased if we scroll down (again we don’t want it to be lower than 0). You will see why do we restrict these values soon.
We’ve also defined  `onMoved()` - an abstract method which we call during scroll.
It may surprise you but this is it for now!

We have to get back to our `PartTwoActivity` and implement `onMoved()` method inside our scroll listener:
[implementacja]

Yup, that’s all. We can run our app and see what do we have:
![No snap gif](/images/2/nosnap.gif "No snap gif")

Pretty good, the `Toolbar` is moving along with the list and getting back just as we expect it to. This is thanks to the restrictions that we put on the `toolbarOffset` variable. If we would omit checking if it’s bigger than 0 and lower than `toolbarHeight` then when we would scroll up our list, the `Toolbar` would move along far away off the screen, so to show it back you would have to scroll the list down to 0. Right now it just scrolls up to `toolbarHeight` position and not more so it’s „sitting” right above the list all of the time and if we start scrolling down, we can see it immediately showing.

It works pretty well, but this is not what we want. It feels weird that you can stop it in the middle of the scroll and the `Toolbar` will stay half visible. Actually this is how it’s done in Google Play Games app which I consider as a bug.

##Snapping the Toolbar
I think that views should smoothly snap to the position like Logo/SearchBar in Chrome app or `Toolbar` in Play Store app. I’m pretty sure that I saw it somewhere in Material guidelines/checklist or heard in one of Google I/O presentations.

Let’s revisit out `HidingScrollListener` code:
[scroll listener ze snapowaniem]

It got a little bit more complicated but there is nothing scary in there. We’ve just overrided the second method of the `RecyclerView.OnScrollListener` class which is `OnStateChanged`. This is what we’re doing in this method:
- we are checking if the list is in `RecyclerView.STATE_IDLE` state so it’s not scrolling nor flinging (because if it is, we’are translating Y position of the `Toolbar` manually - like before).
- if we lift up our finger and list has stopped (it’s in idle state) we have to check if it’s visible (this means that we have to hide it if `scrolledOffset` is bigger than `HIDE_THRESHOLD` or we have to show it again if `scrolledOffset` is lower than `SHOW_THRESHOLD`):
[code dla tego warunku]

and if it’s not visible then we have to do the opposite - if `scrolledOffset` is bigger than `SHOW_THRESHOLD` then we are showing it and if it’s lower than `HIDE_ThRESHOLD` then we are hiding it again:
[code dla warunku]

Corresponding show/hide methods:
[metody show i hide]

And two new abstract methods:
[show hide abstract methods]

`OnScroll` stays the same as it was, and we don’t have to change anything else here. The last thing that we need to do is to implement our two new abstract methods in `PartTwoActivity` class:
[implementacja metod] 
It’s time to build our project and see the effect:
![Snap no tabs gif](/images/2/snapnotabs.gif "Snap no tabs gif")

Looks like it’s working! :)

You may think that adding the tabs will complicate the code, so let me show you that it’s not the case.

##Adding Tabs

We need to modify our `Activity’s` layout:
[layout z tabami z bugiem]

and `tabs.xml`:
[layout tabow]

As you can see I’m not adding real `Tabs`, just a layout that mimics their look. It won’t change anything in the implementation. You can put any view here. There are some implementations of the `Tabs` for Material Design available on github or you can create them yourself :)

Adding the `Tabs` means that they will cover our list a little, so we need to increase the padding. To make it  flexible we won’t be setting this in xml, because `Toolbar` can have different height depending on orientation or device (e.g on tablets), so we would have to create a bunch of xml to cover all the cases. Instead we will set the padding in code:
[code ustawiania paddingu w aktywnosci]
It’s pretty simple, we’re setting the padding to be the sum of `Toolbar’s` height and `Tabs’` height.
If we run this now we will see something like this:
![Screen with tabs](/images/2/withtabs.png "Screen with tabs")

It’s all good, our first list item is perfectly visible, so we can move along. Actually we won’t change anything in our `HidingScrollListener` class. The only change that needs to be done is in `PartTwoActivity`:
[kod aktywnosci z animacja z zakladkami]

Can you see what has changed? We are getting `toolbarContainer` reference which is a `LinearLayout` instead of `Toolbar`, and in `onMove`, `onHide` and `onShow` methods we are translating and animating this view instead of `Toolbar`. This will move a whole container that contains the `Toolbar` and `Tabs` and this is exactly what we need to do.

If we run it we can see that it seems to be working as expected, but… if you look closely you will see that there is some bug in there. Sometimes there is a white line visible between `Tabs` and `Toolbar` for the fraction of a second. It’s probably because they are not perfectly synchronized when they are animating. Fortunately it’s not something that we couldn’t fix:)

The fix is very simple, just put the background of the `Toolbar` and `Tabs` to their parent layout:
[poprawiony layout z kolrami]

Now even if views are not perfectly synchronized during animation, it won’t be visible.
There is one more bug, the same that we had in [part 1]. If we are at the top of the list, we can scroll a little bit up and if the `HIDE_THRESHOLD` small enough, the `Toolbar` will hide and there will be an empty space(padding) visible above the list. Again - fix is really simple:
[code z naprawionym tym czyms]

We’ve just added one more variable that holds total scroll offset of the list, and when we are about to check if we should show or hide the `Toolbar`, we first check if we scrolled more than `Toolbar’s` height (if not, we show the `Toolbar` again):
[kod z warunkiem]

This is it, let’s run our app!
![Working example gif](/images/2/goal.gif "Working example gif")

It’s working very well now :) And it even works with other LayoutManagers without changing anything else:
[setting with GridLayoutManager]

![Grid layout](/images/2/grid.png "Grid layout")

There was a question in the comment about saving scroll state, and it truly is a problem a little bit. If our item’s text would be long enough to be 2 lines in portrait mode and 1 line in landscape mode then our items heights will be different in portrait and landscape. So if we would scroll to position let’s say 100 in portrait and rotate the device with saving `totalScrollOffset` and after rotation we would scroll up the list to the top, then `totalScrollOffset` could be different than 0. There is no simple fix for that but for our usecase it doesn’t matter. And if you really want to do something about it, I would reset `totalScrollOffset` to 0 and show the `Toolbar` after rotation.

So this is the end of this series of posts. I’m glad that some of you have learned something from the previous part. Thanks for the kind words :) I will continue writing this blog, but I don’t know what will be in the next posts yet:)

I also want to say that I think that methods described in this and previous(link) posts work pretty good but I didn’t test them well enough, so I’m not sure if they are ready to use in production code (see for example the saving state problem described above). The goal of this series was to show that you can achieve effects that looks difficult using only simple methods and standard APIs. I’ve also found out that this method can be used for many other things (e.g. creating sticking tabs with parallax background - like in Google+ profile screen).
Happy coding!

- Micha³ Z.
[part 1]: http://mzgreen.github.io/2015/02/15/How-to-hideshow-Toolbar-when-list-is-scroling%28part1%29/
[previous part]: http://mzgreen.github.io/2015/02/15/How-to-hideshow-Toolbar-when-list-is-scroling%28part1%29/
[project]: https://github.com/mzgreen/HideOnScrollExample
[before]: https://gist.github.com/mzgreen/b44afa284a0c3c4e0b9c#file-recycler_item-xml
[here]: https://gist.github.com/mzgreen/f0dc97062bb5f1c534b1#file-recycleradapter-java


