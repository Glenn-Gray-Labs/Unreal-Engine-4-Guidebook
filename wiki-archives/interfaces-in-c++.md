# Interfaces in C++

> This article is still in the process of being cleaned up, but it has been captured for preservation.

## Overview

Original Author Rama

Updated & Expanded for 4.11 [HuntaKiller](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=User:HuntaKiller&action=edit&redlink=1) Thank you for all that you contribute to the community Rama!

[Rama](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=User:Rama) : You're welcome and thank you for this lovely addition!

Dear Community,

Here's a tutorial on using **UE4 C++ Interfaces in 4.11+**

Interfaces allow different objects to share common functions, but allow objects to handle that function differently if it needs to. Any classes that use an interface must implement the functions that are associated with that interface.

This gives you a lot of power over your game actors, allowing you to trigger events both in C++ and in blueprints that your game actors can handle differently.

For example, the interface implemented in this tutorial enables you to have an interface like TimeBasedBehaviour, which has a function ReactToHighNoon, and have a bunch of actors respond to this event differently, each with their own behaviour.

Flower actors that implement this interface could override the ReactToHighNoon method to open blossoms completely Frog actors implementing it could override ReactToHighNoon to hide under rocks, for example

You can then have an event, SunReachedHighNoon that is triggered anywhere \(such as the level blueprint, in an actor, or a static blueprint library\) which can take any actor, check if it implements the interface, and if it does it can call any of the functions of that interface and the actor will act according to how that specific actor has the behaviours defined.

This means you can trigger events anywhere and as long as you have a pointer to your actor, you can ask it to do specific things without needing to know its types because you can **easily determine whether any given actor has an interface or not by casting an actor to that interface**. If the cast succeeds then the actor does implement the given interface, and you can call functions using that interface.

We will implement two interface functions: one which forces you to implement default C++ behaviour on any class which uses the interface, a **BlueprintNativeEvent** called ReactToHighNoon\(\), and one **BlueprintImplementableEvent** which does not force you to define default C++ behaviour, called ReactToMidnight\(\).

### Creating The Interface

The following is an example implementation of a ReactsToTimeOfDay interface.

When following this tutorial and creating your interface, you'd replace ReactToHighNoon\(\) with your function you want to force default behaviour, and ReactToMidnight\(\) with your function that has no default behaviour.

\(If you wish the function to be treated as an event, then it must return void. If you wish the function to be able to be overridden in the BP editor, then it must have a non-void return type. Replace the return type of the function with a string or void if you want to perform a simplistic test. The reasoning is discussed further below in the Critical To Note section\)

#### ReactsToTimeOfDay.h

// Copyright 1998-2013 Epic Games, Inc. All Rights Reserved.

1. pragma once
2. include "ReactsToTimeOfDay.generated.h"

/\* must have BlueprintType as a specifier to have this interface exposed to blueprints

```text
  with this line you can easily add this interface to any blueprint class */
```

UINTERFACE\(BlueprintType\) class MYPROJECT\_API UReactsToTimeOfDay : public UInterface { GENERATED\_UINTERFACE\_BODY\(\) };

class MYPROJECT\_API IReactsToTimeOfDay { GENERATED\_IINTERFACE\_BODY\(\)

public: //classes using this interface must implement ReactToHighNoon UFUNCTION\(BlueprintNativeEvent, BlueprintCallable, Category = "MyCategory"\) bool ReactToHighNoon\(\);

 //classes using this interface may implement ReactToMidnight UFUNCTION\(BlueprintImplementableEvent, BlueprintCallable, Category = "MyCategory"\) bool ReactToMidnight\(\);

};&lt;/syntaxhighlight&gt;

Note checked for 4.18: GENERATED\_UINTERFACE\_BODY\(\) and GENERATED\_IINTERFACE\_BODY\(\), can be now changed to GENERATED\_BODY\(\), which is an updated version of those two that works for structs, etc, but errors could be a little confusing if you get compile errors since there's no way to differentiate. You could create an automatic interface to see how it looks now using Create C++ Class context menu on the content editor and choosing Interface type.

#### ReactsToTimeOfDay.cpp

&lt;syntaxhighlight lang="cpp"&gt;// Copyright 1998-2013 Epic Games, Inc. All Rights Reserved.

1. include "MyProject.h"
2. include "ReactsToTimeOfDay.h"

UReactsToTimeOfDay::UReactsToTimeOfDay\(const class FObjectInitializer& ObjectInitializer\) : Super\(ObjectInitializer\) {

} &lt;/syntaxhighlight&gt;

### Using An Interface With C++ Classes

You have to use **multiple inheritance**, and inherit from the IReactsToTimeOfDay class we created.

The first inherited class will be the base class of your actor, anything you want, a ASkeletalMeshActor is used here as an example.

#### Flower.h

&lt;syntaxhighlight lang="cpp"&gt; //..other includes may appear here depending on your class

1. include "ReactsToTimeOfDay.h"
2. include "ASkeletalMeshActor.generated.h"

UCLASS\(\) class AFlower : public ASkeletalMeshActor, public IReactsToTimeOfDay { GENERATED\_BODY\(\)

public: /\* ... other AFlower properties and functions declared ... \*/

 UFUNCTION\(BlueprintCallable, BlueprintNativeEvent, Category = "MyCategory"\) bool ReactToHighNoon\(\); virtual bool ReactToHighNoon\_Implementation\(\) override;

  
 };&lt;/syntaxhighlight&gt;

virtual bool ReactToHighNoon\_Implementation\(\) override; This line tells your class that it has a function of this name and signature to inherit from the interface, which is how calls to the interface functions are able to interact with this class.

UFUNCTION\(BlueprintCallable, BlueprintNativeEvent, Category = "MyCategory"\) bool ReactToHighNoon\(\); This tells your class that you can both call and override this function in blueprints. You need this part as well if you want to be able to override C++ functionality within BP, as BlueprintNativeEvents are intended to be used.

Notice that ReactToMidnight\(\), the BlueprintImplementableEvent, is not defined here. A BlueprintImplementableEvent is declared \(its existance\) in our interface, but defined \(its behaviour\) in blueprints only.

#### Flower.cpp

&lt;syntaxhighlight lang="cpp"&gt; //other flower.cpp code

bool AFlower::ReactToHighNoon\_Implementation\(\) { //Default behaviour for how flower would react at noon //OpenPetals\(\); //AcceptBugs\(\); //...

 return true;

} &lt;/syntaxhighlight&gt;

Any number of classes and subclasses can implement this interface using this format

#### Frog.h

&lt;syntaxhighlight lang="cpp"&gt; //..other includes may appear here depending on your class

1. include "ReactsToTimeOfDay.h"
2. include "AFrog.generated.h"

UCLASS\(\) class AFrog : public ACharacter, public IReactsToTimeOfDay { GENERATED\_BODY\(\) /\* ... other AFrog properties and functions declared ... \*/

 UFUNCTION\(BlueprintCallable, BlueprintNativeEvent, Category = "MyCategory"\) bool ReactToHighNoon\(\); virtual bool ReactToHighNoon\_Implementation\(\) override;

};&lt;/syntaxhighlight&gt;

#### Frog.cpp

&lt;syntaxhighlight lang="cpp"&gt; //other Frog code

bool AFrog::ReactToHighNoon\_Implementation\(\) { //Default behaviour for how a frog would react at noon //GoSwim\(\); //...

 return true; }

&lt;/syntaxhighlight&gt;

#### Determining If a Given Actor Has The Interface

To determine if an actor implements an interface in C++, simply cast your class to the interface, if it returns NULL then the object is not using it. If it is successful, you can use that pointer cast to the interface to call your function, which will execute from the proper class.

&lt;syntaxhighlight lang="cpp"&gt; //Example: somewhere else in code we are trying to see if our object reacts to time of day

//Some pointer is defined to any class inheriting from UObject UObject\* pointerToAnyUObject;

//....

  
 IReactsToTimeOfDay\* TheInterface = Cast&lt;IReactsToTimeOfDay&gt;\(pointerToAnyUObject\); if \(TheInterface\) { //Don't call your functions directly, use the 'Execute\_' prefix //the Execute\_ReactToHighNoon and Execute\_ReactToMidnight are generated on compile //you may need to compile before these functions will appear TheInterface-&gt;Execute\_ReactToHighNoon \(pointerToAnyUObject\); TheInterface-&gt;Execute\_ReactToMidnight \(pointerToAnyUObject\); }

  
 //end of code segment&lt;/syntaxhighlight&gt;

  
 **Critical To Note**

*  Whenever calling your interface functions in C++, never call the direct functions, always use the one with the Execute\_ prefix
*  Although it might seem, that you function must return a value to be properly implemented, this is not true. If your interface doesn't return a value UE4 treats it as an event. At first glance this might seem as an error, but this is not the case. You just have to create the implementation details in the event graph instead of overriding. You still can call the function normally via function, or interface call. --[Ruhrpottpatiot](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=User:Ruhrpottpatiot&action=edit&redlink=1) \([talk](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=User_talk:Ruhrpottpatiot&action=edit&redlink=1)\) 21:00, 27 December 2017 \(UTC\)
*  To determine if an actor implements an interface in both C++ and Blueprints use

&lt;syntaxhighlight lang="cpp"&gt; if \(pointerToAnyUObject-&gt;GetClass\(\)-&gt;ImplementsInterface\(UReactsToTimeOfDay::StaticClass\(\)\)\)

```text
   IReactsToTimeOfDay::Execute_ReactToHighNoon(pointerToAnyUObject);
```

&lt;/syntaxhighlight&gt;

#### The Magic Interfaces

TheInterface-&gt;Execute\_ReactToHighNoon\(\)

From the above code you can see that the function is being called off of the interface, you never even need to know what type of object you're dealing with, just whether it supports the interface you need.

It produces different results depending on the actual class it is, calling the overridden function. This is called polymorphism.

### Overriding Behaviour In Blueprints

Once this is all implemented, the classes that you have set up with the interface in C++ will have its interface functions appear with the blueprint's variables and other functions.

[![InterfaceBP1.png](https://web.archive.org/web/20181002150134im_/https://d26ilriwvtzlb.cloudfront.net/7/7b/InterfaceBP1.png)](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=File:InterfaceBP1.png)

[![InterfaceBP2.png](https://web.archive.org/web/20181002150134im_/https://d26ilriwvtzlb.cloudfront.net/6/6a/InterfaceBP2.png)](https://web.archive.org/web/20181002150134/https://wiki.unrealengine.com/index.php?title=File:InterfaceBP2.png)

  
 Again, your function **must** have a return value for it to appear in this list, otherwise it is considered an event and cannot be overridden as a function. You can however use the event from the interface in blueprint's event graph and override it that way.

### Summary

You can trigger global events that only certain actors will respond to each actor can respond to an event in their own unique way. 

While it's a little bit more complicated of a setup it helps keeping the code very simple and is much more performance friendly than casting to multiple different types of classes!

Rama  
 HuntaKiller  
 Darkgaze

