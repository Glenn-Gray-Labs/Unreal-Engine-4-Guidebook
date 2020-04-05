---
description: This wiki article was written by Rama; Converted by jfaw.
---

# Logs: Printing Messages to Yourself during Runtime

## Overview

Dear Community,

Logs are essential for giving yourself feedback as to whether

* Your new functions are even being called
* What data your algorithm is using during runtime
* Reporting errors to yourself and the end user / debugging team
* Imposing a fatal error to stop runtime execution in special circumstances

This page describes how to use the **Unreal output log**.

Other options are also discussed at the bottom of the page.

## Accessing Logs

### In-Game

To see logs you must run your game with `-Log` \(you must create a shortcut to the Editor executable and add `-Log` to the end\).

or use console command "showlog" in your game.

### Within Editor \(Play-In-Editor\)

Log messages are sent to the 'Output' log which is accessible via _Window -&gt; Developer Tools -&gt; Output Log_.

If you are using the Editor and PIE, logging should be enabled by default due to the presence of `GameCommandLine=-log` in your Engine INI file. If no logging is visible, add the `-Log` command line option as per the instructions for In-Game logging above.

### Quick Usage

```cpp
UE_LOG(LogTemp, Warning, TEXT("Your message"));
```

This way you can log without the need of creating a custom category. Doing so will keep everything clean and sorted though.

### Log Verbosity Levels

Log verbosity levels are used to more easily control what is being printed, allowing you to keep even the most detailed log statements in your code without having them spam output when you don't want them to. Each log statement declares which log it belongs to and it's verbosity level. Verbosity level is controlled on a per-log basis.

Each log's verbosity is controlled by four things: 1. Compile-time verbosity 2. Default verbosity 3. `.ini` verbosity 4. Runtime-verbosity.

If a log statement is more verbose than it's log's compile time verbosity it won't even be compiled into the game code. From there the log's level is set to the default verbosity, which can then be overridden in the Engine.ini file, either of those can then be overridden from the command line \(the runtime verbosity\). Once the game \(or editor\) is running it may not be possible to change a log category's verbosity \(I am not sure, someone who knows please correct this\).

Here are the verbosity levels available to use:

* **Fatal** Fatal level logs are always printed to console and log files and crashes even if logging is disabled.
* **Error** Error level logs are printed to console and log files. These appear red by default.
* **Warning** Warning level logs are printed to console and log files. These appear yellow by default.
* **Display** Display level logs are printed to console and log files.
* **Log** Log level logs are printed to log files but not to the in-game console. They can still be viewed in editor as they appear via the Output Log window.
* **Verbose** Verbose level logs are printed to log files but not the in-game console. This is usually used for detailed logging and debugging.
* **VeryVerbose** VeryVerbose level logs are printed to log files but not the in-game console. This is usually used for very detailed logging that would otherwise spam output.

For the `CompileTimeVerbosity` parameter of `DECLARE_LOG_CATEGORY_EXTERN` it is also valid to use `All` \(functionally the same as using `VeryVerbose`\) or `NoLogging` \(functionally the same as using `Fatal`\).

## Setting Up Your Own Log Category

### Log Category Macros

The macros `DECLARE_LOG_CATEGORY_EXTERN` and `DEFINE_LOG_CATEGORY` go in _YourGame.h_ and _YourGame.cpp_ respectively.

The macro to declare a log category has three parameters. Each declared log category should have a corresponding defined log category in a cpp.

```cpp
DECLARE_LOG_CATEGORY_EXTERN(CategoryName, DefaultVerbosity, CompileTimeVerbosity);
```

`CategoryName` is simply the name for the new category you are defining.

`DefaultVerbosity` is the verbosity level used when one is not specified in the ini files or on the command line. Anything more verbose than this will not be logged.

`CompileTimeVerbosity` is the maximum verbosity to compile in the code. Anything more verbose than this will not be compiled.

The macro to define a log category takes only the name of the category.

```cpp
DEFINE_LOG_CATEGORY(CategoryName);
```

### Usage Example

You can have different log categories for different aspects of your game!

This gives you additional info, because `UE_LOG` prints out which log category is displaying a message.

Here is an example of where the different log levels start to become useful.

Say you're often having trouble with a certain system in your game. In debugging you might want very detailed logs, but when you've finished debugging for now you know you might need those detailed logs later on, but they're spamming the output. What do you do? Use different log levels.

#### MyGame.H

```cpp
//General Log
DECLARE_LOG_CATEGORY_EXTERN(LogMyGame, Log, All);

//Logging during game startup
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameInit, Log, All);

//Logging for your AI system
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameAI, Log, All);

//Logging for a that troublesome system
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameSomeSystem, Log, All);

//Logging for Critical Errors that must always be addressed
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameCriticalErrors, Log, All);
```

#### MyGame.CPP

```cpp
#include "MyGame.h"

//General Log
DEFINE_LOG_CATEGORY(LogMyGame);

//Logging during game startup
DEFINE_LOG_CATEGORY(LogMyGameInit);

//Logging for your AI system
DEFINE_LOG_CATEGORY(LogMyGameAI);

//Logging for some system
DEFINE_LOG_CATEGORY(LogMyGameSomeSystem);

//Logging for Critical Errors that must always be addressed
DEFINE_LOG_CATEGORY(LogMyGameCriticalErrors);
```

#### MyClass.CPP

```cpp
//...
void UMyClass::FireWeapon()
{
    UE_LOG(LogMyGameSomeSystem, Verbose, TEXT("UMyClass %s entering FireWeapon()"), *GetNameSafe(this));
    //Logic
    UE_LOG(LogMyGameSomeSystem, Verbose, TEXT("UMyClass %s Attempting to fire."), *GetNameSafe(this));
    if (CheckSomething())
    {
        UE_LOG(LogMyGameSomeSystem, Log, TEXT("UMyClass %s is firing their weapon with charge of %f"), *GetNameSafe(this), GetCharge());
        //Firing logic
    }
    else
    {
        UE_LOG(LogMyGameSomeSystem, Error, TEXT("UMyClass %s CheckSomething() returned false during FireWeapon(), this is bad!"), *GetNameSafe(this));
        //Fail with grace
    }
    //More code!
    UE_LOG(LogMyGameSomeSystem, Verbose, TEXT("UMyClass %s leaving FireWeapon()"), *GetNameSafe(this));
}

void UMyClass::Tick(float DeltaTime)
{
    UE_LOG(LogMyGameSomeSystem, VeryVerbose, TEXT("UMyClass %s's charge is %f"), *GetNameSafe(this), GetCharge());
    if (something)
    {
        UE_LOG(LogMyGameSomeSystem, VeryVerbose, TEXT("Idk"));
    }
    if (somethingelse)
    {
        UE_LOG(LogMyGameSomeSystem, VeryVerbose, TEXT("Stuff"));
    }
}
//...
```

When you're not working on this system all these log statements would absolutely flood your output, and even when you are working on it you might not want the level of detail that is putting out multiple logs per tick.

By using log levels you can simply change the verbosity in the category's declaration, in the ini files, or on the command line to hide/reveal different layers of log statements as you need them. Ex:

```cpp
//All log statements are shown.
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameSomeSystem, Log, All);

//VeryVerbose statements won't be shown.
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameSomeSystem, Verbose, All);

//Neither VeryVerbose nor Verbose statements will be shown.
DECLARE_LOG_CATEGORY_EXTERN(LogMyGameSomeSystem, VeryVerbose, All);
```

The log categories used by Unreal Engine use different log levels, but by default have a higher `CompileTimeVerbosity`. In debugging interaction with Unreal code it might be helpful to turn up the verbosity of Unreal code in _DefaultEngine.ini_ under `[Core.Log]` by adding an entry like `LogOnline=Verbose`.

## Log Formatting

#### Log Message

```cpp
//"This is a message to yourself during runtime!"
UE_LOG(YourLog,Warning,TEXT("This is a message to yourself during runtime!"));
```

#### Log an FString

* `%s` strings are wanted as `TCHAR*` by `Log`, so use `*FString()`

```cpp
//"MyCharacter's Name is %s"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's Name is %s"), *MyCharacter->GetName() );
```

#### Log an Bool

```cpp
//"MyCharacter's Bool is %s"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's Bool is %s"), (MyCharacter->MyBool ? TEXT("True") : TEXT("False")));
```

#### Log an Int

```cpp
//"MyCharacter's Health is %d"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's Health is %d"), MyCharacter->Health );
```

#### Log a Float

```cpp
//"MyCharacter's Health is %f"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's Health is %f"), MyCharacter->Health );
```

#### Log an FVector

```cpp
//"MyCharacter's Location is %s"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's Location is %s"), 
    *MyCharacter->GetActorLocation().ToString());
```

#### Log an FName

```cpp
//"MyCharacter's FName is %s"
UE_LOG(YourLog,Warning,TEXT("MyCharacter's FName is %s"), 
    *MyCharacter->GetFName().ToString());
```

#### Log an FString,Int,Float

```cpp
//"%s has health %d, which is %f percent of total health"
UE_LOG(YourLog,Warning,TEXT("%s has health %d, which is %f percent of total health"),
    *MyCharacter->GetName(), MyCharacter->Health, MyCharacter->HealthPercent);
```

## Log Coloring

#### Log: Grey

```cpp
//"this is Grey Text"
UE_LOG(YourLog,Log,TEXT("This is grey text!"));
```

#### Warning: Yellow

```cpp
//"this is Yellow Text"
UE_LOG(YourLog,Warning,TEXT("This is yellow text!"));
```

#### Error: Red

```cpp
//"This is Red Text"
UE_LOG(YourLog,Error,TEXT("This is red text!"));
```

#### Fatal: Crash for Advanced Runtime Protection

You can throw a fatal error yourself if you want to make sure that certain code never runs.

I have used this myself to help protect against algorithm cases that I wanted to make sure never occurred again.

It's actually really useful!

But it does look like a crash, and so if you use this, dont be worried, just look at the crash call stack :\)

* Again this is an advanced case that crashes the program, **use only for extremely important circumstances**.

```cpp
//some complicated algorithm
if(some fringe case that you want to tell yourself if the runtime execution ever reaches this point)
{
    //"This fringe case was reached! Debug this!"
    UE_LOG(YourLog,Fatal,TEXT("This fringe case was reached! Debug this!"));
}
```

## Quick tip print

This a trick for easy print debug, you can use this MACRO at the begin of your cpp

```cpp
#define print(text) if (GEngine) GEngine->AddOnScreenDebugMessage(-1, 1.5, FColor::White,text)
```

then you can use a regular lovely `print();` inside to all.

To prevent your screen from being flooded, you can change the first parameter, key, to a positive number. Any message printed with that key will remove any other messages on screen with the same key. This is great for things you want to log frequently.

## Other Options for Debugging

### Logging message to the screen

For the times when you want to just display the message on the screen, you can also do:

```cpp
 #include <EngineGlobals.h>
 #include <Runtime/Engine/Classes/Engine/Engine.h>
 // ...
 GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("This is an on screen message!"));
 GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, FString::Printf(TEXT("Some variable values: x: %f, y: %f"), x, y));
```

To prevent your screen from being flooded, you can change the first parameter, key, to a positive number. Any message printed with that key will remove any other messages on screen with the same key. This is great for things you want to log frequently.

### Logging message to the ~ Client Console

Pressing the `~` key in Unreal brings up the client console.

If you use the `PlayerController` class you can print a message to this console, which has the advantage of being a completely different logging space which does not require tabbing out of the game to view easily

```cpp
 PC->ClientMessage("Your Message");
```

* [Answerhub post on using `ClientMessage`](https://answers.unrealengine.com/questions/81662/vshow-function.html):
* [Forum Post on post messages to the client console](https://forums.unrealengine.com/showthread.php?33367-Log-to-Console%7Csend)

## Log conventions \(in the console, ini files, or environment variables\)

* \[cat\] = a category for the command to operate on, or 'global' for all categories.
* \[level\] = verbosity level, one of: none, error, warning, display, log, verbose, all, default

At boot time, compiled in default is overridden by ini files setting, which is overridden by command line

## Log console command usage

* `Log list` - list all log categories
* `Log list [string]` - list all log categories containing a substring
* `Log reset` - reset all log categories to their boot-time default
* `Log [cat]` - toggle the display of the category \[cat\]
* `Log [cat] off` - disable display of the category \[cat\]
* `Log [cat] on` - resume display of the category \[cat\]
* `Log [cat] [level]` - set the verbosity level of the category \[cat\]
* `Log [cat] break` - toggle the debug break on display of the category \[cat\]

## Log command line

* `-LogCmds=\"[arguments],[arguments]...\"` - applies a list of console commands at boot time
* `-LogCmds=\"foo verbose, bar off\"` - turns on the foo category and turns off the bar category

## Environment variables

Any command line option can be set via the environment variable **UE-CmdLineArgs**

`set UE-CmdLineArgs=\"-LogCmds=foo verbose breakon, bar off\"`

## Config file

In _DefaultEngine.ini_ or _Engine.ini_:

```text
[Core.Log]
global=[default verbosity for things not listed later]
[cat]=[level]
foo=verbose break
```

♥ -Rama

