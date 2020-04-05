---
description: 'Object & Actor Iterators, Optional Class Scope For Faster Search'
---

# Iterators

### Overview

Dear Community,

In the UE4 engine two of the most powerful tools I use constantly are the Object and the Actor Iterators.

You can use these functions to search for all Run-Time instances of actors and objects, or only specific classes!

**The advantage of using the UE4 iterators is that they are always accurate!**

You dont have to maintain dynamic arrays of actors, and then remember to remove actors when they are destroyed!

The Actor and Object Iterators always give you the real and accurate list of all actors / objects currently still active in your game world

Yay!

#### Include

**\#include "EngineUtils.h"**

**Controller Class**

You dont have to use these functions in the Controller Class,

I was just doing this for the sake of ClientMessage and easy testing on your part :\)

### Object Iterator

```cpp
void AYourControllerClass::PrintAllObjectsNamesAndClasses()
{
    for ( TObjectIterator<UObject> Itr; Itr; ++Itr )
    {
        ClientMessage(Itr->GetName());
        ClientMessage(Itr->GetClass()->GetDesc());
    }
}
```

### Actor Iterator

```cpp
void AYourControllerClass::PrintAllActorsLocations()
{
    //EngineUtils.h
    for (TActorIterator<AActor> ActorItr(GetWorld()); ActorItr; ++ActorItr )
    {
        ClientMessage(ActorItr->GetName());
        ClientMessage(ActorItr->GetActorLocation().ToString());
    }
}
```

### Object Iterator & Actor Iterator Comparison

#### Disadvantage of Object Iterator

Unlike the Actor Iterator, the Object iterator is going to iterate over objects in the Pre-PIE world / the Editor World.

This can lead to unexpected results.

This is not an issue if you are running your game as an independent Game Instance / the editor is closed :\)

#### Critical Advantage of Object Iterator

A critically important advantage of the Object Iterator is that it does not require a UWorld\* Context!

Notice how all the uses of Actor Iterator involve GetWorld\(\)

```cpp
TActorIterator ActorItr<AStaticMeshActor>(GetWorld());
```

If you need to find an object in the game world from a static context, where you cannot obtain the UWorld via some other means,

then the Object Iterator is the way to get the proper context and access the entire living game world!

#### Object Iterator Can Search for AActors

Because AActor extends UObject, the Object Iterator can search for AActors!

But the AActor Iterator cannot search for instances of UObjects that do not extend AActor at some point.

So the Object Iterator can do a search for all UStaticMeshComponents, as well as all ACharacters!

```cpp
TObjectIterator<UStaticMeshComponent> Itr;
```

```cpp
TObjectIterator<ACharacter> Itr;
```

### Specifying Classes & Subclasses To Search For

Perhaps the most powerful feature of the Actor and Object Iterators is the ability to limit the scope of the search to a chosen base class and its subclasses!

This makes the iterator run faster and helps you gather only the data you really want from the game world!

#### Object Iterator, Specific Base Class

```cpp
void AYourControllerClass::PrintAllSkeletalMeshComponentsNames()
{
    for ( TObjectIterator<USkeletalMeshComponent> Itr; Itr; ++Itr )
    {
        ClientMessage(Itr->GetName());
    }
}
```

#### Actor Iterator, Specific Base Class

```cpp
void AYourControllerClass::PrintAllStaticMeshActorsLocations()
{
    //EngineUtils.h
    for (TActorIterator<AStaticMeshActor> ActorItr(GetWorld()); ActorItr; ++ActorItr)
    {
        ClientMessage(ActorItr->GetName());
        ClientMessage(ActorItr->GetActorLocation().ToString());
    }
}
```

### Using a World-Filter with ObjectIterator

ObjectIterator can and will return editor-instance / default object objects that simply should not be edited at runtime!

To filter out objects that you should not be editing at runtime, you can do a world check with an object that you know is part of the correct world \(not the editor world\)!

```cpp
UWorld* YourGameWorld = //set this somehow, from another UObject or pass it in as parameter

for(TObjectIterator<UYourObject> Itr; Itr; ++Itr)
{
   //World Check
   if(Itr->GetWorld() != YourGameWorld)
   {
      continue;
   }
   //now do stuff
}
```

#### In-Engine Example ~ Get All Widgets Of Class

The above is the code structure that I used for my Get All Widgets of Class node, pull request that Epic accepted that is now live in 4.7 !

**Github Link**

[https://github.com/EpicGames/UnrealEngine/pull/569](https://github.com/EpicGames/UnrealEngine/pull/569)

I avoid getting the UMG widget default objects /editor objects by passing in the world using the Blueprint method of setting a WorldContextObject!

Enjoy!

## Authors

Original author: Rama &lt;3

Ported from wiki by Firefly74940

