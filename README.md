# Better-Android-Dev
Tips and advices collected while learning Android

* ### Using LiveData for navigation and UI events
  * Use an Event wrapper class. [LiveData with SnackBar, Navigation and other events (the SingleLiveEvent case)](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150)
  ```kotlin
  /**
  * Used as a wrapper for data that is exposed via a LiveData that represents an event.
  */
  open class Event<out T>(private val content: T) {

     var hasBeenHandled = false
         private set // Allow external read but not write

     /**
      * Returns the content and prevents its use again.
      */
     fun getContentIfNotHandled(): T? {
         return if (hasBeenHandled) {
             null
         } else {
             hasBeenHandled = true
             content
         }
     }

     /**
      * Returns the content, even if it's already been handled.
      */
     fun peekContent(): T = content
  }

  class EventObserver<T>(private val onEventUnhandledContent: (T) -> Unit) : Observer<Event<T>> {
      override fun onChanged(event: Event<T>?) {
          event?.getContentIfNotHandled()?.let { value ->
              onEventUnhandledContent(value)
          }
      }
  }
  
  fun <T> LiveData<Event<T>>.observeEvent(owner: LifecycleOwner, onChanged: (T) -> Unit) {
      observe(owner, EventObserver(onChanged))
  }

* ### TextSwitcher for easy transition animation in Android texts
  * When used with `ValueAnimator` for timing animations, it can create really good looking animations in a simple way. [TextSwitcher](https://www.zoftino.com/android-textswitcher-tutorial)

  In XML layout:
   ```xml

   <TextSwitcher
              android:id="@+id/textSwitcher"
              android:layout_width="match_parent"
              android:layout_height="wrap_content">

              <TextView
                  android:layout_width="match_parent"
                  android:layout_height="wrap_content"
                  android:layout_gravity="center"
                  android:gravity="center" />

              <TextView
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:layout_gravity="center"
                  android:gravity="center" />
   </TextSwitcher>
  ```

  In activity or fragment:
   ```kotlin

   textSwitcher.setInAnimation(this, R.anim.slide_in_from_bottom)
   textSwitcher.setOutAnimation(this, R.anim.slide_out_to_top)

   ValueAnimator.ofInt(0, animationDuration).apply {
       duration = animationDuration.toLong()
       interpolator = LinearInterpolator()

       addUpdateListener {
           if ((it.animatedValue in valueOfInterest) {
               textSwitcher.setText(texts[keyOrIndex])
           }
       }
       start()
   }
   
* ### Converting an Android Fragment to an Activity

  * Change name and extend `AppCompatActivity` instead of Fragment()
  * Take code from `onCreateView`, `onViewCreated` and `onActivityCreated` and put in into your Activity's `onCreate` or your `init()` method and call that `init()` method from `onCreate`.
  * Replace the `newInstance` method with `getStartIntent` or `startActivity` as per your use case. Change required variable names for the same.
  * Replace all the calls to `getActivity` and `context` with `this` as currently you are in the Activity and it extends the Context class.
  * If you are using `LiveData` observer, instead of passing `viewLifeCycleOwner`, pass `this` as Activity is a lifecycle owner.
  * If using ViewModel, change ViewModel's name to follow activity name.
  * Replace all the `childFragmentManager`s with `supportFragmentManager`.
  * If you are using Dagger for Dependency Injection:
    * Change the name of all the modules to match your activity name.
    * Change the scope of your activity from `PerFragment` to `PerFragment` in your binding module.
    * In all the fragments which were before added using this Fragment's `childFragmentManager`, when getting ViewModel instance using factory, instead of using `parentFragment`, use  `activity` as `parentFragment` will return null if the fragment is attached to an activity.
  * There can be some more minor changes depending on some function calls, where you need to find their alternatives in activity.
  
* ### Activity Orientation facts and tips
  * Use `recreate()` inside the activity to reinstantiate the activity. You can do this when you know that a configuration change should take place. For example, let' say you enabled rotation programmatically for your activity at a later time. Then calling `recreate()` will bring the activity in it's proper orientation as per the device orientation.
  
  * Use `configChanges` attribute in your activity manifest entry to avoid configuration change and an activity recreation when any of the things mentioned in the attribute value occur. [Handling configuration changes](https://developer.android.com/guide/topics/resources/runtime-changes#HandlingTheChange)
  
  * Create a `CustomOrientationEventListener` to listen for orientation changes of your device. This is useful when you do not want to recreate your activity on device rotation as that can be too costly in some cases, e.g, when your activity is a camera screen.
  
  ```kotlin
  abstract class CustomOrientationEventListener(context: Context)
    : OrientationEventListener(context) {

    private var previousOrientation = ORIENTATION_UNKNOWN
    private var currentOrientation = ORIENTATION_UNKNOWN

    override fun onOrientationChanged(orientation: Int) {
        currentOrientation = when (orientation) {
            in 45..134 -> {
                Surface.ROTATION_270
            }
            in 135..224 -> {
                Surface.ROTATION_180
            }
            in 225..314 -> {
                Surface.ROTATION_90
            }
            else -> {
                Surface.ROTATION_0
            }
        }
        if (previousOrientation != currentOrientation && orientation != ORIENTATION_UNKNOWN) {
            previousOrientation = currentOrientation
            if (currentOrientation != ORIENTATION_UNKNOWN) {
                onSimpleOrientationChanged(currentOrientation)
            }
        }
    }

    abstract fun onSimpleOrientationChanged(orientation: Int)
  }

* ### Android RemoteViews
  * A class that describes a view hierarchy that can be displayed in another process. The hierarchy is inflated from a layout resource file, and this class provides some basic operations for modifying the content of the inflated hierarchy.
  * RemoteViews work with only a very limited set of Views and ViewGroups as mentioned here. [Link](https://developer.android.com/reference/android/widget/RemoteViews)
  
* ### RecyclerView ViewHolders partial update

  * [Answer on StackOverflow](https://stackoverflow.com/a/36957892/7891801)
  * You can notify your RecyclerView.Adapter's observers to issue a partial update of your RecyclerView.ViewHolders by passing a payload Object.

 `notifyItemRangeChanged(positionStart, itemCount, payload);`
 
   Where payload could be or contain a flag that represents relative or absolute time. To bind the payload to your view holders, override the following      onBindViewHolder(viewHolder, position, payloads) method in your adapter, and check the payloads parameter for data.

```kotlin
 @Override
 public void onBindViewHolder(MyViewHolder viewHolder, int position, List<Object> payloads) {
     if (payloads.isEmpty()) {
         // Perform a full update
         onBindViewHolder(viewHolder, position);
     } else {
         // Perform a partial update
         for (Object payload : payloads) {
             if (payload instanceof TimeFormatPayload) {
                 viewHolder.bindTimePayload((TimeFormatPayload) payload);
             }
         }
     }
 }
 ```
  Within your MyViewHolder.bindTimePayload(payload) method, update your time TextViews to the time format specified in the payload.
  
* ### An issue with `FragmentPagerAdapter` in `ViewPager`

  * When using `FragmentPagerAdapter`, avoid storing data in instance variables in `Fragment` instance, as they may not have proper state when fragment is removed and added from `ViewPager`. To prevent this issue with instance variables, use `FragmentStatePagerAdapter`.

* ### Injecting ViewModel in Fragment

  * Inject dependencies in `onAttach` befor calling `super.onAttach`. Reason: https://stackoverflow.com/a/46043545/7891801
  * Create your ViewModel in `onViewCreated`. Actually, it can be created in any lifecycle method after `onAttach`, we just need to be consistent about it in our codebase.
  * If using Kotlin, better to use `by viewModels` delegate function for instantiating ViewModel.

* ### Communicating between fragments and other UI components when using Navigation components

  * Use the following extension functions for communicating between fragments and other UI components when using Navigation components, like `startActivityForResult`:

```kotlin
fun <T> Fragment.getNavigationResult(key: String = "result") =
    findNavController().currentBackStackEntry?.savedStateHandle?.getLiveData<T>(key)
    
fun <T> Fragment.setNavigationResult(result: T, key: String = "result") {
    findNavController().previousBackStackEntry?.savedStateHandle?.set(key, result)
}
```

* ### Second thought on using `lazy` in classes serialized using `Gson`

   * Gson tries to serialize property delegates created using `lazy` but fails to do so since there is no backing field for a delegate property by default because it might not store an actual value. This is a general delegate definition, although when we look at the `Lazy<T>` implementation, we can see that it has a value `field` but it's unknow to Gson for obvious reasons. [More info on delegate properties.](https://kotlinlang.org/docs/delegated-properties.html) This results in a crash as:
  ```kotlin
  Caused by: java.lang.InstantiationException: can't instantiate class kotlin.Lazy
  ```
  
   * We can annotate the delegate field with a `@Transient` annotation. But using it directly results in the crash:
   ```kotlin
   This annotation is not applicable to target 'member property without backing field'
   ```
   
   * To fix this, we can use this annotation with the [annotation use-site target](https://kotlinlang.org/docs/annotations.html#annotation-use-site-targets) `delegate` which puts the annotation on "the field storing the delegate instance for a delegated property", with the syntax as `@delegate:Transient`. As pointed out earlier, since `Lazy` has a field for storing the value, this method works. Example:
   ```kotlin
   @delegate:Transient val area by lazy { calculateArea() }
   ```
