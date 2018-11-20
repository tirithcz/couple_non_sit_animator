# LSL Couple non sit animator

**FIXME - NOT WORKS YET**

Basic idea is make couple animation (like fight) for owner of hud and avatar
what will be choosen (in area 20m max from owner/wearer). Similar to well known 
Kiss&Hug hud.

### How it should works

  1. winer - person who wear hud/object
  1. target - avatar who wish animate with me
  * **get target(avatar) name** 
   - start listen for owner on channel 1 for 
   - event: _LISTEN_
      1.  received target name
      - get targetKey (UUID)
      - get targetPos 
      - compute distance from target - llVelMag
  1. **walk toward target**
    1.  **set target**  -  ` llTarget(  targetPos, targetDistance );`
    - turn to target
    - **walk to target** -  `llMoveToTarget( targetPos , 0.4 );`
    - **when** ( distance target_owner < 10)  -->  **stop walk**  - `llStopMoveToTarget();`
  1. **do_fight**
    1.    start animation of owner
    - start animation of target
    - play sound 
    - dust particles 
  1. wait 10secs
  1. stop animations
    - animation of owner
    - animation of target
  1. stop owner animation 
  1. **randomly pick winner** = owner/target
    1.  animate winner (owner/target)
    - animate looser (owner/target)
    - wait 5 secs
    - stop animation of winner
    - stop animation of loser

### Problems

  - start same animation for:
     - owner of object (wear this object)
     - avatar who not wear hud or sit with owner on same object
  - start animation for both (owner/target) in same moment 
  - stop animations in same moment after some defined time

**understand to:**

  - request permissions first 
    -  ` llRequestPermissions( targetKey , PERMISSION_TRIGGER_ANIMATION);`
    -  ` llRequestPermissions( ownerKey , PERMISSION_TRIGGER_ANIMATION);`
  - in event:  `run_time_permissions`
    - must check - `if(perm & PERMISSION_TRIGGER_ANIMATION)`
    - in event am able get information - UUID (key) of who granted/deny permission request


### Idea what not works  
``javascript

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

