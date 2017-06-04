# Explore-AOSP

## Architecture 


##  Deep Dive into State Restoration on Android

### onSaveInstanceState 

* When it is and isn't called ?
- This method is called before an activity may be killed so that when it comes back some time in the future it can restore its state.

- Do not confuse this method with activity lifecycle callbacks such as onPause, which is always called when an activity is being placed
in the background or on its way to destruction, or onStop which is called before destruction.
One example of when onPause and onStop is called and not this method is when a user navigates back from activity B to activity A: there 
is no need to call onSaveInstanceState on B because that particular instance will never be restored, so the system avoids calling it.  
An example when onPause is called and not onSaveInstanceState is when activity B is launched in front of activity A:
the system may avoid calling onSaveInstanceState on activity A if it isn't killed during the lifetime of B since the state of the user interface 
of A will stay intact. 
     
- If called, this method will occur before onStop. There are no guarantees about whether it will occur before or after onPause.
  
- If you call finish() (or your activity is being finished for any other reason), it won't be called because the user will never be able to 
  return to the activity so it will never need to restore its state.

the base method code:
```java
protected void onSaveInstanceState(Bundle outState) {
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
    getApplication().dispatchActivitySaveInstanceState(this, outState);
}
```
First it saves the entire window hierarchy state (by calling onSaveInstanceState on each view in the hierarchy that has and id).

## Bundle 
Represents the mapping from Keys to Parcelable values where each key is part of the state that we want to save and restore.

### Examples in the framework of saving the state

## Window 
Provides standard UI policies like background, title area, default key processing, etc.
