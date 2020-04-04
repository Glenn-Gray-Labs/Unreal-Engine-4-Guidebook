---
description: Guide on using USTRUCTS by Rama the legend
---

# Structs, USTRUCTS\(\), They're Awesome

### Overview

**Original Author: Rama**

Structs enable you to create custom variable types to organize your data, by relating other c++ or UE4 C++ data types to each other.

The power of structs is extreme organization, as well as ability to have functions for internal data type operations!

#### Technical

Structs enable you to create custom variable types to organize your data, by relating other C++ or UE4 C++ data types to each other. The power of structs is extreme organization as well as the ability to have functions for internal data type operations. '

In UE4, structs should be used for simple data type combining and data management purposes. For complex interactions with the game world, you should make a `UObject` or `AActor` subclass instead.

### Core Syntax

```cpp
//If you want this to appear in BP, make sure to use this instead //USTRUCT(BlueprintType)
USTRUCT() struct FJoyStruct
{
    GENERATED_BODY()

    // Always make USTRUCT variables into UPROPERTY()
    // any non-UPROPERTY() struct vars are not replicated

    // So to simplify your life for later debugging, always use UPROPERTY()
    UPROPERTY()
    int32 SampleInt32;

    //If you want the property to appear in BP, make sure to use this instead
    //UPROPERTY(BlueprintReadOnly)

    UPROPERTY()
    AActor* TargetActor;

    //Set
    void SetInt(const int32 NewValue)
    {
        SampleInt32 = NewValue; 
    }

    //Get
    AActor* GetActor()
    {
        return TargetActor;
    }

    //Check
    bool ActorIsValid() const
    {
        if(!TargetActor)
            return false;

        return TargetActor->IsValidLowLevel();
    }

    //Constructor
    FJoyStruct()
    {
        // Always initialize your USTRUCT variables!
        // exception is if you know the variable type has its own default 
        constructor SampleInt32 = 5;
        TargetActor = nullptr;
    } 
};
```

> **Additional Note Author: DesertEagle\_PWN**  
> The idea of USTRUCTS\(\) is to declare engine data types that are in global scope and can be accessed by other classes/structs/blueprints. Because of this, it is invalid UE4 syntax to declare a struct inside of a class or other struct if using the USTRUCT\(\) macro. Regular structs can still be utilized inside your classes and other structs; however these cannot be replicated natively and will not be available for UE4 reflective debugging or other engine systems such as Blueprints.
>
> **Additional Note Author: Darkgaze**  
> Concerning the variables visibility on the editor: In the example above, if you don't add "EditAnywhere" parameter into UPROPERTY inside the members of the USTRUCT, whey won't show up in the Editor panel. You will see the variable but there will be no way to see/change/unfold the values inside. The class that defines a new UPROPERTY using that struct type should have that parameter too. In case you can't modify the data and you are using blueprints, you should add BlueprintType inside the USTRUCT parenthesis.

### Examples

#### Example 1

You want to relate a float brightness value with a world space location FVector, both of which are interpolated using an Alpha value.

And you want to do this for 100 different game locations simultaneously. And you want to do this process repeatedly over time! You need to store the incremental interpolation values between game events. AActors/UObjects are not involved \(You could just subclass `AActor`/`UObject` and store the data per instance\)

```cpp
USTRUCT()
struct FMyInterpStruct
{
    GENERATED_BODY()

    UPROPERTY()
    float Brightness;

    UPROPERTY()
    float BrightnessGoal; //interping to

    UPROPERTY()
    FVector Location;

    UPROPERTY()
    FVector LocationGoal;

    UPROPERTY()
    float Alpha;


    void InterpInternal()
    {
        Location = FMath::Lerp<FVector>(Location,LocationGoal,Alpha);
        Brightness = FMath::Lerp<float>(Brightness,BrightnessGoal,Alpha);
    }

    //Brightness out is returned, FVector is returned by reference 
    float Interp(const float& NewAlpha, FVector& Out)
    { 
        // value received from rest of your game engine
        Alpha = NewAlpha;

        //Internal data structure management
        InterpInternal();

        //Return Values
        Out = Location;
        return Brightness;
    }

    FMyInterpStruct()
    {
        Brightness = 2;
        BrightnessGoal = 100;
        Alpha = 0;
        Location = FVector::ZeroVector;
        LocationGoal = FVector(0,0,200000);
    } 
};
```

#### Example 2

You want to track information about particle system components that you have spawned into the world through

```cpp
UGameplayStatics::SpawnEmitterAtLocation() // returns a UParticleSystemComponent
```

and you want to track the lifetime of the particle and apply parameter changes from C++. You could write your own class, but if your needs are simple or you do not have project-permissions to make a subclass of `UParticleSystemComponent`, you can just make a `USTRUCT` to relate the various data types!

```cpp
USTRUCT()
struct FParticleStruct
{
    GENERATED_BODY()

    UPROPERTY()
    UParticleSystemComponent* PSCPtr;

    UPROPERTY()
    float LifeTime;


    void SetColor()
    {
        // your code here
    }

    FLinearColor GetCurrentColor() const
    {
        // your code here
    }

    // For GC
    void Destroy()
    {
        PSCPtr = nullptr;
    }

    //Constructor
    FParticleStruct()
    {
        PSCPtr = nullptr;
        LifeTime = -1;
    }
};
```

**Particle Data Tracker**

Now you can have an array of these `USTRUCTS` for each particle that you spawn!

```cpp
// Particle Data Tracking Array
UPROPERTY()
TArray<FParticleStruct> PSCArray;
```

**Garbage Collection**

By marking a `USTRUCT` or `USTRUCT` array as `UPROPERTY()` and marking any UObject / AActor members as `UPROPERTY()`, you are protected from dangling pointer crashes

[link to article](How_To_Prevent_Crashes_Due_To_Dangling_Actor_Pointers)

However you must also clear ustructs you no longer need if they have pointers to `UObjects` if you ever want GC to be able garbage collect those `UObjects`.

### Structs With Struct Member Variables

The struct that wants to use another struct must be defined below the struct it wants to include.

```cpp
USTRUCT()
struct FFlowerStruct
{
    GENERATED_BODY()

    UPROPERTY()
    int32 NumPetals;

    UPROPERTY()
    FLinearColor Color;

    UPROPERTY()
    FVector Scale3D;

    void SetFlowerColor(const FLinearColor& NewColor)
    {
        Color = NewColor;
    }

    FFlowerStruct()
    {
        NumPetals = 5;
        Scale3D = FVector(1,1,1);
        Color = FLinearColor(1,0,0,1);
    }
};

USTRUCT()
struct FIslandStruct
{
    GENERATED_BODY()

    UPROPERTY()
    int32 Type;

    UPROPERTY()
    TArray<FVector> StarLocations;

    UPROPERTY()
    float RainAlpha;

    //Dynamic Array of Flower Custom USTRUCT()
    UPROPERTY() 
    TArray<FFlowerStruct> FlowersOnThisIsland;


    void SetRainAlpha(const float& NewAlpha)
    {
        RainAlpha = NewAlpha;
    }

    int32 GetStarCount() const
    {
        return StarLocations.Num();
    }

    FIslandStruct()
    {
        Type = 0;
        Percent = 1;
    }
};
```

### Struct Assignment

My personal favorite thing about structs is that unlike `UObject` or `AActor` classes, which must be utilized via pointers \(`AActor*`\) you can directly copy the entire contents of a `USTRUCT` to another `USTRUCT` of the same type with a single line of assignment!

```cpp
FFlowerStruct ExistingFlower;

// ... create ExistingFlower here

FFlowerStruct NewFlower;
NewFlower = ExistingFlower;
```

#### Deep Copy

If you have struct members pointing to UObjects or array pointers, you must be careful to copy these members yourself!

```cpp
USTRUCT()
struct FMyStruct
{
   int32* MyIntArray;
};

FMyStruct MyFirstStruct, MySecondStruct;

// Create the integer array on the first struct
MyFirstStruct.MyIntArray = new int32[10];
for( int i = 0; i < 10; ++i )
{
    MyFirstStruct.MyIntArray[i] = i;
}

GEngine->AddOnScreenMessage(-1, 10.f, FColor::Blue, FString::FromInt(MyFirstStruct.MyIntArray[4]));

// Assign the first struct to the second struct, i.e. create a shallow copy
MySecondStruct.MyIntArray[4] = 6;

GEngine->AddOnScreenMessage(-1, 10.f, FColor::Blue, FString::Printf(
    TEXT("%d %d"), MyFirstStruct.MyIntArray[4], MySecondStruct.MyIntArray[4]));
```

On screen the output will be

```text
4
6 6
```

instead of the expected

```text
4
4 6
```

This is because the data stored in `MyStruct::MyIntArray` is not actually stored inside of `MyStruct`. The new keyword creates the data somewhere in RAM and we simply store a pointer there. The address the pointer stores is copied over to `MySecondStruct`, but it still points to the same data. In fact, it would be counterproductive to remove this functionality since there are cases where you want exactly that. Additionally the Unreal Property System does not support non-UObject pointers, which is why `MyIntArray` is not marked with `UPROPERTY()`.

However, copying arrays of integers \(e.g. `int32[10]` instead of `int32*`\) means the data is stored directly inside the struct and as such "deep copied". However, if you store a pointer to a `UObject`, this object is NOT deep copied! Once again only the pointer is copied and the original `UObject` left unchanged. Which is good because otherwise you might manipulate the wrong instance thinking you only had one to begin with leaving the original `UObject` unaffected, thus resembling a very nerve-wrecking and very difficult to track down bug!

### Automatic Make/Break in BP

Marking the `USTRUCT` as `BlueprintType` and adding `EditAnywhere, BlueprintReadWrite, Category = "Your Category"` to `USTRUCT` properties causes UE4 to automatically create Make and Break Blueprint functions, allowing to construct or extract data from the custom `USTRUCT`.

Special thanks to Community member **Iniside** for pointing this out. :\)

```cpp
USTRUCT(BlueprintType)
struct FFlowerStruct
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Flower Struct")
    int32 NumPetals;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Flower Struct")
    FLinearColor Color;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Flower Struct")
    FVector Scale3D;
};
```

[image \(todo\)](CustomUStructMakeBreak.jpg)

### Replication

Remember that only `UPROPERTY(`\) variables of `USTRUCTS()` are considered for replication!

So if your `USTRUCT` is not replicating properly, the first thing you should check is that every member is at least `UPROPERTY()`! The struct does not have be a `BlueprintType`, it just needs `UPROPERTY()` above all properties that you want replicated.

### Other notes

In case you are looking for `GENERATED_USTRUCT_BODY`, in 4.11+, `GENERATED_BODY()` should be used instead.

### Related Links

[UStruct data member memory management]()

### Thank You Epic for USTRUCTS\(\)

I love `USTRUCTS()`, thank you Epic!

## Authors

Original author: Rama &lt;3  
Captured from the epic wiki via the Wayback Machine. Reformatted by Maldonacho

