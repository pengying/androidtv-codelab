<toc-element></toc-element>

### The Goal

In this codelab, you'll learn how to quickly enable a mobile app for AndroidTv using the Leanback library.  At the end of the codelab you can expect to have a UX compliant single apk for mobile devices and AndroidTv.

### Concepts

To start off lets learn a little bit about AndroidTv.  What is AndroidTv and how is it different?  At it's core, it is Android so most of the things that you've learned developing your mobile app can be reused.  The key difference is input and the presentation of information.

<figure layout vertical center>
  <img src="img/tv.png" alt="androidtv">
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
All right, checkpoint_0 is the base app that we'll be building upon. <img src="img/checkpoint_0.png">

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
<li>Select the checkpoint_0 configuration and click run. <img src="img/checkpoint_0_run.png"></li>
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

<div class="stepbystep">
<ul>
<li>Open Settings
<!-- Chrome Dev Editor callout block -->
<aside class="callout">
This codelab uses **Chrome Dev Editor**, a Chrome app IDE.
<div class="kiosk">
  Run Chrome Dev Editor by clicking its icon at the bottom of your screen:
  <figure>
    <img src="/static/images/app-icons/chrome_dev_editor_screenshot.png">
  </figure>
</div>

<div class="extended">If you don't have it installed yet, please
[install it from Chrome Web Store](https://chrome.google.com/webstore/detail/spark/pnoffddplpippgcfjdhbmhkofpnaalpg).</div>
</aside>
<!-- End of Chrome Dev Editor callout block -->

<div class="stepbystep">
  <ul>
    <li>
      <a href="zips/PolymerApp.zip">Download the project source</a> and save it to your computer.
    </li>
    <li>
      Unzip the project file, there should be one `PolymerApp` directory.
    </li>
  </ul>
</div>

<div class="stepbystep">
  <ul>
    <li>
      In Chrome Dev Editor, click <img src="img/hamburger.png" class="icon"> and select `Open Folder...`
    </li>
  </ul>
  <div>
    <img src="img/s1-open-folder.png" alt="open folder" style="height:250px;">
  </div>
</div>

<div class="stepbystep">
  <ul>
    <li>
      Select the `PolymerApp` directory to load it into the editor.
    </li>
  </ul>
  <div>
    <img src="img/s1-open-folder2.png" alt="open folder" style="height:190px;">
  </div>
</div>


You should see the following structure in your editor's sidebar.

    PolymerApp/
      api/          <!-- a fake API for our app to consume -->
      components/   <!-- installed dependencies from Bower -->
      images/
      post-service/ <!-- a component used in the tutorial -->
      starter/      <!-- the starting point for your project! -->
      step-2/       <!-- checkpoint steps, for reference -->
      step-3/
      step-4/
      step-5/
      .bowerrc      <!-- bower configuration file -->
      .gitignore
      bower.json    <!-- bower metadata file. Used for managing dependencies -->

### Preview the app

&rarr;  Open `starter/index.html` and hit the <img src="img/runbutton.png" class="icon"> button in the top toolbar to run the app.

Chrome Dev Editor fires up a web server and navigates to the `index.html` page. This is great way to preview changes as you make them.

<figure>
  <img src="img/s1-first-run.png">
  <figcaption>Preview of index.html</figcaption>
</figure>

You won't see much, just a grey background, but it at least lets us know that our server is running and we're ready to start hacking!

### Summary

In this step, you learned how to:

- Load a project into Chrome Dev Editor
- Run Chrome Dev Editor's web server to preview the app

### Next up

At this point the app doesn't do much. Let's add some code!
