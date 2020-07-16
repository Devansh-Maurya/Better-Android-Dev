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
    
