<toc-element></toc-element>

In this step, you'll learn how to use the classes provided by the leanback support library to implement a user interface for browsing videos from your app's media catalog.

In this step, you'll learn about:

-   Providing a home screen banner
-   Using Leanback theme
-   Declaring a TV Activity
-   Creating Fragment that extends BrowseFragment
-   Adding BrowseFragment to a layout
-   Creating LeanbackActivity

<div layout vertical center>
  <img class="sample" src="img/s3-card.png" style="border: 1px solid #ccc;">
</div>

### Providing a home screen banner

An application must provide a home screen banner if it includes a Leanback launcher intent filter. The banner is the app launch point that appears on the home screen in the apps and games rows. Describe the banner in the manifest as follows:

<pre>
&lt;application
    ...
    android:banner="&#64;drawable/filmi_banner" &gt;
    ...
&lt;/application&gt;
</pre>

Banners should have a size of: 320 x 180 px, and be included as a xhdpi resource.

### Using Leanback theme

The leanback support library provides a standard theme for TV activities, called Theme.Leanback. This theme establishes a consistent visual style for TV apps.Use of this theme is recommended for most TV apps. The following code sample shows how to apply this theme to a given activity within an app:

<code><pre>
  &lt;activity
    android:name=&quot;.fastlane.<strong>LeanbackActivity</strong>&quot;
    android:label=&quot;&#64;string/title_activity_player&quot;
    android:theme=&quot;&#64;style/Theme.Leanback&quot;&gt;
    ....
  &lt;/activity&gt;
</code></pre>

### Declaring a TV Activity

An application intended to run on TV devices must declare a launcher activity for TV in its manifest using a CATEGORY_LEANBACK_LAUNCHER intent filter. This filter identifies your app as being enabled for TV, and is required for your app to be considered a TV app in Google Play. Declaring this intent also identifies which activity in your app to launch when a user selects its icon on the TV home screen.

The following code snippet shows how to include this intent filter in your manifest. The  activity manifest entry in this example specifies that activity as the one to launch on a TV device.

<code><pre>&lt;activity
            android:name="com.android.example.leanback.fastlane.LeanbackActivity"
            android:label="@string/title_activity_player"
            android:theme="@style/AppTheme"&gt;
            &lt;intent-filter&gt;
                &lt;action android:name="android.intent.action.MAIN" /&gt;
                &lt;category android:name="android.intent.category.LEANBACK_LAUNCHER" /&gt;
            &lt;/intent-filter&gt;
        &lt;/activity&gt;
</code></pre>

Caution: If you do not include the CATEGORY_LEANBACK_LAUNCHER intent filter in your app, it is not visible to users running the Google Play store on TV devices. Also, if your app does not have this filter when you load it onto a TV device using developer tools, the app does not appear in the TV user interface.

### Create a Media Browse Layout

The Android framework provides classes for building user interfaces for these types of apps with the v17 leanback support library. This library provides a framework of classes for creating an efficient and familiar interface for browsing and playing media files with minimal coding. The classes are designed be extended and customized so you can create an experience that is unique to your app.

### Creating LeanbackActivity


### Displaying Media Lists



### Next up

Creating the video details activity

