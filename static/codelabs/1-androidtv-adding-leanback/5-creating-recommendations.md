<toc-element></toc-element>

When interacting with TVs, users generally prefer to give minimal input before watching content. An ideal scenario for many TV users is: sit down, turn on, and watch. The fewest steps to get users to content they enjoy is generally the path they prefer.

Content recommendations appear as the first row of the TV launch screen after the first use of the device. Contributing recommendations from your app's content catalog can help bring users back to your app.

In this step, you'll learn how to create recommendations and provide them to the Android framework so your app content can be easily discovered and enjoyed by users.

In this step, you'll learn about:

-   Creating a Recommendations Service
-   Building Recommendations
-   Running the Recommendations Service

<figure layout vertical center>
  <img class="sample" src="img/checkpoint-5-home-recommendations.png" width="600px">
</figure>

### Create a Recommendations Service

Content recommendations are created with background processing. In order for your application to contribute to recommendations, create a service that periodically adds listings from your app's catalog to the system list of recommendations.

&rarr; Under `fastlane` create a new class `RecommendationsService` extending [`IntentService`](http://developer.android.com/reference/android/app/IntentService.html).

    public class RecommendationsService extends IntentService

&rarr; We'll define a few constants for rendering and tagging, and define a `NotificationManager`

    private static final String TAG = "RecommendationsService";
    private static final int MAX_RECOMMENDATIONS = 3;
    public static final String EXTRA_BACKGROUND_IMAGE_URL = "background_image_url";
    private static final int DETAIL_THUMB_WIDTH = 274;
    private static final int DETAIL_THUMB_HEIGHT = 274;
    private NotificationManager mNotificationManager;

&rarr;  Create the default constructor.

    public RecommendationsService() {
        super("RecommendationsService");
    }

&rarr; Next override the `onHandleIntent` function.  As an example recommendation service, we'll
use the same video selections as the browse fragment.  We store a `ContentProviderClient`, then
create a `Cursor` from the client.

    @Override
    protected void onHandleIntent(Intent intent) {
        ContentProviderClient client = getContentResolver()
            .acquireContentProviderClient(VideoItemContract.VideoItem.buildDirUri());
        try {
            Cursor cursor = client.query(VideoItemContract.VideoItem.buildDirUri(),
                VideoDataManager.PROJECTION,
                null,
                null,
                VideoItemContract.VideoItem.DEFAULT_SORT);

&rarr; Instantiate a `VideoItemMapper` that we've defined in `VideoDataManager` and map it to
cursor with `bindColumns`.

    VideoDataManager.VideoItemMapper mapper = new VideoDataManager.VideoItemMapper();
          mapper.bindColumns(cursor);

&rarr;  Instantiate a `NotificationManager`.

    mNotificationManager = (NotificationManager) getApplicationContext()
          .getSystemService(Context.NOTIFICATION_SERVICE);

&rarr; Create a counter for the iteration to create recommendations up to `MAX_RECOMMENDATIONS`.

    int count = 1;

&rarr; Loop through the cursor until we're out of recommendations or we've hit our max and create
 pending intents for each.  The pending intents will direct the user to the details view of the
 video.

    while (cursor.moveToNext() && count <= MAX_RECOMMENDATIONS) {
        Video video = mapper.bind(cursor);
        PendingIntent pendingIntent = buildPendingIntent(video);
        Bundle extras = new Bundle();
        extras.putString(EXTRA_BACKGROUND_IMAGE_URL, video.getThumbUrl());
        count++;
    }

&rarr; Create a utility function to create the `PendingIntent` from `Video`.

    private PendingIntent buildPendingIntent(Video video) {
        Intent detailsIntent = new Intent(this, PlayerActivity.class);
        detailsIntent.putExtra(Video.INTENT_EXTRA_VIDEO, video);

        TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
        stackBuilder.addParentStack(VideoDetailsActivity.class);
        stackBuilder.addNextIntent(detailsIntent);
        // Ensure a unique PendingIntents, otherwise all recommendations end up with the same
        // PendingIntent
        detailsIntent.setAction(Long.toString(video.getId()));

        PendingIntent intent = stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
        return intent;
    }

&rarr; Finally close the cursor and catch potential errors and you should have something similar to
the code below.

    @Override
    protected void onHandleIntent(Intent intent) {
        ContentProviderClient client = getContentResolver().acquireContentProviderClient(VideoItemContract.VideoItem.buildDirUri());
        try {
            Cursor cursor = client.query(VideoItemContract.VideoItem.buildDirUri(), VideoDataManager.PROJECTION, null, null, VideoItemContract.VideoItem.DEFAULT_SORT);

            VideoDataManager.VideoItemMapper mapper = new VideoDataManager.VideoItemMapper();
            mapper.bindColumns(cursor);

            mNotificationManager = (NotificationManager) getApplicationContext()
                    .getSystemService(Context.NOTIFICATION_SERVICE);

            int count = 1;
            while (cursor.moveToNext() && count <= MAX_RECOMMENDATIONS) {
                Video video = mapper.bind(cursor);
                PendingIntent pendingIntent = buildPendingIntent(video);
                Bundle extras = new Bundle();
                extras.putString(EXTRA_BACKGROUND_IMAGE_URL, video.getThumbUrl());
                count++;
            }
            cursor.close();
        } catch (RemoteException re) {

        } catch (IOException re) {

        } finally {
            mNotificationManager = null;
        }
    }

### Building Recommendations

Once the recommended videos are loaded, the service must create recommendations and pass them to the Android framework. The framework receives the recommendations as Notification objects that use a specific template and are marked with a specific category.

The following code example demonstrates how to get an instance of the `NotificationManager`, build a recommendation, and pass it to the manager.  This code needs to be added in the `while loop` after
 the `PendingIntent` has been created.

    Bitmap image = Picasso.with(getApplicationContext())
            .load(video.getThumbUrl())
            .resize(VideoDetailsFragment.dpToPx(DETAIL_THUMB_WIDTH, getApplicationContext()), VideoDetailsFragment.dpToPx(DETAIL_THUMB_WIDTH, getApplicationContext()))
            .get();

    Notification notification = new NotificationCompat.BigPictureStyle(
            new NotificationCompat.Builder(getApplicationContext())
                    .setContentTitle(video.getTitle())
                    .setContentText(video.getDescription())
                    .setPriority(4)
                    .setLocalOnly(true)
                    .setOngoing(true)
                    .setColor(getApplicationContext().getResources().getColor(R.color.primary))
                    .setCategory(Notification.CATEGORY_RECOMMENDATION)
                    .setCategory("recommendation")
                    .setLargeIcon(image)
                    .setSmallIcon(R.drawable.ic_stat_f)
                    .setContentIntent(pendingIntent)
                    .setExtras(extras))
            .build();
    mNotificationManager.notify(count, notification);


### Add recommendation service to manifest

In order for this service to be recognized by the system and run, register it using your app manifest. The following code snippet illustrates how to declare this class as a service:

<pre>
&lt;manifest ... &gt;
  &lt;application ... &gt;
    ...
    &lt;service android:name=&quot;com.android.example.leanback.fastlane.RecommendationsService&quot;
        android:enabled=&quot;true&quot; android:exported=&quot;true&quot;/&gt;
  &lt;/application&gt;
&lt;/manifest&gt;
</pre>


### Run the recommendations service

Your app's recommendation service must run periodically in order to create current recommendations. To run your service, create a class that runs a timer and invokes it at regular intervals. The following code example extends the `BroadcastReceiver` class to start periodic execution of a recommendation service every 1/2 hour:

    public class BootCompleteReceiver extends BroadcastReceiver {
        private static final long INITIAL_DELAY = 5000;

        public BootCompleteReceiver() {
        }

        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent.getAction().endsWith(Intent.ACTION_BOOT_COMPLETED)) {
                scheduleRecommendationUpdate(context);
            }
        }
        private void scheduleRecommendationUpdate(Context context) {

            AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
            Intent recommendationIntent = new Intent(context, RecommendationsService.class);
            PendingIntent alarmIntent = PendingIntent.getService(context, 0, recommendationIntent, 0);

            alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
                    INITIAL_DELAY,
                    AlarmManager.INTERVAL_HALF_HOUR,
                    alarmIntent);
        }
    }

### Add boot receiver to Android manifest

This implementation of the `BroadcastReceiver` class must run after start up of the TV device where it is installed. To accomplish this, register this class in your app manifest with an intent filter that listens for the completion of the device boot process. The following  code demonstrates how to add this configuration to the manifest.

<code><pre>&lt;manifest ... &gt;
  &lt;application ... &gt;
    &lt;receiver android:name=&quot;com.android.example.leanback.fastlane.BootCompleteReceiver&quot; android:enabled=&quot;true&quot;
              android:exported=&quot;false&quot;&gt;
      &lt;intent-filter&gt;
        &lt;action android:name=&quot;android.intent.action.BOOT_COMPLETED&quot;/&gt;
      &lt;/intent-filter&gt;
    &lt;/receiver&gt;
  &lt;/application&gt;
&lt;/manifest&gt;
</pre></code>

<aside class="callout"><strong>Important:</strong> Receiving a boot completed notification requires that your app
  requests the <code><a href="/reference/android/Manifest.permission.html#RECEIVE_BOOT_COMPLETED">RECEIVE_BOOT_COMPLETED</a></code> permission.
  For more information, see <code><a href="/reference/android/content/Intent.html#ACTION_BOOT_COMPLETED">ACTION_BOOT_COMPLETED</a></code>.
</aside>

Congrats, you've completed adding recommendations for your app.

Try running it and you should start seeing recommendations after 30 minutes.  If you want to see
something immediately, star the service through adb.

    adb shell am startservice com.android.example.leanback/.fastlane.RecommendationsService

### Summary

In this step you've learned about:

- Ideal user flow
- Creating and starting a recommendations service

### Next up

Adding polish animations and transitions.

