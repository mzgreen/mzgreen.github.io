---
layout: post
title: How to hide/show Toolbar when list is scroling (part 1)
tags: [android, material design, ui, animations, Toolbar, FAB]
---

In this post we will see how to achieve an effect that you can observe in Google+ app - hiding Toolbar and FAB (and any other views) when list is scrolling down and showing it again when it’s scrolling up. This behavior is mentioned in [Material Design Checklist].

> "Where appropriate, upon scrolling down, the app bar can scroll off the screen, leaving more vertical space for content. Upon scrolling back up, the app bar should be shown again.”

This is how our goal should look:
![Working example screenshot](/images/1/demo_gif.gif "Working example screenshot")


We will be using `RecyclerView` for our list but it’s possible to implement it in any other scrolling container (with a little more work in some cases i.e. `ListView`).
There are two ways that come to my mind on how to achieve this:

1. With adding a padding to the list. 
2. With adding a header to the list.

I decided to implement only the second one because I saw multiple questions on how to add a header to `RecyclerView` and this is a good opportunity to cover this but I will also briefly descripe the first one.

##Let’s get started!

We will begin from creating our project and adding necesarry libraries:
{% highlight Groovy %}
dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  compile 'com.android.support:appcompat-v7:21.0.3'
  compile "com.android.support:recyclerview-v7:21.0.0"
  compile 'com.android.support:cardview-v7:21.0.3'
} 
{% endhighlight %}

Now we should define `styles.xml` so that our app will use Material Theme but without `ActionBar` (we will be using `Toolbar`):
{% highlight xml %}
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  <item name="colorPrimary">@color/color_primary</item>
  <item name="colorPrimaryDark">@color/color_primary_dark</item>
</style>
 {% endhighlight %}

The next thing is to create our `Activity` layout:
{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
  
  <android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
    
  <android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"/>
    
  <ImageButton
    android:id="@+id/fabButton"
    android:layout_width="56dp"
    android:layout_height="56dp"
    android:layout_gravity="bottom|right"
    android:layout_marginBottom="16dp"
    android:layout_marginRight="16dp"
    android:background="@drawable/fab_background"
    android:src="@drawable/ic_favorite_outline_white_24dp"
    android:contentDescription="@null"/>
    
</FrameLayout>
{% endhighlight %}

It’s a simple layout with `RecyclerView`, `Toolbar` and `ImageButton` which will be our FAB. We need to put them in a `FrameLayout` because `Toolbar` needs to be overlayed on `RecyclerView`. If we don’t do this, there will be an empty space visible above the list when we hide the `Toolbar`.

Let’s jump into the code of our `MainActivity`:
{% highlight Java %}
public class MainActivity extends ActionBarActivity {
  private Toolbar mToolbar;
  private ImageButton mFabButton;

@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);
  initToolbar();
  mFabButton = (ImageButton) findViewById(R.id.fabButton);
  initRecyclerView();
}

private void initToolbar() {
  mToolbar = (Toolbar) findViewById(R.id.toolbar);
  setSupportActionBar(mToolbar);
  setTitle(getString(R.string.app_name));
  mToolbar.setTitleTextColor(getResources().getColor(android.R.color.white));
}

private void initRecyclerView() {
  RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
  recyclerView.setLayoutManager(new LinearLayoutManager(this));
  RecyclerAdapter recyclerAdapter = new RecyclerAdapter(createItemList());
  recyclerView.setAdapter(recyclerAdapter);
}

}
{% endhighlight %}

As you can see it’s a relatively small class. It only implements onCreate which does the following things:

1. Initializing `Toolbar`
2. Getting a reference to our FAB
3. Initializing `RecyclerView`

Now we will create an adapter for our `RecyclerView`. But first, we have to add a layout for our list items:
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:card_view="http://schemas.android.com/apk/res-auto"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:layout_gravity="center"
  android:layout_margin="8dp"
  card_view:cardCornerRadius="4dp">
  
    <TextView
    android:id="@+id/itemTextView"
    android:layout_width="match_parent"
    android:layout_height="?attr/listPreferredItemHeight"
    android:gravity="center_vertical"
    android:padding="8dp"
    style="@style/Base.TextAppearance.AppCompat.Body2"/>
</android.support.v7.widget.CardView>
{% endhighlight %}

And corresponding `ViewHolder`:
{% highlight Java %}
public class RecyclerItemViewHolder extends RecyclerView.ViewHolder {
  private final TextView mItemTextView;

public RecyclerItemViewHolder(final View parent, TextView itemTextView) {
  super(parent);
  mItemTextView = itemTextView;
}

public static RecyclerItemViewHolder newInstance(View parent) {
  TextView itemTextView = (TextView) parent.findViewById(R.id.itemTextView);
  return new RecyclerItemViewHolder(parent, itemTextView);
}

public void setItemText(CharSequence text) {
  mItemTextView.setText(text);
}

}
{% endhighlight %}

Our list will be showing cards with only text so this is it - easy!

Now we can jump to the `RecyclerAdapter` code:
{% highlight Java %}
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
  private List<String> mItemList;

public RecyclerAdapter(List<String> itemList) {
  mItemList = itemList;
}

@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
  final View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.recycler_item, parent, false);
  return RecyclerItemViewHolder.newInstance(view);
}

@Override
public void onBindViewHolder(RecyclerView.ViewHolder viewHolder, int position) {
  RecyclerItemViewHolder holder = (RecyclerItemViewHolder) viewHolder;
  String itemText = mItemList.get(position);
  holder.setItemText(itemText);
}

@Override
public int getItemCount() {
  return mItemList == null ? 0 : mItemList.size();
}

}
{% endhighlight %}

It’s a basic `RecyclerView.Adapter` implementation. There is nothing special about it. If you want to find about more about `RecyclerView`, I recommend you reading Mark Allison’s great [series of posts] 

We got all the pieces in place so let’s run it!
![Clipped screenshot](/images/1/clipped.png "Clipped screenshot")

Oh, what is this?! The `Toolbar` hides our list items and as you have probably noticed it’s because we are using `FrameLayout` in our `activity_main.xml`. This is the moment when we have two options that I mentioned at te beginning.
First option will be to add a paddingTop to our `RecyclerVie`w and set it to `Toolbar` height. But we have to be careful because `RecyclerView` will clip it’s chilren to padding by default so we have to turn it off. Our layout would look like this:
{% highlight xml %}
<android.support.v7.widget.RecyclerView
  android:id="@+id/recyclerView"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:paddingTop="?attr/actionBarSize"
  android:clipToPadding="false"/>
{% endhighlight %}

And it would do the thing. But as I said, I wanted to show you another way - maybe a little more complicated which involves adding a header to the list.

##Adding a header to the `RecyclerView`:

First we need to modify our `Adapter` a little:
{% highlight Java %}
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
  //added view types
  private static final int TYPE_HEADER = 2;
  private static final int TYPE_ITEM = 1;
  
  private List<String> mItemList;
  
public RecyclerAdapter(List<String> itemList) {
  mItemList = itemList;
}

//modified creating viewholder, so it creates appropriate holder for a given viewType
@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
  Context context = parent.getContext();
  if (viewType == TYPE_ITEM) {
    final View view = LayoutInflater.from(context).inflate(R.layout.recycler_item, parent, false);
    return RecyclerItemViewHolder.newInstance(view);
  } else if (viewType == TYPE_HEADER) {
    final View view = LayoutInflater.from(context).inflate(R.layout.recycler_header, parent, false);
    return new RecyclerHeaderViewHolder(view);
  }
  throw new RuntimeException("There is no type that matches the type " + viewType + " + make sure your using types    correctly");
}

//modifed ViewHolder binding so it binds a correct View for the Adapter
@Override
public void onBindViewHolder(RecyclerView.ViewHolder viewHolder, int position) {
  if (!isPositionHeader(position)) {
    RecyclerItemViewHolder holder = (RecyclerItemViewHolder) viewHolder;
    String itemText = mItemList.get(position - 1); // we are taking header in to account so all of our items are correctly positioned
    holder.setItemText(itemText);
  }
}

//our old getItemCount()
public int getBasicItemCount() {
  return mItemList == null ? 0 : mItemList.size();
}

//our new getItemCount() that includes header View
@Override
public int getItemCount() {
  return getBasicItemCount() + 1; // header
}

//added a method that returns viewType for a given position
@Override
public int getItemViewType(int position) {
  if (isPositionHeader(position)) {
  return TYPE_HEADER;
  }
  return TYPE_ITEM;
}

//added a method to check if given position is a header
private boolean isPositionHeader(int position) {
  return position == 0;
}

}
{% endhighlight %}

Here is how it works:
1. We need to define types of items that the `Recycler` will display. `RecyclerView` is a very flexible component. Item types are used when you want to have different layout for some of your list items. And this is exactly what we want to do - our first item will be a header view, so it will be different from the rest of items.
2. We need to tell the `Recycler` how many item types we want it to display.
3. We need to modify `onBindViewHolder()` method to bind a normal item if it’s type is `TYPE_ITEM` and a header item if it’s type is `TYPE_HEADER`.
4. We need to modify `getItemCount()` - we return a number of items in our dataset +1 because we have also a header.

Now let’s create a layout and `ViewHolder` for the header view:
{% highlight xml%}
<View xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="?attr/actionBarSize"/>
{% endhighlight %}

The layout is very simple. Important thing to notice is it’s height to be equal to our `Toolbar` height. And it's `ViewHolder` is also pretty straightforward:
{% highlight Java%}
public class RecyclerHeaderViewHolder extends RecyclerView.ViewHolder {
  public RecyclerHeaderViewHolder(View itemView) {
  super(itemView);
}
}
{% endhighlight %}

Ok, it’s done so let’s run it!
![Fixed clipping screenshot](/images/1/clipping_fixed.png "Fixed clipping screenshot")

Much better, right?
So to sum up, we have added a header to our `RecyclerView` that has the same height as `Toolbar`, so now our `Toolbar` hides header view (which is an empty view) and all of our list items are perfectly visible.
And finally we can implement showing/hiding views when list is scrolling.

##Showing/hiding views when list is scrolling.

To achieve this we will create only one more class - `OnScrollListener` for `RecyclerView`.
{% highlight Java%}
public abstract class HidingScrollListener extends RecyclerView.OnScrollListener {
  private static final int HIDE_THRESHOLD = 20;
  private int scrolledDistance = 0;
  private boolean controlsVisible = true;
  
@Override
public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
  super.onScrolled(recyclerView, dx, dy);
 
  if (scrolledDistance > HIDE_THRESHOLD && controlsVisible) {
    onHide();
    controlsVisible = false;
    scrolledDistance = 0;
  } else if (scrolledDistance < -HIDE_THRESHOLD && !controlsVisible) {
    onShow();
    controlsVisible = true;
    scrolledDistance = 0;
  }
  
  if((controlsVisible && dy>0) || (!controlsVisible && dy<0)) {
    scrolledDistance += dy;
  }
}

public abstract void onHide();
public abstract void onShow();

}
{% endhighlight %}

As you can see there is only one method where all the magic happens - `onScrolled()` method.
It’s parameters - dx, dy are the amounts of horizontal and vertical scrolls. Actually they are deltas, so it’s the amount between two events, not total scroll amount.

Basically an algorithm works like this:

1. We calculate total scroll amount (sum of deltas) but only if views are hidden and we are scrolling up or if views are visible and we are scrolling down because these are the cases that we care about.
{% highlight Java %}
if((controlsVisible && dy>0) || (!controlsVisible && dy<0)) {
  scrolledDistance += dy;
}
{% endhighlight %}
2. Now if this total scroll amount exceeds some threshold (that you can adjust - the bigger it is, the more you have to scroll to show/hide views) we show/hide views depending on the direction (dy>0 means that we are scrolling down, dy<0 means that we are scrolling up).
{% highlight Java %}
if (scrolledDistance > HIDE_THRESHOLD && controlsVisible) {
  onHide();
  controlsVisible = false;
  scrolledDistance = 0;
} else if (scrolledDistance < -HIDE_THRESHOLD && !controlsVisible) {
  onShow();
  controlsVisible = true;
  scrolledDistance = 0;
}
{% endhighlight %}
3. We don’t actually show/hide views in our scroll listener class, instead we make it abstract and call show()/hide() methods, so the caller can implement them as he wants.

Now we need to add this listener to our `RecyclerView`:
{% highlight Java %}
private void initRecyclerView() {
  RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
  recyclerView.setLayoutManager(new LinearLayoutManager(this));
  RecyclerAdapter recyclerAdapter = new RecyclerAdapter(createItemList());
  recyclerView.setAdapter(recyclerAdapter);
  //setting up our OnScrollListener
  recyclerView.setOnScrollListener(new HidingScrollListener() {
    @Override
    public void onHide() {
      hideViews();
    }
    @Override
    public void onShow() {
      showViews();
    }
  });
}
{% endhighlight%}

And here are the methods where we animate our views:
{% highlight Java %}
private void hideViews() {
  mToolbar.animate().translationY(-mToolbar.getHeight()).setInterpolator(new AccelerateInterpolator(2)).start();
  FrameLayout.LayoutParams lp = (FrameLayout.LayoutParams) mFabButton.getLayoutParams();
  int fabBottomMargin = lp.bottomMargin;
  mFabButton.animate().translationY(mFabButton.getHeight()+fabBottomMargin).setInterpolator(new AccelerateInterpolator(2)).start();
}

private void showViews() {
  mToolbar.animate().translationY(0).setInterpolator(new DecelerateInterpolator(2)).start();
  mFabButton.animate().translationY(0).setInterpolator(new DecelerateInterpolator(2)).start();
}
{% endhighlight %}

We have to take margins into account when we are hiding views, otherwise fab would’t fully hide.

It’s time to run our app!
![Broken scrolling screenshot](/images/1/broken_gif.gif "Broken scrolling screenshot")

It looks almost good! Almost because there is a little bug - if you are at the top of the list and threshold is small, you can hide the `Toolbar` and have empty space at the top of the list visible. Fortunately there is an easy fix for this. All we need to do is to detect if the first item of the list is visible and trigger our show/hide logic only if it’s not.
{% highlight Java %}
@Override
public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
  super.onScrolled(recyclerView, dx, dy);
  //find first item
  int firstVisibleItem = ((LinearLayoutManager) recyclerView.getLayoutManager()).findFirstVisibleItemPosition();
  //show views if first item is first visible position and views are hidden
  if (firstVisibleItem == 0) {
    if(!controlsVisible) {
      onShow();
      controlsVisible = true;
    }
  } else {
    if (scrolledDistance > HIDE_THRESHOLD && controlsVisible) {
      onHide();
      controlsVisible = false;
      scrolledDistance = 0;
    } else if (scrolledDistance < -HIDE_THRESHOLD && !controlsVisible) {
      onShow();
      controlsVisible = true;
      scrolledDistance = 0;
    }
  }
  
  if((controlsVisible && dy>0) || (!controlsVisible && dy<0)) {
    scrolledDistance += dy;
  }
}
{% endhighlight %}

After this change if the first item is visible and views are hidder, we are showing them, otherwise it works as before. Let’s run our project again and see if it helped.
![Working example screenshot](/images/1/demo_gif.gif "Working example screenshot")

Yup! It seems like everything is working like a charm now :)

It was the first blog post in my life so forgive me if it was boring or if I have made some mistakes. I will improve in the future.

Oh and if you don’t want to use this method, you can still use the second one with adding padding to the `RecyclerView`. There will be a need to change the logic in our scroll listener a little bit (only the part with detecting if first list item is visible) but it’s so simple that I will leave it to you as a homework ;)

In the next part I will show you how to make it to behave like scrolling in Google Play Store app.

If you have any questions feel free to ask them in the comments below.

###Code
Source code of the full project described in this post is available on GitHub [repo].

 *- Michał Z.*

[Material Design Checklist]:http://android-developers.blogspot.com/2014/10/material-design-on-android-checklist.html
[series of posts]:https://blog.stylingandroid.com/material-part-4/
[repo]:https://github.com/mzgreen/HideOnScrollExample
