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

Each of the items individual items displayed is an [`ImageCardView`](https://developer.android.com/reference/android/support/v17/leanback/widget/ImageCardView.html).  The zoom and additional detail affects are automatically handled by the Leanback library.

To tie your video data and the `ImageCardView` together, we use a `Presenter`.  The `Presenter` defines which elements of the view are populated from which elements of the model.

Lastly we have the `ViewHolder` pattern which is a container for the created view.

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


### Summary


### Next up

Creating the video details activity

