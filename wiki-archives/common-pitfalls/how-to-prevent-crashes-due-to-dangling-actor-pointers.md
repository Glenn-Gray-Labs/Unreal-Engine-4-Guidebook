# How To Prevent Crashes Due To Dangling Actor Pointers

While working on Abatron, an RTS/FPS hybrid game with tons of character units to keep track of, I created a lot of arrays of Actors:

```cpp
TArray<AActor*> UnitArray;
```

During multiplayer testing especially, stale / dangling AActor pointers were causing a lot of crashes!

The problem with stale pointers is that just checking ActorPtr != nullptr is not enough, a stale pointer will return true but wont actually still be pointing to a valid AActor, which is what causes the crash.

### UPROPERTY\(\) UObjects Clear References Properly

A less-advertised feature of UObject pointers that are made UPROPERTY\(\) is that they are properly updated to NULL when the object is destroyed, unlike raw pointers like I was using above.

Automatic Updating of UObject References [https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Objects/Optimizations/index.html\#automaticupdatingofreferences](https://docs.unrealengine.com/latest/INT/Programming/UnrealArchitecture/Objects/Optimizations/index.html#automaticupdatingofreferences)

So the **simple solution** if you are having issues with dangling / stale actor pointers is to make sure all AActor pointers are marked with UPROPERTY\(\).

```cpp
UPROPERTY() //<~~~ That's it! This now makes the pointers much more stable! -Rama
TArray<AActor*> UnitArray;
```

### TWeakObjectPtr

For UObjects especially, having lots of UPROPERTY\(\) references to them can prevent them from getting garbage collected properly. For this situation you can use TWeakObjectPtr which will still give you additional validity option using IsValid\(\) but will not prevent GC from running.

### Conclusion

If you are encountering AActor\* pointers that are going stale and crashing your game, make sure they are marked with UPROPERTY\(\) and you will be taking advantage of a rather essential feature of UObjects in UE4, which is that all UPROPERTY\(\) references get updated to NULL when a UObject is destroyed.

Have fun today!

Rama

