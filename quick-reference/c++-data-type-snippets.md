---
description: >-
  This page contains several code snippets for quickly creating C++ data types
  that can be used with Blueprints. Use this as reference or as a copy + paste
  resource as needed.
---

# C++ Data Type Snippets

## Interfaces

> For more information about interfaces in Unreal, [check out this wiki article](../wiki-archives/macros-and-data-types/interfaces-in-c++.md).

You'll need to create or add the following code snippet to a header file. Be sure to replace `FILENAME` with the name of the file, replace `MYPROJECT_API` with your project's `_API` identifier, `Example` with the names of your interface/class, and replace the methods as necessary.

```cpp
#pragma once

#include "FILENAME.generated.h"

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

