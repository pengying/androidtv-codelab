<toc-element></toc-element>

In this step we'll learn about how the Leanback `BrowseFragment` works and put some content into it.

Lets get started.

### Concepts
First, lets cover how the `BrowseFragment` works.  The `BrowseFragment` is basically a View that renders rows of data that you provide.

<figure layout vertical center>
  <img src="img/browse_fragment_model.png" alt="browse fragment" class="noborder" width="600px">
</figure>

Think of each row as two pieces, a [`HeaderItem`](https://developer.android.com/reference/android/support/v17/leanback/widget/HeaderItem.html) which defines the category and an array of objects represented by the [`ListRow`](https://developer.android.com/reference/android/support/v17/leanback/widget/ListRow.html) class which defines the content.

The ArrayObjectAdapter is an array of the defined `ListRows` that aggregates the rows for the `BrowseFragment` view.

We can store any sort of View in ListRows, but in our app we'll use the Leanback [`ImageCardView`](https://developer.android.com/reference/android/support/v17/leanback/widget/ImageCardView.html).  The zoom and additional detail affects are automatically handled by the Leanback library.

To tie your video data and the `ImageCardView` together, we use a `Presenter`.  The `Presenter` defines which elements of the view are populated from which elements of the model.

Lastly we have the [`ViewHolder`](https://developer.android.com/reference/android/support/v17/leanback/widget/Presenter.ViewHolder.html)  which is a container for the created view.

Lets put all of these concepts together to create the video browsing experience.

### Create a presenter

We need to create a presenter to tie our `Video` model to the `ImageCardView`.

&rarr; Under `fastlane` create a new class called CardPresenter and extend Presenter.

&rarr; Define class variables to store the desired `ImageCardView` height and width and the application context.

    private static int CARD_WIDTH = 200;
    private static int CARD_HEIGHT = 200;

    private static Context mContext;

&rarr; Import the `Presenter` package and implement the abstract methods `onCreateViewHolder`, `onBindViewHolder`, and `onUnbindViewHolder`.

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup viewGroup) {
      return null;
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, Object o) {

    }

    @Override
    public void onUnbindViewHolder(ViewHolder viewHolder) {

    }

### Create a Picasso Target to handle image loading

We're leveraging [Picasso](http://square.github.io/picasso/), an open source library that simplifies image loading, caching, and resizing.

<aside class="callout">
The base sample app already uses Picasso so this should be done for you.  But if you want to add it in a separate app, in your build.gradle file add the following dependency.
    <pre>compile 'com.squareup.picasso:picasso:2.3.4'</pre>
</aside>

&rarr; Create a PiccasoImageCardViewTarget implementing Target and implement it's onBitmapLoaded, onBitmapFailed and onPrepareLoad methods.

    static class PicassoImageCardViewTarget implements Target {

      @Override
      public void onBitmapLoaded(Bitmap bitmap, Picasso.LoadedFrom from) {

      }

      @Override
      public void onBitmapFailed(Drawable errorDrawable) {

      }

      @Override
      public void onPrepareLoad(Drawable placeHolderDrawable) {

      }
    }

&rarr; To this class we'll add an instance variable to store the `ImageCardView` we'll draw into once the bitmap is loaded.

    private ImageCardView mImageCardView;

&rarr; Create a constructor with the target `ImageCardView` as the parameter and store it as the instances `mImageCardView`.

    public PicassoImageCardViewTarget(ImageCardView mImageCardView) {
        this.mImageCardView = mImageCardView;
    }

&rarr; In `onBitmapLoaded`, we create a new `Drawable` from the bitmap and set it as the main image for the ImageCardView.

    Drawable bitmapDrawable = new BitmapDrawable(mContext.getResources(), bitmap);
    mImageCardView.setMainImage(bitmapDrawable);

&rarr; In `onBitmapFailed`, we set the ImageCardView image to the error default.

    mImageCardView.setMainImage(drawable);

### Create ViewHolder class

We'll use the `ViewHolder` pattern to store all of the data associated with the view.

&rarr; Create a static class that extends `Presenter.ViewHolder` and create the default constructor.

    static class ViewHolder extends Presenter.ViewHolder {

      public ViewHolder(View view) {
        super(view);
      }
    }

&rarr; Define class variables to store the `ImageCardView`, `Drawable`, and `PicassoImageCardViewTarget`.

    private ImageCardView mCardView;
    private Drawable mDefaultCardImage;
    private PicassoImageCardViewTarget mImageCardViewTarget;

&rarr; In the constructor, cast the view parameter as an `ImageCardView` and store it in `mCardView`.  Instantiate a new `PicassoImageCardViewTarget` passing the cardView as the target parameter.  Finally get the default card image from resources.

    public ViewHolder(View view) {
        super(view);
        mCardView = (ImageCardView) view;
        mImageCardViewTarget = new PicassoImageCardViewTarget(mCardView);
        mDefaultCardImage = mContext
            .getResources()
            .getDrawable(R.drawable.filmi);
    }

&rarr; Add a getter for `mCardView`.

    public ImageCardView getCardView() {
        return mCardView;
    }

&rarr; Create a function that loads the image from a URL.

    protected void updateCardViewImage(String url) {

        Picasso.with(mContext)
                .load(url)
                .resize(CARD_WIDTH, CARD_HEIGHT)
                .centerCrop()
                .error(mDefaultCardImage)
                .into(mImageCardViewTarget);
    }

Now lets create the `ImageCardView` to hold and bind it with some data from the model.

### Creating the ImageCardView

`onCreateViewHolder` is called to create a new view. In it we'll handle the logic of storing the context, and creating a new ImageCardView.

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup viewGroup) {

        Log.d("onCreateViewHolder", "creating viewholder");
        mContext = viewGroup.getContext();
        ImageCardView cardView = new ImageCardView(mContext);
        cardView.setFocusable(true);
        cardView.setFocusableInTouchMode(true);
        ((TextView)cardView.findViewById(R.id.content_text)).setTextColor(Color.LTGRAY);
        return new ViewHolder(cardView);
    }

We set the cardView `Focusable` and `FocusableInTouchMode` to true to enable it to be selected when browsing through the rows of content.

Finally we set the `TextColor` of the ImageCardView to light gray.

### Binding data to the ViewHolder

We define the data binding logic in `onBindViewHolder`.  We can cast the Object that's being passed in as our `Video` data then set the title text, subtext/content text, and image dimensions.  Finally we tell it to load the image with a thumbnail URL.

<img src="img/image_view_card.png">

    @Override
    public void onBindViewHolder(Presenter.ViewHolder viewHolder, Object o) {
        Video video = (Video) o;
        ((ViewHolder) viewHolder).mCardView.setTitleText(video.getTitle());
        ((ViewHolder) viewHolder).mCardView.setContentText(video.getDescription());
        ((ViewHolder) viewHolder).mCardView.setMainImageDimensions(CARD_WIDTH * 2, CARD_HEIGHT * 2);
        ((ViewHolder) viewHolder).updateCardViewImage(video.getThumbUrl());
    }

And our CardPresenter is complete.  Lets fill out some ListRows with our video content.

### Populating the videos

In the `LeanbackBrowseFragment` lets create some sample categories.  Here we're defining them as constants, but in a real app you would probably pull them from your database.

    private static final String[] HEADERS = new String[]{
        "Featured", "Popular","Editor's choice"
    };

Now in `init` after we've set the badge drawable we'll loop through the categories and create a row of content for each one.

In each row, we'll create an ObjectAdapter to define how to render the content that well pull from our database.  We'll load the videos, create a header, finally instantiating a `ListRow` with the header and video data and adding it to `mRowsAdapter`.

    for (int position = 0; position < HEADERS.length; position++) {
        ObjectAdapter rowContents = new CursorObjectAdapter((new SinglePresenterSelector(new CardPresenter())));
        VideoDataManager manager = new VideoDataManager(getActivity(),
            getLoaderManager(),
            VideoItemContract.VideoItem.buildDirUri(),
            rowContents );
        manager.startDataLoading();

        HeaderItem headerItem = new HeaderItem(position, HEADERS[position], null);
        mRowsAdapter.add(new ListRow(headerItem, manager.getItemList()));
    }

Congrats!  You've completed this step.  Try running the App on Android TV.  You should see a screen similar to the one below.

<aside class="callout">
When running the app from Android Studio, the standard launcher activity loads.  On Android TV back out to the home screen and start the app from Android TV to launch into the proper activity.
</aside>

<figure layout vertical center>
  <img src="img/filmi_browse.png" alt="browse fragment" class="noborder" width="600px">
</figure>


### Summary

In this step you've learned about:

- The `BrowseFragment` and how you can populate it with your videos
- Tying your video data to the view through the MVP pattern

### Next up

Lets create the video details activity.
