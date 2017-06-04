# Explore-AOSP


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
 
     
      
      
