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

in Activity the base method code:
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
and mWindow has PhoneWindow implementation so the code for saveHierarchyState :
```java
    public Bundle saveHierarchyState() {
        Bundle outState = new Bundle();
        if (mContentParent == null) {
            return outState;
        }

        SparseArray<Parcelable> states = new SparseArray<Parcelable>();
        mContentParent.saveHierarchyState(states);
        outState.putSparseParcelableArray(VIEWS_TAG, states);

        // Save the focused view ID.
        final View focusedView = mContentParent.findFocus();
        if (focusedView != null && focusedView.getId() != View.NO_ID) {
            outState.putInt(FOCUSED_ID_TAG, focusedView.getId());
        }

        // save the panels
        SparseArray<Parcelable> panelStates = new SparseArray<Parcelable>();
        savePanelState(panelStates);
        if (panelStates.size() > 0) {
            outState.putSparseParcelableArray(PANELS_TAG, panelStates);
        }

        if (mDecorContentParent != null) {
            SparseArray<Parcelable> actionBarStates = new SparseArray<Parcelable>();
            mDecorContentParent.saveToolbarHierarchyState(actionBarStates);
            outState.putSparseParcelableArray(ACTION_BAR_TAG, actionBarStates);
        }

        return outState;
    }
```
the most important part is mContentParent saveHierarchyState call, mContentParent is the view in which the entire window content is placed
so let's take a look at View's saveHierarchyState to understand what gets saved:
```java 
public void saveHierarchyState(SparseArray<Parcelable> container) {
    dispatchSaveInstanceState(container);
}

protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
  if (mID != NO_ID && (mViewFlags & SAVE_DISABLED_MASK) == 0) {
      mPrivateFlags &= ~PFLAG_SAVE_STATE_CALLED;
      Parcelable state = onSaveInstanceState();
      if ((mPrivateFlags & PFLAG_SAVE_STATE_CALLED) == 0) {
          throw new IllegalStateException(
                "Derived class did not call super.onSaveInstanceState()");
      }
      if (state != null) {
          container.put(mID, state);
      }
  }
}     
```
NO_ID is just a value to mark a view that has no id so the state for views without ids is not saved.
the second condition is for checking if the property saveEnabled is not set to false (can be set for example in any view's xml with 
 android:saveEnable="false") 
 then we get the state to be saved from the view's onSaveInstanceState to get the view and its children state which can be overriden by implementers and
 put the state with the id as its key.
dispatchSaveInstanceState can be overriden to change how freezing happens to a view's children.

let's jump back to the rest of saveHierarchyState the currently focused view id is saved in the state to be restored and the view is focused on restoration 
 as in restoreHierarchyState :
 ```java 
  int focusedViewId = savedInstanceState.getInt(FOCUSED_ID_TAG, View.NO_ID);
  if (focusedViewId != View.NO_ID) {
     View needsFocus = mContentParent.findViewById(focusedViewId);
     if (needsFocus != null) {
        needsFocus.requestFocus();
     } 
     //.... 
 ```
 then panels and toolbar state are saved.
 
 and lets jump back up again to the caller onSaveInstanceState in Activity where after saving hierarchy state the fragments states are saved
 //TODO: explore fragments state save 
 then Application object is notified that activity saveInstanceState has been called so that lifecycle callback for onActivitySaveInstanceState is triggered.
 
 Where is onSaveInstanceState called ?
 performSaveInstanceState called by callCallActivityOnSaveInstanceState of ActivityThread (main entry point of an android application : it has the static main 
 java method that starts any  android app) which <b>may be called</b> on 4 cases: 
    - handleRelaunchActivity 
    - handleSleeping 
    - performPauseActivity
    - performStopActivityInner 
    
  which are all cases in handleMessage of ActivityThread handler, most calls sending messages (for PAUSE, STOP, LAUNCH, CONGIGURATION CHANGES, ...) are scheduleXXXActivity 
  (ex: schedulePauseActivity, scheduleStopActivity, ...) or the ApplicationThread class from there it gets to Binder territory that i dont understand yet (TODO).
    
 
## Bundle 
Represents the mapping from Keys to Parcelable values where each key is part of the state that we want to save and restore.

### Saving and restoring state for custom views

#### Saving and restoring state for compound views 

### Examples in the framework of saving the state

#### CompoundButton
A button with two states, checked and unchecked

#### TextView
- setFreezesText 

#### EditText

#### ScrollView 

## ActivityThread 


## Window 
Provides standard UI policies like background, title area, default key processing, etc.

### DecorView

## View 

## ViewGroup


## Text

### [Draft] TextSelection

#### TextSelection Actions 
- ACTION_PROCESS_TEXT 
- change I62ec618010edf01da41338c8c1a7dd4292a15227 adding dispatchActivityResult to dispatch results to views for intents
  that are selected for selection 


Related:
[In-app translations in Android Marshmallow](https://android-developers.googleblog.com/2015/10/in-app-translations-in-android.html)

## Contextual Menus

## Animation

### Transitions 

