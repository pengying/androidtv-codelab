<h1>Goal of checkpoint 3: Add an activity to show details of a video</h1>

<h2>Step 1: Create a new Activity VideoDetailsActivity in the fastlane package</h2>

<p>In order to display details of an activity we have to create a new Activity:</p>

	public class VideoDetailsActivity extends Activity {
	    @Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_leanback_details);
	    }
	}

<p>The content view is defined in R.layout.activity_leanback_details which we have to create:<p>

<code><pre>&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;fragment xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;
    android:name=&quot;com.android.example.leanback.fastlane.VideoDetailsFragment&quot;
    android:id=&quot;@+id/details_fragment&quot;
    android:layout_width=&quot;match_parent&quot;
    android:layout_height=&quot;match_parent&quot;
/&gt;</pre></code>


<h2>Step 2: Add the new activity to android_manifest.xml</h2>

<p>Every Activity needs to be registered in the AndroidManifest.xml so we do:</p>

<code><pre>&lt;activity
android:name=&quot;.fastlane.VideoDetailsActivity&quot;
    	android:label=&quot;@string/title_activity_player&quot;
    	android:theme=&quot;@style/AppTheme&quot;
android:exported=&quot;true&quot;&gt;
&lt;/activity&gt;</pre></code>


<h2>Step 3: Create VideoDetailsFragment</h2>

<p>The layout file of the VideoDetailsActivity references the VideoDetailsFragment which extends the DetailsFragment:</p>

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
	
<h2>Step 4: Show the VideoDetailActivity when an item is clicked</h2>

<p>Register an OnItemViewClickedListener in the init method of the LeanbackBrowseFragment:</p>

    setOnItemViewClickedListener(getDefaultItemViewClickedListener());

<p>and create the method getDefaultItemViewClickerListener:</p>

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

<p>As we now pass the Video element with an Intent we need a key LeanbackActivity.VIDEO for it and make the data/Video class Serializable.</p>


<h2>Step 5: Use the Model-View-Presenter pattern to display the details of the video</h2>

<p>The project now compiles and can be run but does not show anything. To make that happen we use a presenter from the Model-View-Presenter pattern which is leveraged by the Leanback library. We create the class DetailsDescriptionPresenter which is responsible to put the values of the model (the Video class) to a ViewHolder:</p>

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

<p>To put the presenter in use we have to make the VideoDetailFragment smarter. Much smarter. We add an inner class DetailRowBuilderTask to the fragment which to load the image:</p>

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
	
<p>The Picasso library loads and and resized the image off the UI thread. After it has completed we create the presenters in the onPostExecute method of the AsyncTask and set the adapter of the DetailsFragment</p> 

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

<p>As an extra step we want to add a row of videos related to the one shown in detail. To do this we only have to add another presenter and adapter in the <code>onPostExecute</code> of the AsyncTask.</p>

<p>We first add an additional presenter</p>

    ps.addClassPresenter(ListRow.class, new ListRowPresenter());
	
<p>Accordinlgy the ArrayObjectAdapter requires an additional ListRow to be added. We create a new ListRow to which we pass a HeaderItem and a CursorObjectAdapater in the constructor</p>

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