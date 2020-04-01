---
description: >-
  A quick reference around Unreal's UFUNCTION macro in C++ and available
  keywords.
---

# The UFUNCTION Macro

## Keywords

These keywords are also valid for the `UDELEGATE` macro.

### **BlueprintAuthorityOnly**

This function will only execute from Blueprint code if running on a machine with network authority \(a server, dedicated server, or single-player game\).

* Useful for visually marking methods in Blueprint for designers.
* Will still execute on non-network authority clients when called from C++.

```cpp
UFUNCTION(BlueprintCallable, BlueprintAuthorityOnly)
void SpawnProjectile();
```

### **BlueprintCallable**

 This function can be executed in a Blueprint and will appear in Blueprint tooling.

* Using the `const` C++ keyword on the related method will remove the execution pin, making this a pure Blueprint function node.

```cpp
UFUNCTION(BlueprintCallable)
void SetValue(float InValue);
```

### **BlueprintCosmetic**

This function is cosmetic and will not run on dedicated servers.

* Will still execute on network authority servers when called from C++.

```cpp
UFUNCTION(BlueprintImplementableEvent, BlueprintCosmetic)
void PlayHitEffects();
```

### BlueprintGetter

 This function will be used as the accessor for a Blueprint-exposed property. This specifier implies `BlueprintPure` and `BlueprintCallable`.

> More information needed about this keyword.

### BlueprintInternalUseOnly

Indicates that the function should not be exposed to the end user.

> More information needed about this keyword.

### BlueprintImplementableEvent

This function is designed to be overridden \(implemented\) in Blueprint.

* Do not provide a body for this function; the auto-generated code will include a thunk that calls `ProcessEvent` to execute the overridden body.
* You'll need to add the `BlueprintCallable` keyword if you want to call this function from Blueprint, otherwise it's only callable via C++.

```cpp
UFUNCTION(BlueprintImplementableEvent)
void OnSomethingHappened();
```

### BlueprintNativeEvent

This function is designed to be overridden in Blueprint, but also has a native \(C++\) implementation.

* To create a native implementation of the function, you'll need to define a method named `[FunctionName]_Implementation` instead of just the function name. This is due to how the auto-generated code will include a thunk that calls the implementation method when necessary.
* You'll need to add the `BlueprintCallable` keyword if you want to call this function from Blueprint, otherwise it's only callable via C++.

```cpp
// Example.h
UFUNCTION(BlueprintNativeEvent)
void DoSomething();

// Example.cpp
void Example::DoSomething_Implementation() {
    // Your code here
}
```

### **BlueprintPure**

The function does not affect the owning object in any way and can be executed in a Blueprint.

* It's effectively the same as marking a method as `BlueprintCallable` with the `const` C++ keyword.
* These functions must have a return type.
* It is not required, but generally recommended that the function be marked `const`.

```cpp
UFUNCTION(BlueprintPure)
float GetValue() const;
```

### **BlueprintSetter**

 This function will be used as the mutator for a Blueprint-exposed property. This specifier implies `BlueprintCallable`.

> More information needed about this keyword.

### CallInEditor

This function can be called in the editor on selected instances via a button in the Details panel.

> More information needed about this keyword.

### **Category**

Specifies the category of the function when displayed in Blueprint editing tools. 

* You can define nested categories using the `|` operator.
* Quotes are only required when adding spaces or the `|` operator.

```cpp
UFUNCTION(BlueprintCallable, Category="Weapon|Gun")
void Fire();
```

### **Client**

 The function is only executed on the client that owns the Object on which the function is called. See [Unreal's documentation on RPCs](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/RPCs/index.html) for more information.

* Declares an additional function named the same as the main function, but with `_Implementation` added to the end. The auto-generated code will call the `_Implementation` method when necessary.
* Owning client is the object with `ENetRole` of `AutonomousProxy`.
* RPC functions should not have a return value.
* RPC functions are unreliable by default.

```cpp
UFUNCTION(Client)
void ReportHit(float Damage, FVector Direction);
```

### **Custom Thunk**

The `UnrealHeaderTool` code generator will not produce a thunk for this function; it is up to the user to provide one.

> More information needed about this keyword.

### **Exec**

This function is executable from the command line. For more information, [check out this wiki article about the Exec Functions](../wiki-archives/common-pitfalls/exec-functions.md).

```cpp
UFUNCTION(Exec)
void GodMode(bool bEnabled);
```

### NetMulticast

 The function is executed both locally on the server, and replicated to all clients, regardless of the Actor's `NetOwner`. See [Unreal's documentation on RPCs](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/RPCs/index.html) for more information.

* Declares an additional function named the same as the main function, but with `_Implementation` added to the end. The auto-generated code will call the `_Implementation` method when necessary.
* Multicast RPCs behave differently when called by the network authority \(server\) or client:
  * If they are called from the server, the server will execute them locally as well as execute them on all currently connected clients.
  * If they are called from clients, they will only execute locally, and will not execute on the server.
* Multicast functions are throttled and will not replicate more than twice in a given Actor's network update period.
* RPC functions should not have a return value.
* RPC functions are unreliable by default.

```cpp
UFUNCTION(NetMulticast)
void BroadcastGameplayEvent(EGameplayEventType EventType);
```

### **Reliable**

The function is replicated over the network, and is guaranteed to arrive regardless of bandwidth or network errors. Only valid when used in conjunction with the `Client` or `Server` keywords.

```cpp
UFUNCTION(Client, Reliable)
void SendPrivateMessage(FString Text);
```

### **SealedEvent**

 This function cannot be overridden in sub-classes. The `SealedEvent` keyword can only be used for events. For non-event functions, declare them as `static` or `final` to seal them.

```cpp
UFUNCTION(BlueprintNativeEvent, SealedEvent)
void DoSomething();
```

### **ServiceRequest**

This function is an RPC \(Remote Procedure Call\) service request**.**

> More information needed about this keyword.

### ServiceResponse

This function is an RPC service response.

> More information needed about this keyword.

### **Server**

 The function is only executed on the server. See [Unreal's documentation on RPCs](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/RPCs/index.html) for more information.

* Declares an additional function named the same as the main function, but with `_Implementation` added to the end, which is where code should be written. The auto-generated code will call the `_Implementation` method when necessary.
* The `WithValidation` keyword must be used with the `Server` keyword.
* RPC functions should not have a return value.
* RPC functions are unreliable by default.

```cpp
UFUNCTION(Server, WithValidation)
void ServerSendInputValue(float Value);
```

### **Unreliable**

The function is replicated over the network but can fail due to bandwidth limitations or network errors. 

* Only valid when used in conjunction with `Client` or `Server`.

```cpp
UFUNCTION(Client, Unreliable)
void SendObjectLocation(FVector Location);
```

### **WithValidation**

Declares an additional function named the same as the main function, but with `_Validate` added to the end. This function takes the same parameters, and returns a `bool` to indicate whether or not the call to the main function should proceed.

* Required for the `Server` keyword. This was done to encourage secure server RPC functions, and to make it as easy as possible for someone to add code to check each and every parameter to be valid against all the known input constraints.

```cpp
// MyCharacter.h
UFUNCTION(Server, Reliable, WithValidation)
void ServerSetSprint(bool bSprinting);

// MyCharacter.cpp
void AMyCharacter::ServerSetSprint_Implementation(bool bSprinting) {
  SetSprint(bSprinting);
}

bool AMyCharacter::ServerSetSprint_Validate(bool bSprinting) {
  return true;
}
```

\*\*\*\*

## Additional Resources

* [Unreal Official Documentation: UFunctions](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Functions/index.html)
* [Tom Looman: UFUNCTION Keywords Explained](https://www.tomlooman.com/ue4-ufunction-keywords-explained/)

