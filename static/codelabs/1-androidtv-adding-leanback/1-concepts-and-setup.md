<toc-element></toc-element>

### The goal

In this codelab, you'll learn how to quickly enable a mobile app for AndroidTv using the Leanback library.  At the end of the codelab you can expect to have a UX compliant single apk for mobile devices and AndroidTv.

### Concepts

To start off lets learn a little bit about AndroidTv.  What is AndroidTv and how is it different?  At it's core, it is Android so most of the things that you've learned developing your mobile app can be reused.  The key difference is input and the presentation of information.

<figure layout vertical center>
  <img src="img/tv.png" alt="androidtv" class="noborder">
</figure>

AndroidTv is designed for the 10 foot experience.  Instead of a touchscreen, users will be navigating using a controller.  Instead of swiping the notification bar down, the notifications will be displayed as the top row of cards.  And the screen is always filled with rich visual content.

In an effort to simplify integration for developers we created the Leanback library.  Leanback has fragments allow you to quickly and easily create rich animated experiences.  The core fragments we'll be working with are:
* BrowseFragment - Browse through a video library
* DetailsFragment - Display the details of a specific video
* PlaybackOverlayFragment - Control video playback

These fragments use the Model View Presenter pattern.  You'll bind your data model to the view using Presenter classes.

There's a lot of ground to cover, so let's get started!

### Clone the starter project repo
<aside class="callout">
This codelab uses **Android Studio**, an IDE for developing Android apps.

<div class="extended">If you don't have it installed yet, please
[download](https://developer.android.com/sdk/installing/studio.html) and install it.</div>
</aside>

The first thing we need to do is get the mobile app to build on.

    git clone https://github.com/pengying/androidtv-codelab-app.git

In the project directory open up `build.gradle` with Android Studio.

### Understanding the starter project
All right, `checkpoint_0` is the base app that we'll be building upon. <img src="img/checkpoint_0.png">

Each of the following checkpoints can be used as reference points to check your work or for help if you encounter any issues.

A brief overview of each of the components:

* MainActivity - Video browser
* PlayerActivity - Video player
* SlidingTabLayout, SlidingTabStrip, VideoItemFragment - UI for video browser
* data/

  Video - Object storing video info

* data/

  VideoContentProvider, AbstractVideoItemProvider, VideoItemContract, VideoDataManager - Mock local video database

### Running the starter project
Lets run it on a phone.

<div class="stepbystep">
<ul>
<li>Connect your Android device or start an emulator.</li>
<li>Select the `checkpoint_0` configuration and click run. <img src="img/checkpoint_0_run.png"></li>
<li>Select your Android device and click ok.</li>
</div>

<aside class="callout">
If you want to learn more about querying databases take a look at our [documentation](http://developer.android.com/training/basics/data-storage/databases.html).
</aside>

Here's what it should look like.

<figure layout vertical center>
  <img src="img/checkpoint_0_screenshot.png" alt="checkpoint_0 screenshot" width="300" class="noborder">
</figure>

Now lets see how it looks on AndroidTv.

### ADB connect to AndroidTv

First we need to connect to the AndroidTv device.  In order to that you can use a male to male USB cable or adb connect.  In this codelab we'll cover the adb connect method.

#### On the AndroidTv

Lets enable developer mode if it's not yet enabled.
<div class="stepbystep">
<ul>
<li>Open <strong>Setting</strong></li>
<li>Under preferences open <strong>Developer Options</strong></li>
<aside class="callout">
  If Developer Options doesn't display go into Device -> About, scroll down to **Build** and click **A** a few times until you receive the &quot;You're A Developer&quot; toast.  Then exit out of Settings and open Settings up again.
</aside>
<li>Click <strong>Debugging</strong>.</li>
<li>Change <strong>USB debugging</strong> to On</li>
</ul>
</div>

Now we need to find the ip of the device.
<div class="stepbystep">
<ul>
<li>Back out to the main Settings page and click <strong>Network</strong>.</li>
<li>Select the method that you've connected the device to your intranet.</li>
<li>If you're using Wi-Fi, select the network you're currently connected to.</li>
<li>Click on <strong>Status info</strong> and record your ip.</li>
<li>If you're using ethernet select the Internet connection and record your ip.</li>
</ul>
</div>

#### On your development machine

Once you have the ip of the device, you can connect to it using adb connect in a terminal.

    adb connect [ip address]:4321

Sweet we're now connected.

### Run the app on AndroidTv

Lets run the app on AndroidTv.  In Android studio select checkpoint_0 and click run again.  This time the AndroidTv device should apper in the list of running devices.  Select it and click ok.

And now the mobile app is running on AndroidTv.  It could use some TV UI love right?  In the next few steps we'll cover adding some AndroidTv UI to the existing data sources and video player.

<figure layout vertical center>
  <img src="img/checkpoint_0_androidtv_screencap.png" alt="androidtv screencap" width="600px">
</figure>


### Summary

In this step you've learned about:

- AndroidTv and it's concepts
- The codelab checkpoint_0 base
- How to connect to AndroidTv devices and deploy apks

### Next up

Lets start creating the video browsing experience.
