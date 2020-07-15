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
