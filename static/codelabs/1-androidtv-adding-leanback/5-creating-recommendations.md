<toc-element></toc-element>

When interacting with TVs, users generally prefer to give minimal input before watching content. An ideal scenario for many TV users is: sit down, turn on, and watch. The fewest steps to get users to content they enjoy is generally the path they prefer.

Content recommendations appear as the first row of the TV launch screen after the first use of the device. Contributing recommendations from your app's content catalog can help bring users back to your app.

In this step, you'll learn how to create recommendations and provide them to the Android framework so your app content can be easily discovered and enjoyed by users.

In this step, you'll learn about:

-   Creating a Recommendations Service
-   Building Recommendations
-   Running the Recommendations Service

<div layout vertical center>
  <img class="sample" src="img/checkpoint-5-home-recommendations.png" style="border: 1px solid #ccc;">
</div>

### Creating a Recommendations Service

Content recommendations are created with background processing. In order for your application to contribute to recommendations, create a service that periodically adds listings from your app's catalog to the system list of recommendations.

The following code example illustrates how to extend <a href="http://developer.android.com/reference/android/app/IntentService.html"><code>IntentService</code></a> to create a recommendation service for your application:

<code><pre>public class RecommendationsService extends IntentService {
  private static final String TAG = "RecommendationsService";
  private static final int MAX_RECOMMENDATIONS = 3;
  public static final String EXTRA_BACKGROUND_IMAGE_URL = "background_image_url";
  private static final int DETAIL_THUMB_WIDTH = 274;
  private static final int DETAIL_THUMB_HEIGHT = 274;
  private NotificationManager mNotificationManager;

  ...
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

              buildRecommendation(getApplicationContext(), video, count);
              count++;
          }
          cursor.close();
      } catch (RemoteException re) {

      } catch (IOException re) {

      } finally {
          mNotificationManager = null;
      }
  }
</pre></code>

In order for this service to be recognized by the system and run, register it using your app manifest. The following code snippet illustrates how to declare this class as a service:

<code><pre>
&lt;manifest ... &gt;
  &lt;application ... &gt;
    ...
    &lt;service android:name=&quot;com.android.example.leanback.fastlane.RecommendationsService&quot;
        android:enabled=&quot;true&quot; android:exported=&quot;true&quot;/&gt;
  &lt;/application&gt;
&lt;/manifest&gt;
</pre></code>

### Building Recommendations

Once your recommendation server starts running, it must create recommendations and pass them to the Android framework. The framework receives the recommendations as Notification objects that use a specific template and are marked with a specific category.

The following code example demonstrates how to get an instance of the NotificationManager, build a recommendation, and post it to the manager:

<code><pre>
....
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
...
</pre></code>

### Running the Recommendations Service

Your app's recommendation service must run periodically in order to create current recommendations. To run your service, create a class that runs a timer and invokes it at regular intervals. The following code example extends the BroadcastReceiver class to start periodic execution of a recommendation service every 1/2 hour:

<code><pre>public class BootCompleteReceiver extends BroadcastReceiver {
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
</pre></code>

This implementation of the BroadcastReceiver class must run after start up of the TV device where it is installed. To accomplish this, register this class in your app manifest with an intent filter that listens for the completion of the device boot process. The following sample code demonstrates how to add this configuration to the manifest:
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

<strong>Important:</strong> Receiving a boot completed notification requires that your app
  requests the <code><a href="/reference/android/Manifest.permission.html#RECEIVE_BOOT_COMPLETED">RECEIVE_BOOT_COMPLETED</a></code> permission.
  For more information, see <code><a href="/reference/android/content/Intent.html#ACTION_BOOT_COMPLETED">ACTION_BOOT_COMPLETED</a></code>.

### Next up

Polishing the animations

