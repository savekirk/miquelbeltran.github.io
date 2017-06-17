---
title: "Refactor with the new ViewModel class"
layout: post
date: 2017-05-27 17:00
image: /assets/images/refactor-viewmodel.png
headerImage: false
tag:
- android
category: blog
author: miquel
description: Make an All-In-One Activity into a ViewModel powered one with the new Android Architecture Components
---

![Title](/assets/images/refactor-viewmodel.png)

I use Lara Martin’s android_book_listing project to try different architecture
patterns. I have a branch using MVP, a branch using MVVM and one branch with
MVP + Dagger.

So I thought it would be a good idea to do a new one using the
Architecture Components presented at Google I/O 2017.  

**This is a beginner friendly article.**

## Starting Code

Before we start, please take a look at the
original MainActivity.java. It is a very simple example in which all the calls
to Retrofit are made inside the Activity and the configuration changes are
handled manually.

Our goal with using ViewModel is:

- Use the new LiveData to store the response into the new ViewModel and survive configuration changes
thanks to the new LifecycleActivity.
- Remove the onSaveInstanceState method and
handling code.
- Although not mandatory, we will move all Retrofit code into the
ViewModel. A proper solution would be to move Retrofit into a Model class and
make this a full MVVM app.

## Configuration 

Add the new maven Google repository
to the build.gradle.

{% highlight groovy %}
allprojects { 
  repositories { 
    jcenter() 
    maven { 
      url 'https://maven.google.com'
    } 
  } 
} 
{% endhighlight %}

Then add the dependency to the Lifecycle

Architecture Components library, which contain the new ViewModel.

{% highlight groovy %}
compile "android.arch.lifecycle:runtime:1.0.0-alpha1"
compile "android.arch.lifecycle:extensions:1.0.0-alpha1"
annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha1"
{% endhighlight %}

## Creating a ViewModel class 

Our first step will be to create a ViewModel, in this case BooksViewModel

<script src="https://gist.github.com/miquelbeltran/51b3cb82769d99d94c53556dc4d0ff7d.js"></script>

The API response will be stored in the MutableLiveData
books object, we will update its contents by calling
books.setValue() with the API response content.

Now we
can use the new ViewModel on the MainActivity onCreate
method:

<script src="https://gist.github.com/miquelbeltran/51b3cb82769d99d94c53556dc4d0ff7d.js"></script>

The ViewModelProviders will create a new instance of
the BooksViewModel for us if required, and we just need
to listen (observe) for data changes in it.

As you can
see, you are not calling new BooksViewModel(); so if
you need to pass constructor parameters, check Danny
Preussler’s article Add the new ViewModel to your MVVM
where he explains how to use a Factory to create the
ViewModel instances.

Next, **change your Activity from
AppCompatActivity to LifecycleActivity**. This will
likely not be required in future releases of the
Architecture Components.

Now you are free to remove
the onSaveInstanceState method from the MainActivity.

## Finish the ViewModel 

Time to add the API calls to the
ViewModel.

Inside loadBooks we will perform the API
call that will update the MutableLiveData with the new
content.

<script src="https://gist.github.com/miquelbeltran/72cf1a9fad63d9f2fb844ef70a0e9797.js"></script>

Now from the MainActivity, call to
model.loadBooks(query) to update the ViewModel.

{% highlight java %}
private void performSearch() {
    String formatUserInput = getUserInput().trim().replaceAll("\\s+", "+");
    model.loadBooks(formatUserInput); 
}
{% endhighlight %}

Once you get the
response and update the MutableLiveData, you will get a
call on the onChanged method in the MainActivity.

The
data inside the BooksViewModel will survive the
Activity lifecycle, each time the Activity gets
recreated, the onChanged method will be called with the
previous data.

See the full example here in the
refactor/ViewModel branch:
[miquelbeltran/android_book_listing](https://github.com/miquelbeltran/android_book_listing/tree/refactor/ViewModel)

