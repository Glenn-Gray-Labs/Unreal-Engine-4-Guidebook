---
description: >-
  This page contains several code snippets for quickly creating C++ data types
  that can be used with Blueprints. Use this as reference or as a copy + paste
  resource as needed.
---

# C++ Data Type Snippets

## Interfaces

For more information about interfaces in Unreal, [check out this wiki article](../wiki-archives/macros-and-data-types/interfaces-in-c++.md).

* You need to define two classes: `U<Name>` and `I<Name>`. The second class is what your C++ code will extend to implement the interface.

```cpp
UINTERFACE(BlueprintType)
class MYPROJECT_API UExample : public UInterface
{
  GENERATED_BODY()
};

class MYPROJECT_API IExample
{
  GENERATED_BODY()

public:

  UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
  bool NativeEventExampleMethod();

  UFUNCTION(BlueprintImplementableEvent, BlueprintCallable)
  bool BlueprintEventExampleMethod();

};
```

## Structs

For more information about structs in Unreal, [check out this wiki article](../wiki-archives/macros-and-data-types/structs-ustructs-theyre-awesome.md).

* To access your struct from Blueprint, make sure to add the `BlueprintType` keyword to the `USTRUCT` macro.
* Structs must have a default constructor.
* It's Unreal coding standard to prefix your structs with a capital `F`.
* You cannot use the `UFUNCTION` macro with methods on structs.

```cpp
USTRUCT(BlueprintType)
struct MYPROJECT_API FExample
{
    GENERATED_BODY()
    
public:

    UPROPERTY(BlueprintReadOnly)
    int32 SomeValue;

};
```

