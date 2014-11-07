<toc-element></toc-element>

The Leanback library include classes for displaying additional information about a media item, such as a description or reviews, and for taking action on that item, such as purchasing it or playing its content. This lesson discusses how to create a presenter class for media item details, and how to extend the DetailsFragment class to implement a details view for a media item when it is selected by a user.

In this step, you'll learn about:

-   Showing details of a video in a dedicated Activity.
-   Creating an action to play the video to start the video player.
-   Displaying related videos in an row below the detail card.

<b style="color: red">TBD: add screenshot of detail view with related content</b>
<div layout vertical center>
  <img class="sample" src="img/s4-card.png" style="border: 1px solid #ccc;">
</div>

### Creating a new activity VideoDetailsActivity

In this lesson we will create an additional Activity containing a <a href="https://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html">DetailsFragment</a> of the Leanback library which is populated with the model data by leveraging the <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter">Model-View-Presenter pattern</a>.

Fragments such as the <a href="https://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html">DetailsFragment</a> must be contained within an activity in order to be used for display. Creating an activity for your details view, separate from the browse activity, enables you to invoke your details view using an Intent. This section explains how to build an activity to contain your implementation of the detail view for your media items.

Start creating the details activity by building a layout that references your implementation of the DetailsFragment:
The content view is defined in <code>R.layout.activity_leanback_details</code> which we have to create:

<code><pre>&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;fragment xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:name=&quot;com.android.example.leanback.fastlane.VideoDetailsFragment&quot;
    android:id=&quot;@+id/details_fragment&quot;
    android:layout_width=&quot;match_parent&quot;
    android:layout_height=&quot;match_parent&quot;
/&gt;</pre></code>

Next, create an activity class that uses the layout shown in the previous code example:
<code><pre>
public class VideoDetailsActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_leanback_details);
    }
}
</pre></code>


### Adding the new activity to <i>AndroidManifest.xml</i>

Add this new activity to the manifest. Remember to apply the Leanback theme to ensure that the user interface is consistent with the media browse activity:

<code><pre>&lt;activity
android:name=&quot;.fastlane.VideoDetailsActivity&quot;
    	android:label=&quot;@string/title_activity_player&quot;
    	android:theme=&quot;@style/Theme.Leanback&quot;
android:exported=&quot;true&quot;&gt;
&lt;/activity&gt;</pre></code>


### Creating VideoDetailsFragment

The layout file of the <code>VideoDetailsActivity</code> references the <code>VideoDetailsFragment</code> which extends the <a href="https://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html">DetailsFragment</a>:
The following example code demonstrates how to use the presenter class shown in the previous section, to add a preview image and actions for the media item being viewed. This example also shows the addition of a related media items row, which appears below the details listing.

<code><pre>public class VideoDetailsFragment extends DetailsFragment {
    private Video selectedVideo;
    private static final int DETAIL_THUMB_WIDTH = 274;
    private static final int DETAIL_THUMB_HEIGHT = 274;
    private static final int ACTION_PLAY = 1;
    private static final int ACTION_WATCH_LATER = 2;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        selectedVideo = (Video) getActivity().getIntent().
        				getSerializableExtra(LeanbackActivity.VIDEO);
    }
    // Utility method for converting dp to pixels
    public static int dpToPx(int dp, Context ctx) {
        float density = ctx.getResources().getDisplayMetrics().density;
        return Math.round((float) dp * density);
    }
}</pre></code>


### Defining a Listener for Clicked Items

After you have implemented the DetailsFragment, modify your main media browsing view to move to your details view when a user clicks on a media item. In order to enable this behavior, add an <a href="https://developer.android.com/reference/android/support/v17/leanback/widget/OnItemViewClickedListener.html"><code>OnItemViewClickedListener</code></a> object to the LeanbackBrowseFragment that fires an intent to start the item details activity.

The following example shows how to implement a listener to start the details view when a user clicks a media item in the LeanbackBrowseFragment:

<code><pre>public class LeanbackBrowseFragment extends BrowseFragment {
...
    public void init() {
    	...
    	setOnItemViewClickedListener(getDefaultItemViewClickedListener());
    	...
	}
...
}</pre></code>

and create the method <code>getDefaultItemViewClickerListener</code>:

<code><pre>private OnItemViewClickedListener getDefaultItemViewClickedListener() {
	return new OnItemViewClickedListener() {
        @Override
        public void onItemClicked(Presenter.ViewHolder viewHolder, Object o,
			RowPresenter.ViewHolder viewHolder2, Row row) {

			Intent intent = new Intent(getActivity(), VideoDetailsActivity.class);
			intent.putExtra(LeanbackActivity.VIDEO, (Serializable)o);
    		startActivity(intent);
		}
	};
}</pre></code>

<p>As we now pass the <code>Video</code> object with an <code>Intent</code> we need a key <code>LeanbackActivity.VIDEO</code> for it and make the data/Video class <code>Serializable</code>.</p>


### Building a Details Presenter

In the media browsing framework provided by the leanback library, you use presenter objects to control the display of data on screen, including media item details. The framework provides the <a href="https://developer.android.com/reference/android/support/v17/leanback/widget/AbstractDetailsDescriptionPresenter.html"><code>AbstractDetailsDescriptionPresenter</code></a> class for this purpose, which is a nearly complete implementation of the presenter for media item details. All you have to do is implement the onBindDescription() method to bind the view fields to your data objects, as shown in the following code sample:

<code><pre>public class DetailsDescriptionPresenter
	extends AbstractDetailsDescriptionPresenter {
	@Override
	protected void onBindDescription(AbstractDetailsDescriptionPresenter.ViewHolder viewHolder,
		Object item) {
		Video video = (Video) item;
		if (video != null) {
			viewHolder.getTitle().setText(video.getTitle());
			viewHolder.getSubtitle().setText(String.valueOf(video.getRating()));
			viewHolder.getBody().setText(video.getDescription());
		}
	}
}</pre></code>

### Creating an Async task to load images

To put the presenter in use, we have to make the <code>VideoDetailFragment</code> smarter. We add an inner class <code>DetailRowBuilderTask</code> to the fragment to load the images:
<code><pre>private class DetailRowBuilderTask extends AsyncTask&lt;Video, Integer, DetailsOverviewRow&gt; {
    @Override
    protected DetailsOverviewRow doInBackground(Video... videos) {
        DetailsOverviewRow row = new DetailsOverviewRow(videos[0]);
        Bitmap poster = null;
        try {
			// the Picasso library helps us dealing with images
            poster = Picasso.with(getActivity())
                    .load(videos[0].getThumbUrl())
                    .resize(dpToPx(DETAIL_THUMB_WIDTH, getActivity().getApplicationContext()),
                          dpToPx(DETAIL_THUMB_HEIGHT, getActivity().getApplicationContext()))
                    .centerCrop()
                    .get();
        } catch (IOException e) {
            e.printStackTrace();
        }
        row.setImageBitmap(getActivity(), poster);
        row.addAction(new Action(ACTION_PLAY, getResources().getString(R.string.action_play)));
        row.addAction(new Action(ACTION_WATCH_LATER, getResources().getString(R.string.action_watch_later)));
        return row;
    }
    @Override
    protected void onPostExecute(DetailsOverviewRow detailRow) {
        // implemented in next step
    }
}</pre></code>

### Creating a Presenter when the task has finished executing

The <a href="http://square.github.io/picasso/">Picasso library</a> loads and and resizes the image off the UI thread. After it has completed we create the presenters in the <code>onPostExecute</code> method of the <code>AsyncTask</code> and set the adapter of the <code>DetailsFragment</code>
<code><pre>@Override
protected void onPostExecute(DetailsOverviewRow detailRow) {
    ClassPresenterSelector ps = new ClassPresenterSelector();
    DetailsOverviewRowPresenter dorPresenter = new DetailsOverviewRowPresenter(
		new DetailsDescriptionPresenter());
	// add some style
	dorPresenter.setBackgroundColor(getResources().getColor(R.color.primary));
    dorPresenter.setStyleLarge(true);
	// we listen to two different actions: play and show
	dorPresenter.setOnActionClickedListener( new OnActionClickedListener() {
        @Override
        public void onActionClicked(Action action) {
            if (action.getId() == ACTION_PLAY) {
                Intent intent = new Intent(getActivity(), PlayerActivity.class);
				intent.putExtra(LeanbackActivity.VIDEO, (Serializable)selectedVideo);
                startActivity(intent);
            } else {
                Toast.makeText(getActivity(), action.toString(), Toast.LENGTH_SHORT).show();
            }
        }
    });

    ps.addClassPresenter(DetailsOverviewRow.class, dorPresenter);

	ArrayObjectAdapter adapter = new ArrayObjectAdapter(ps);
    adapter.add(detailRow);
	// finally we set the adapter of the DetailsFragment
    setAdapter(adapter);
}</pre></code>

### Executing the builder task to build the view

And then we instantiate and execute that DetailRowBuilderTask in the onCreate method of the VideoDetailsFragment:
<code><pre>
	new DetailRowBuilderTask().execute(selectedVideo);
</pre></code>

Compile, run and watch how you can play the video now!

### Displaying a row of related videos below the detail panel

As an additional step, you may add a row of related videos below the detail card. To do this we only have to add another presenter and adapter to the <code>onPostExecute</code> method of the <code>AsyncTask</code>.

First, let's add an additional presenter:
<code><pre>
    ps.addClassPresenter(ListRow.class, new ListRowPresenter());
</pre></code>

Accordingly the <code>ArrayObjectAdapter</code> requires an additional <code>ListRow</code> to be added. We create a new <code>ListRow</code> to which we pass a <code>HeaderItem</code> and a <code>CursorObjectAdapater</code> in the constructor.
<code><pre>String subcategories[] = {
	You may also like"
};
CursorObjectAdapter rowAdapter = new CursorObjectAdapter(
	new SinglePresenterSelector(new CardPresenter()));
VideoDataManager manager = new VideoDataManager(getActivity(),getLoaderManager(),
	VideoItemContract.VideoItem.buildDirUri(),rowAdapter);
manager.startDataLoading();
HeaderItem header = new HeaderItem(0, subcategories[0], null);
adapter.add(new ListRow(header, rowAdapter));</pre></code>

### Next up

Creating recommendations that display on the home screen
