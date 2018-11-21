# LSL Couple non sit animator

**FIXME - NOT WORKS YET**

Basic idea is make couple animation (like fight) for owner of hud and avatar
what will be choosen (in area 20m max from owner/wearer). Similar to well known 
Kiss&Hug hud.

### How it should works

  - **owner** - person who wear hud/object
  - **target** - avatar who wish animate with me
  
1. **start**
  - get target(avatar) name
  - start listen for owner on channel 1 for 
  - event: _LISTEN_
    - received target name
    - get targetKey (UUID)
    - get targetPos 
    - compute distance from target - llVecDist( llGetPos(), targetPos );
2. walk toward target
  1. **set target**  -  [llTarget](http://wiki.secondlife.com/wiki/LlTarget)(  targetPos, targetDistance );
  - **turn to target** - [llRotLookAt](http://wiki.secondlife.com/wiki/LlRotLookAt)( targetPos, 1.0, 	0.4 );
  - **walk to target** - [llMoveToTarget](http://wiki.secondlife.com/wiki/LlMoveToTarget)( targetPos , 0.4 );
  - **when** ( distance target_owner < 10)  -->  **stop walk**  - [llStopMoveToTarget();](http://wiki.secondlife.com/wiki/LlStopMoveToTarget)
3. **do_fight**
  1. start animation of owner
  2. start animation of target
  3. play sound 
  4. dust particles 
4. wait 10secs
5. stop animations
  - animation of owner
  - animation of target
6. **randomly pick winner** = owner/target
  1. animate winner (owner/target)
  2. animate looser (owner/target)
  3. wait 5 secs
  4. stop animation of winner
  5. stop animation of loser

### Problems

  - start same animation for:
     - owner of object (wear this object)
     - avatar who not wear hud or sit with owner on same object
  - start animation for both (owner/target) in same moment 
  - stop animations in same moment after some defined time

**understand to:**

  - request permissions first
    -  [llRequestPermissions](http://wiki.secondlife.com/wiki/LlGetPermissionsKey)( targetKey , `PERMISSION_TRIGGER_ANIMATION`);
    -  [llRequestPermissions](http://wiki.secondlife.com/wiki/LlRequestPermissions)( ownerKey , `PERMISSION_TRIGGER_ANIMATION`);
  - **in event** -  [`run_time_permissions`](http://wiki.secondlife.com/wiki/Run_time_permissions)
    - must check - `if(perm & PERMISSION_TRIGGER_ANIMATION)`
    - function [LlGetPermissionsKey](http://wiki.secondlife.com/wiki/LlGetPermissionsKey) will tell me who permited animating and for who will start [llStartAnimation](http://wiki.secondlife.com/wiki/LlStartAnimation)("animation_name");


#### Idea what not works  
```javascript

    touch_start(integer num_detected){
        // lets suppose that know UUID of target and keeping it in global targetKey
        // idea what i tried was that first will try animate target 
        // and if it will be granted (permission) then will start animate owner
        // to avoid start owner animation without animating target, it would be silly

        llRequestPermissions( targetKey ,  PERMISSION_OVERRIDE_ANIMATIONS);
    }

    run_time_permissions(integer permissions){    
        if( llGetPermissionsKey() == targetKey){ 
            llStartAnimation("rotate avatar");	// start target animation  
            llRequestPermissions( ownerKey ,  PERMISSION_OVERRIDE_ANIMATIONS);                      
        }
       
        if( llGetPermissionsKey() == ownerKey){
            llStartAnimation("rotate avatar");	// start owner animation

            llPlaySound("some_sound",1.0);
            llSetTimerEvent( 10.0 );			// time after should stop animations of both             
        }
    }

    timer(){
        llStopAnimation("rotate avatar");       // it will stop running animation of owner?
                                                // or target?   or both? in same moment?
                                                //or should use way:  requestion permissions again -> in that stop
        llLoopSound("some_sound");
    }
    
```

#### Or should do it like this?
  - **default state**
    - when know target key and position will do 
      - walk to target
      - jump to state *state animation_start*
  - **state animations_start**
    - state_entry()
      - llRequestPermissions - for target
    - run_time_permissions
      - if is granted for target 
        - start target animation? 
        - llRequestPermissions - for owner
      - if is granted for owner
        - set timer event for time how long with animate both avatars
    - timer()
      state animation_stop - *jump to third state*
  - **state animations_stop**
    - state_entry()
      - llRequestPermissions - for target  
      - llRequestPermissions - for owner
    - run_time_permissions()
      - llStopAnimation
    

