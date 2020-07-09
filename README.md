# Better-Android-Dev
Tips and advices collected while learning Android

* ### Using LiveData for navigation and UI events
  * Use an Event wrapper class. ([link](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150))
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
```
