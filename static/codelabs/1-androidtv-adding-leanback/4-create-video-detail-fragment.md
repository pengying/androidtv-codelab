<h1>Create the video detail fragment</h1>

<h2>Goal of checkpoint 3: Add an activity to show details of a video</h2>

* show details of a video in a dedicated Activity
* offer an action to play the video to start the video player
* show related videos in an extra row below the detail info

<b style="color: red">TBD: add screenshot of detail view with related content</b>

<h2>Concepts</h2>

The Leanback library offers ready to use components to create a detail view for a video clip. The detail view shows additonal information about the video and actions like play. Goal of checkpoint 3 is to create an additional Activity containing a <a href="https://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html">DetailsFragment</a> of the Leanback library which is populated with the model data by leveraging the <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter">Model-View-Presenter pattern</a>. 

<h2>Step 1: Create a new activity <code>VideoDetailsActivity</code> in the fastlane package</h2>

<p>In order to display details of an activity we have to create a new class <code>VideoDEtailsActivity</code>:</p>

	public class VideoDetailsActivity extends Activity {
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_leanback_details);
	    }
	}

<p>The content view is defined in <code>R.layout.activity_leanback_details</code> which we have to create:<p>

<code><pre>&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;fragment xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:name=&quot;com.android.example.leanback.fastlane.VideoDetailsFragment&quot;
    android:id=&quot;@+id/details_fragment&quot;
    android:layout_width=&quot;match_parent&quot;
    android:layout_height=&quot;match_parent&quot;
/&gt;</pre></code>


<h2>Step 2: Add the new activity to <i>AndroidManifest.xml</i></h2>

<p>Every Activity needs to be registered in the <i>AndroidManifest.xml</i> so we do:</p>

<code><pre>&lt;activity
android:name=&quot;.fastlane.VideoDetailsActivity&quot;
    	android:label=&quot;@string/title_activity_player&quot;
    	android:theme=&quot;@style/AppTheme&quot;
android:exported=&quot;true&quot;&gt;
&lt;/activity&gt;</pre></code>


<h2>Step 3: Create VideoDetailsFragment</h2>

<p>The layout file of the <code>VideoDetailsActivity</code> references the <code>VideoDetailsFragment</code> which extends the <a href="https://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html">DetailsFragment</a>:</p>

   	public class VideoDetailsFragment extends DetailsFragment {

	    private Video selectedVideo;
	    private static final int DETAIL_THUMB_WIDTH = 274;
	    private static final int DETAIL_THUMB_HEIGHT = 274;

	    private static final int ACTION_PLAY = 1;
	    private static final int ACTION_WATCH_LATER = 2;


	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        selectedVideo = (Video) getActivity().getIntent().getSerializableExtra(LeanbackActivity.VIDEO);
	    }


	    // Utility method for converting dp to pixels
	    public static int dpToPx(int dp, Context ctx) {
	        float density = ctx.getResources().getDisplayMetrics().density;
	        return Math.round((float) dp * density);
	    }
	}
	
<h2>Step 4: Show the <code>VideoDetailActivity</code> when an item is clicked</h2>

<p>Register an <a href="https://developer.android.com/reference/android/support/v17/leanback/widget/OnItemViewClickedListener.html"><code>OnItemViewClickedListener</code></a> in the init method of the <code>LeanbackBrowseFragment</code>:</p>

    setOnItemViewClickedListener(getDefaultItemViewClickedListener());

<p>and create the method <code>getDefaultItemViewClickerListener</code>:</p>

    private OnItemViewClickedListener getDefaultItemViewClickedListener() {
		return new OnItemViewClickedListener() {
	        @Override
	        public void onItemClicked(Presenter.ViewHolder viewHolder, Object o, 
				RowPresenter.ViewHolder viewHolder2, Row row) {

				Intent intent = new Intent(getActivity(), VideoDetailsActivity.class);
				intent.putExtra(LeanbackActivity.VIDEO, (Serializable)o);
	    		startActivity(intent);
			}
		};
    }

<p>As we now pass the <code>Video</code> object with an <code>Intent</code> we need a key <code>LeanbackActivity.VIDEO</code> for it and make the data/Video class <code>Serializable</code>.</p>


<h2>Step 5: Use the Model-View-Presenter pattern to display the details of the video</h2>

<p>The project now compiles and can be run but does not show anything. To make that happen we use a presenter from the Model-View-Presenter pattern which is leveraged by the Leanback library. We create the class DetailsDescriptionPresenter deriven from the <a href="https://developer.android.com/reference/android/support/v17/leanback/widget/AbstractDetailsDescriptionPresenter.html"><code>AbstractDetailsDescriptionPresenter</code></a> which is responsible to put the values of the model (the Video class) to a ViewHolder:</p>

	public class DetailsDescriptionPresenter
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
	}

<h2>Step 6: Create async task to load image</h2>

<p>To put the presenter in use, we have to make the <code>VideoDetailFragment</code> smarter. We add an inner class <code>DetailRowBuilderTask</code> to the fragment which to load the image:</p>

	private class DetailRowBuilderTask extends AsyncTask&lt;Video, Integer, DetailsOverviewRow&gt; {

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
    }

<h2>Step 7: Create presenter when the task has executed</h2>
	
<p>The <a href="http://square.github.io/picasso/">Picasso library</a> loads and and resized the image off the UI thread. After it has completed we create the presenters in the <code>onPostExecute</code> method of the <code>AsyncTask</code> and set the adapter of the <code>DetailsFragment</code></p> 

    @Override
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
	}

<h2>Step 8: Execute the builder task to build the view</h2>

<p>And then we instantiate and execute that DetailRowBuilderTask in the onCreate method of the VideoDetailsFragment:</p>

	new DetailRowBuilderTask().execute(selectedVideo);

<p>Compile, run and see! The video can be played now!</p>


<h2>Step 9: Display a row of related videos below the detail panel</h2>

<p>As an extra step we want to add a row of videos related to the one shown in detail. To do this we only have to add another presenter and adapter in the <code>onPostExecute</code> of the <code>AsyncTask</code>.</p>

<p>We first add an additional presenter</p>

    ps.addClassPresenter(ListRow.class, new ListRowPresenter());
	
<p>Accordinlgy the <code>ArrayObjectAdapter</code> requires an additional <code>ListRow</code> to be added. We create a new <code>ListRow</code> to which we pass a <code>HeaderItem</code> and a <code>CursorObjectAdapater</code> in the constructor</p>

	String subcategories[] = {
		You may also like"
	};

	CursorObjectAdapter rowAdapter = new CursorObjectAdapter(
		new SinglePresenterSelector(new CardPresenter()));

	VideoDataManager manager = new VideoDataManager(getActivity(),getLoaderManager(),
		VideoItemContract.VideoItem.buildDirUri(),rowAdapter);
	manager.startDataLoading();

	HeaderItem header = new HeaderItem(0, subcategories[0], null);
	adapter.add(new ListRow(header, rowAdapter));

<h2>Step 10: Run the app</h2>