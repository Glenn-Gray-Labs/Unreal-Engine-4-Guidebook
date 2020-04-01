---
description: >-
  This guide shows you how to package and compile your game as a standalone
  dedicated server for both Windows and Linux.
---

# Standalone Dedicated Server

## Standalone Dedicated Server

_This is currently only possible using an engine compiled from source. It is not possible through an engine installed via the Epic Launcher. This was deemed necessary due to the increase in size that would occur if the required files were included in the installed version of the engine._

### Packaging the Content

The dedicated server needs packaged content. Go to File -&gt; Package Project -&gt; Packaging Settings. Here you have a few options:

* Use Pak File: Pack all the assets into one .pak file - disable if you want to have the regular content structure \(e.g. for incremental uploads\)
* Full Rebuild: You might want to disable this to lower packaging times

  Then go to File -&gt; Package Project -&gt; Package Windows/Linux and select an output directory. The engine will now package the content and compile the standalone client code - this may take some time in case of a full rebuild.

### Compiling the Server

#### Linux Compiler Toolchain

Linux compilation is currently only supported on Windows using a cross-compilation toolchain based on Clang. A precompiled toolchain from Epic is available here: [https://github.com/EpicGames/UnrealEngine/releases/tag/4.1.0-release](https://github.com/EpicGames/UnrealEngine/releases/tag/4.1.0-release). After you unzipped the toolchain, make sure to add the environment variable LINUX\_ROOT and set it to the location of the toolchain \(see README.md for details\).

#### Compilation

The next step is to compile the server code using Visual Studio. First, you need to set up a special server target for UnrealBuildTool.

You can use the following template:

```cpp
// Copyright 1998-2014 Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;
using System.Collections.Generic;

public class GameServerTarget : TargetRules
{
    public GameServerTarget(TargetInfo Target)
    {
        Type = TargetType.Server;
    }

    //
    // TargetRules interface.
    //
    public override void SetupBinaries(
        TargetInfo Target,
        ref List<UEBuildBinaryConfiguration> OutBuildBinaryConfigurations,
        ref List<string> OutExtraModuleNames
        )
    {
        base.SetupBinaries(Target, ref OutBuildBinaryConfigurations, ref OutExtraModuleNames);
        OutExtraModuleNames.Add("Game");
    }

    public override bool GetSupportedPlatforms(ref List<UnrealTargetPlatform> OutPlatforms)
    {
        // It is valid for only server platforms
        return UnrealBuildTool.UnrealBuildTool.GetAllServerPlatforms(ref OutPlatforms, false);
    }

    public override List<UnrealTargetPlatform> GUBP_GetPlatforms_MonolithicOnly(UnrealTargetPlatform HostPlatform)
    {
        if (HostPlatform == UnrealTargetPlatform.Mac)
        {
            return new List<UnrealTargetPlatform>();
        }
        return new List<UnrealTargetPlatform> { HostPlatform, UnrealTargetPlatform.Win32, UnrealTargetPlatform.Linux };
    }

    public override List<UnrealTargetConfiguration> GUBP_GetConfigs_MonolithicOnly(UnrealTargetPlatform HostPlatform, UnrealTargetPlatform Platform)
    {
        return new List<UnrealTargetConfiguration> { UnrealTargetConfiguration.Development };
    }
}
```

Just replace all instances of "Game" with the name of your game project.

Its possible that there are no functions to override by the overrides so it will not build. Throw these out aswell if there are problems.

Save it as `<Game>Server.Target.cs` next to the other target files and regenerate the project files.

Open Visual Studio, set the configuration to \*Server and select the platform target accordingly.

Now it's time to build your game project. Using the above UBT target, the executable will end up in `<game>/Binaries/<platform>/<Game>Server`. Move the executable over to `<cooked>/<platform>/<game>/binaries/<platform>`.

### Platform Specifics

#### Windows

Simply execute `<Game>Server.exe`. If you want a log window, start with `-log`.

#### Linux

The server will listen for UDP packets on port 7777 by default, so make sure to open this port in your firewall.

With some Unreal Engine releases \(4.2 and below\) you can run into this message if you don't pass -pak:

```text
Could not adjust number of file handles, consider changing "nofile" in /etc/security/limits.conf and relogin.
```

The solution is to pass -pak on the command line when starting the server.

## Authors

Original author: **Epic Games**  
Captured from the epic wiki via the Wayback Machine. Reformatted by Maldonacho

