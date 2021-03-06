#Zark's notes on UE4's [C++ oriented] stuff that are confusing , lack documentation and proper explanation. Tricks included - Must read for beginners

Tip: Don't change versions while developing/learning. Stuff change and stuff WILL break.

Mobile (especially android devices) developers beware! Different devices combined with different OS versions lack 
specific features (AO,SSR,Different OGL Versions). Keep that in mind when trying to use new technologies to achieve
special graphic effects.

Want to package for 32-bit windows and your build crashes? Make sure you have downloaded and using the source version
of the engine.

I like: Making C++ classes, coding all the behavior there. Then creating blueprints that edit defaults only.
​
TIP: Don't let these debug messages and traces get to production , use:
​
```cpp

#include "Engine.h"
...
#if UE_BUILD_DEBUG
     GEngine->AddOnScreenDebugMessage(-1, 15.0f, FColor::Red, "debug msg");
#endif
```

​
Check if actor mesh is of specific type:
​
```cpp
Check if actor mesh is of specific type
for(UActorComponent  i : OtherActor->GetComponentsByClass(UStaticMeshComponent::StaticClass()) {
 
        if(((UStaticMeshComponent*) i)->StaticMesh->GetPathName().Equals("/Game/FirstPerson/Meshes/FirstPersonTemplateCube.FirstPersonTemplateCube"))... // Dont forget continue;
 }
```
​
Don't forget about UGameplayStatics!
​
Vector math support operator overloading.
​
Apply velocity to an actor that `Actor->IsRootComponentMovable && Actor->GetRootComponent()->IsSimulatingPhysics`
`Actor->GetRootPrimitiveComponent()->SetPhysicsLinearVelocity(velocity, false);`
​
The generated header file include should be the last include and the first include needs to be the header file
with the name of your project.
​
In order to add a timer in C++ you need a `FTimerHandle` in order to keep track of it. It is included in `Actor.h`, the syntax is as follows:
​
```cpp
//Declare somewhere - Probably .h file
FTimerHandle TimerHandle;
​
//Somewhere in your .cpp file - NOT IN THE CONSTRUCTOR , BREAKPOINT WILL TRIGGER IF YOU DON'T DO OTHERWISE
GetWorldTimerManager().SetTimer(TimerHandle, this , &Method, DelayBetweenLoops , LoopTheTimer, FirstDelayInSeconds);
```
​
Obviously if `LoopTheTimer` is `false` , `DelayBetweenLoops` is going to be ignored.
​
```cpp
Delay(FirstDelayInSeconds) -> &Method -> If LoopTheTimer == true -> Delay (DelayBetweenLoops) -> &Method -> ... //Till timer is stopped
```
​
You can have timers without a class and method reference:
​
```cpp
GetWorld()->GetTimerManager().SetTimer(TimerHandle, DelayBetweenLoops, LoopTheTimer, FirstDelayInSeconds);
```
​
No use case for `LoopTheTimer == true` :) You just need to check if it is active with `GetWorld()->GetTimerManager().IsTimerActive(TimerHandle)`.
​
How to initialize custom UActorComponents (and maybe UObjects in general) on runtime:
​
```cpp
UActorComponent* comp = ConstructObject<UActorComponent>(UActorComponent::StaticClass(), Owner);
comp->RegisterComponentWithWorld(GetWorld()); //Only for UActorComponents
```
​
How to hotswap custom UActorComponents in runtime:
​
```cpp
//If registered unregister first
//follow instructions on top to initialize :)
```

For multiplayer games: Gamemode is used only on the server. GameState's replicated values suit the needs of game time,
captured points , drop locations etc.

Just use the GameState to track the state of the game (duh!).

A delay of 0 (zero) will cause code to execute next frame.

Don't use default C++ types because their size may vary. Use Unreal's variations instead (e.g. uint32). You can use float though!

Speaking of that, for simplicity reasons, uint8 (Byte) and int32 (Integer) are the only supported int types in blueprints.

Set a rotation of an actor via a forward vector : Use XVector to Rotation.

Get control rotation is better, more accurate and more reliable than getting let's say the rotation of the camera.

You will come across lot's of Public and Private folders: Public folder contains the header files (interface) and Private folder contains the cpp files (implementation) for the most part. You can find some header files in the private folder if it is internal to the module or similar.

The Developer folder contains modules only for usage at editor, or non-game time.
The Runtime folder contains modules for usage at runtime (or all the time).

Editor and Developer stuff don't get shipped on a packaged game.

VInterp to Constant node needs a much bigger value of Interp Speed that the VInterp to node. Beware.

If you are using a canvas, a UMG element's screen space position is based on it's anchor point and possible offsets.

If you are messing around with viewport positions, use Mouse Position Scaled by DPI (from PlayerController). The other one behaves stupidly.

Apparently there is a "OnLand" event. It also shows you the velocity before the fall (Z != 0). Use this so you don't have to manually track stuff to use with the other collision events.

Recording without a capture card can be tricky. Frames get skipped in the editor to prevent lag and other problems, thus , some stuff tend not to work correctly if your computer is on the lower end. Some keystrokes may not get registered, timelines may behave differently and skip numbers etc...

Don't confine yourself to only UE4 related sites. For example , the material(shader) editor has got a custom expression node - in which you can code HLSL (or copy it from other sources like shadertoy). It will then allow you to do cooler effects and even cross-compile it so it runs on different platforms!

Destructibles: Damage Threshold: 1 point to resist a fall of 0.24 units.

Regarding landscape materials: you can't have more than 1 layer "chain" (1 big one or many small weight-blended ones). Seriously, it is going to cause stuff. Instead, use the lerp node to blend alpha, height , angle and then do your paint-over layer(s) chain, using the generated one as a base. 

Whenever you do velocity and movement stuff, always use Linear and Angular damping. Always.

It's considered a good practise (by me, at least) to learn how the engine works, the way-of-doing-things and API via blueprints first due to the error-less nature. After this phase you should probably learn FIRSTLY how to code and SECONDLY how to code inside the engine. It's recommended to code every sample project by yourself.

BlueprintNativeEvent tagged functions only call the C++ _Implementation part whenever you add a call parent node.
If you want to force an event to call the C++ method first, make a method that first calls your method and then calls a BlueprintImplementableEvent which users override.

Learn what Remote Events are.

Add torque units are badly calculated and need huge values for an actual result.

Add force to point can be tricky because it also adds torque automagically.

Whenever you are using Root Motion, you lose the ability to control rotation while the animation is played. In order to fix that, be sure to use the source version of the engine and at CharacterMovementComponent.cpp line ~1828, you need to call `PhysicsRotation` even if `HasAnimRootMotion` is true. (Credits: dogles) 
