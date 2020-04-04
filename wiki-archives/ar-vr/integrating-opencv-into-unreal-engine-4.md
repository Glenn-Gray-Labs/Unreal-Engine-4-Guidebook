---
description: This article was originally written by Ginku; Converted by jfaw
---

# Integrating OpenCV into Unreal Engine 4

## Overview

Hello all! This is my first wiki tutorial, I hope it can be of help to someone!

I am making this tutorial in response to a few requests. This will be a detailed, step-by-step guide to linking OpenCV 3.2 to Unreal Engine 4 using the Unreal Build Tool. You can find a general tutorial on linking any static library to Unreal [here](Linking_Static_Libraries_Using_The_Build_System). I recommend reading it in addition to this tutorial, as it is what this tutorial is based on.

This tutorial focuses on the windows OS for simplicity. Note that the same process works for other OS, but you might need to build OpenCV from source.

### Why OpenCV?

OpenCV is a powerful open-source computer vision library, and once included into any unreal engine 4 project it will allow for the use of the engine in many non-traditional ways. Including OpenCV in a project as a dependency will allow developers to create state-of-the-art environments with either augmented reality components, virtual reality environments that resemble the user’s surroundings, or any mixture of the two. In this tutorial I will show you how to quickly and painlessly include OpenCV in any unreal engine 4 project with the Windows OS, and then I will guide you through displaying a webcam in a level.

### Using the Plugin

I have created a plugin that does all the OpenCV library linking, which can be found on its github. Installation instructions are inside the README file. With this plugin, you can skip to the Moving to Blueprints section of this tutorial. If you do so, be sure to enable the Computer Vision > OpenCV plugin inside the editor's Edit > Plugins menu.

Note: after installation, be sure to regenerate project files. For visual studios, right click your project (.uasset) file and select 'generate visual studio project files' after deleting your previous visual studios file.




## Linking OpenCV in Visual Studios

Before you continue, make sure you are starting from a code project, or have added code to your project with the editor!

### Copying the OpenCV Files

In order to begin, all of OpenCV’s include and library files will need to be added to your project’s `/ThirdParty` directory. To begin, install **OpenCV 3.2** or locate your installation of OpenCV 3.2 and do the following:

- Inside the OpenCV install directory you will find the `/build/include` directory. Copy all of the contents of this directory into the `[ProjectRootDirectory]/ThirdParty/OpenCV/Includes` directory. 

- Next, copy the _opencv_world320.dll_ and _opencv_ffmpeg320_64.dll_ files in the `/build/x64/vc14/bin` folder and the 'opencv_world320.lib', files in the `/build/x64/vc14/lib` folder to the `[ProjectRootDirectory]/ThirdParty/OpenCV/Libraries/Win64/` directory. 

_Note:_ This process is similar for any version of OpenCV or any third party library. You only need the runtime libraries (not the debug ones with a 'd' appended, such as _opencv_world320d.dll_ unless you need the debug versions).

### Adding OpenCV Dependencies

Locate and open your projects module rules file, which should be in your projects `Source/[Project Name]` directory. (It will be in the format of _ProjectName.Build.cs_) In this file, we will add the necessary code so that the unreal build tool will include all of the necessary dependencies during build time.

First, be sure to add the following include at the top of the file:

```csharp
using System.IO;
```

This lets you use the Path helper class, which is very useful for assembling directory paths!

Inside your ModuleRules class and before the constructor, add the following getter:

```csharp
private string ThirdPartyPath
{ 
    get { return Path.GetFullPath(Path.Combine(ModuleDirectory, "../../ThirdParty/")); } 
}
```

This is a helpful little function for retrieving the `/ThirdParty/` path, and can be very convenient when including more than one third party dependency to a project. Now, add the following function after the constructor:

```csharp
public bool LoadOpenCV(TargetInfo Target)
{
    // Start OpenCV linking here!
    bool isLibrarySupported = false;

    // Create OpenCV Path 
    string OpenCVPath = Path.Combine(ThirdPartyPath, "OpenCV");

    // Get Library Path 
    string LibPath = "";
    bool isdebug = Target.Configuration == UnrealTargetConfiguration.Debug && BuildConfiguration.bDebugBuildsActuallyUseDebugCRT;
    if (Target.Platform == UnrealTargetPlatform.Win64)
    {
        LibPath = Path.Combine(OpenCVPath, "Libraries", "Win64");
        isLibrarySupported = true;
    }
    else
    {
        string Err = string.Format("{0} dedicated server is made to depend on {1}. We want to avoid this, please correct module dependencies.", Target.Platform.ToString(), this.ToString()); System.Console.WriteLine(Err);
    }

    if (isLibrarySupported)
    {
        //Add Include path 
        PublicIncludePaths.AddRange(new string[] { Path.Combine(OpenCVPath, "Includes") });

        // Add Library Path 
        PublicLibraryPaths.Add(LibPath);

        //Add Static Libraries
        PublicAdditionalLibraries.Add("opencv_world320.lib");

        //Add Dynamic Libraries
        PublicDelayLoadDLLs.Add("opencv_world320.dll");
        PublicDelayLoadDLLs.Add("opencv_ffmpeg320_64.dll");
    }

    Definitions.Add(string.Format("WITH_OPENCV_BINDING={0}", isLibrarySupported ? 1 : 0));

    return isLibrarySupported;
}
```

This function includes all of the required includes and libraries for OpenCV. Now, simply call this function inside your project constructor after the standard public modules:

```csharp
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "RHI", "RenderCore", "ShaderCore" });

LoadOpenCV(Target);
```

Be sure to add the **InputCore**, **RHI**, and **RenderCore** engine modules to the _public_ dependency list, we will use these later to create a dynamic texture from the camera feed. The _PrivateIncludePaths_ will allow you to include the OpenCV header files without full paths.

Now, your project should successfully compile with OpenCV included within the engine build! However, there is one more thing that needs to be included before you can launch an instance of your project’s editor.

### Copying the DLL's to the Build

Before your project will run with any OpenCV code, you will first need to add all of the dynamically linked library (_.dll_) files that you use to your editor’s bin folder. Your editor will typically be a 64-bit application, so copy all of the _.dll_ files (_opencv_world320.dll_ and _opencv_ffmpeg320_64.dll_) from the OpenCV’s 64 bit bin folder (full directory shown above) and paste them inside the `[ProjectRootDirectory]/Binaries/Win64` directory.

Note: These DLL's should also be included with any distributions of the project (such as when packaging your game/project), by including them in the same directory as the project's executable (`MY_PROJECT.exe`).

### Fixing Library Collisions

There is a collision between the OpenCV3 library and UE4. To fix this, **comment out lines ~51 to ~55 and line ~852** of the _utility.hpp_ header file in the `'[ProjectRootDirectory]\ThirdParty\OpenCV\Includes\opencv2\core` directory.

```cpp
// NOTE: The OpenCV 'check' function has been commented out, as it conflicts with UE4 check - see line ~852
//#if defined(check)
//#  warning Detected Apple 'check' macro definition, it can cause build conflicts. Please, include this header before any Apple headers.
//#endif

...

//bool check() const;
```



## Adding a WebcamReader Class

You are now ready to launch an instance of your editor and start using OpenCV! Right click the project name in your solution explorer, and select _Debug > Start new instance_. If you get an error about the os being unable to load your dll, check out the discussion page. Once the editor loads, select _File > New C++ Class…_ and select the `Actor` parent class. Press Next, name the actor `WebcamReader` and press _Create Class_. Once Unreal has finished adding the new actor, the new header and source files will be opened inside Visual Studios. 

Add the following code to each of them:

##### Header
```cpp
// A simple webcam reader using the OpenCV library
// Author: The UE4 community
 
#pragma once
 
#include "opencv2/core.hpp"
#include "opencv2/highgui.hpp"	
#include "opencv2/imgproc.hpp"
#include "opencv2/videoio.hpp"
#include "GameFramework/Actor.h"
#include "Runtime/Engine/Classes/Engine/Texture2D.h"
#include "WebcamReader.generated.h"
 
UCLASS()
class YOURPROJECT_API AWebcamReader : public AActor
{
	GENERATED_BODY()
 
public:	
	// Sets default values for this actor's properties
	AWebcamReader();
 
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;
 
	// Called every frame
	virtual void Tick( float DeltaSeconds ) override;
 
	// The device ID opened by the Video Stream
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Webcam)
	int32 CameraID;
 
	// If the webcam images should be resized every frame
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Webcam)
	bool ShouldResize;
 
	// The targeted resize width and height (width, height)
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Webcam)
	FVector2D ResizeDeminsions;
 
	// The rate at which the color data array and video texture is updated (in frames per second)
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Webcam)
	float RefreshRate;
 
	// The refresh timer
	UPROPERTY(BlueprintReadWrite, Category = Webcam)
	float RefreshTimer;
 
	// Blueprint Event called every time the video frame is updated
	UFUNCTION(BlueprintImplementableEvent, Category = Webcam)
	void OnNextVideoFrame();
 
	// OpenCV fields
	cv::Mat frame;
	cv::VideoCapture stream;
	cv::Size size;
 
	// OpenCV prototypes
	void UpdateFrame();
	void DoProcessing();
	void UpdateTexture();
 
	// If the stream has succesfully opened yet
	UPROPERTY(BlueprintReadOnly, Category = Webcam)
	bool isStreamOpen;
 
	// The videos width and height (width, height)
	UPROPERTY(BlueprintReadWrite, Category = Webcam)
	FVector2D VideoSize;
 
	// The current video frame's corresponding texture
	UPROPERTY(BlueprintReadOnly, Category = Webcam)
	UTexture2D* VideoTexture;
 
	// The current data array
	UPROPERTY(BlueprintReadOnly, Category = Webcam)
	TArray<FColor> Data;
 
protected:
 
	// Use this function to update the texture rects you want to change:
	// NOTE: There is a method called UpdateTextureRegions in UTexture2D but it is compiled WITH_EDITOR and is not marked as ENGINE_API so it cannot be linked
	// from plugins.
	// FROM: https://wiki.unrealengine.com/Dynamic_Textures
	void UpdateTextureRegions(UTexture2D* Texture, int32 MipIndex, uint32 NumRegions, FUpdateTextureRegion2D* Regions, uint32 SrcPitch, uint32 SrcBpp, uint8* SrcData, bool bFreeData);
 
	// Pointer to update texture region 2D struct
	FUpdateTextureRegion2D* VideoUpdateTextureRegion;
};
```

##### Source
```cpp
// A simple webcam reader using the OpenCV library
// Author: The UE4 community
 
#include "YOURPROJECT.h"
#include "WebcamReader.h"
 
// Sets default values
AWebcamReader::AWebcamReader()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;
 
	// Initialize OpenCV and webcam properties
	CameraID = 0;
	RefreshRate = 15;
	isStreamOpen = false;
	VideoSize = FVector2D(0, 0);
	ShouldResize = false;
	ResizeDeminsions = FVector2D(320, 240);
	RefreshTimer = 0.0f;
	stream = cv::VideoCapture();
	frame = cv::Mat();
}
 
// Called when the game starts or when spawned
void AWebcamReader::BeginPlay()
{
	Super::BeginPlay();
	
	// Open the stream
	stream.open(CameraID);
	if (stream.isOpened())
	{
		// Initialize stream
		isStreamOpen = true;
		UpdateFrame();
		VideoSize = FVector2D(frame.cols, frame.rows);
		size = cv::Size(ResizeDeminsions.X, ResizeDeminsions.Y);
		VideoTexture = UTexture2D::CreateTransient(VideoSize.X, VideoSize.Y);
		VideoTexture->UpdateResource();
		VideoUpdateTextureRegion = new FUpdateTextureRegion2D(0, 0, 0, 0, VideoSize.X, VideoSize.Y);
 
		// Initialize data array
		Data.Init(FColor(0, 0, 0, 255), VideoSize.X * VideoSize.Y);

		// Do first frame
		DoProcessing();
		UpdateTexture();
		OnNextVideoFrame();
	}
 
}
 
// Called every frame
void AWebcamReader::Tick( float DeltaTime )
{
	Super::Tick( DeltaTime );
 
	RefreshTimer += DeltaTime;
	if (isStreamOpen && RefreshTimer >= 1.0f / RefreshRate)
	{
		RefreshTimer -= 1.0f / RefreshRate;
		UpdateFrame();
		DoProcessing();
		UpdateTexture();
		OnNextVideoFrame();
	}
}
 
void AWebcamReader::UpdateFrame()
{
	if (stream.isOpened())
	{
		stream.read(frame);
		if (ShouldResize)
		{
			cv::resize(frame, frame, size);
		}
	}
	else {
		isStreamOpen = false;
	}
}

void AWebcamReader::DoProcessing()
{
	// TODO: Do any processing here!
}
 
void AWebcamReader::UpdateTexture()
{
	if (isStreamOpen && frame.data)
	{
		// Copy Mat data to Data array
		for (int y = 0; y < VideoSize.Y; y++)
		{
			for (int x = 0; x < VideoSize.X; x++)
			{
				int i = x + (y * VideoSize.X);
				Data[i].B = frame.data[i * 3 + 0];
				Data[i].G = frame.data[i * 3 + 1];
				Data[i].R = frame.data[i * 3 + 2];
			}
		}
 
		// Update texture 2D
		UpdateTextureRegions(VideoTexture, (int32)0, (uint32)1, VideoUpdateTextureRegion, (uint32)(4 * VideoSize.X), (uint32)4, (uint8*)Data.GetData(), false);
	}
}
 
void AWebcamReader::UpdateTextureRegions(UTexture2D* Texture, int32 MipIndex, uint32 NumRegions, FUpdateTextureRegion2D* Regions, uint32 SrcPitch, uint32 SrcBpp, uint8* SrcData, bool bFreeData)
{
	if (Texture->Resource)
	{
		struct FUpdateTextureRegionsData
		{
			FTexture2DResource* Texture2DResource;
			int32 MipIndex;
			uint32 NumRegions;
			FUpdateTextureRegion2D* Regions;
			uint32 SrcPitch;
			uint32 SrcBpp;
			uint8* SrcData;
		};
 
		FUpdateTextureRegionsData* RegionData = new FUpdateTextureRegionsData;
 
		RegionData->Texture2DResource = (FTexture2DResource*)Texture->Resource;
		RegionData->MipIndex = MipIndex;
		RegionData->NumRegions = NumRegions;
		RegionData->Regions = Regions;
		RegionData->SrcPitch = SrcPitch;
		RegionData->SrcBpp = SrcBpp;
		RegionData->SrcData = SrcData;
 
		ENQUEUE_UNIQUE_RENDER_COMMAND_TWOPARAMETER(
			UpdateTextureRegionsData,
			FUpdateTextureRegionsData*, RegionData, RegionData,
			bool, bFreeData, bFreeData,
			{
			for (uint32 RegionIndex = 0; RegionIndex < RegionData->NumRegions; ++RegionIndex)
			{
				int32 CurrentFirstMip = RegionData->Texture2DResource->GetCurrentFirstMip();
				if (RegionData->MipIndex >= CurrentFirstMip)
				{
					RHIUpdateTexture2D(
						RegionData->Texture2DResource->GetTexture2DRHI(),
						RegionData->MipIndex - CurrentFirstMip,
						RegionData->Regions[RegionIndex],
						RegionData->SrcPitch,
						RegionData->SrcData
						+ RegionData->Regions[RegionIndex].SrcY * RegionData->SrcPitch
						+ RegionData->Regions[RegionIndex].SrcX * RegionData->SrcBpp
						);
				}
			}
			if (bFreeData)
			{
				FMemory::Free(RegionData->Regions);
				FMemory::Free(RegionData->SrcData);
			}
			delete RegionData;
		});
	}
}
```

Note: you need to change `YOURPROJECT` in the _class definition of the header file_ and the project include in the source file with the correct version, which is based on your project's name.

This class is used as a wrapper for a future unreal blueprint class. It allows you to specify the device ID, target resolution and framerate of the camera, as well as providing a dynamic texture and an `FColor` array of the current frame's pixels and a blueprint native event that is called whenever the next webcam frame is available!



## Moving to Blueprints

You can now access your webcam feed in blueprints. From this point forward we will be working in the editor.

### Adding the WebcamBillboard Actor

Now that all the code has been included for accessing your webcams, I will now show you how to use the dynamic texture from the WebcamReader actor in a new WebcamBillboard subclass. This time, the code will be implemented in unreal blueprints! Launch the editor again with _Debug > Start_ new instance. In your choice of directory, right click and add a new blueprint class. At the bottom of the new window, expand _All Classes_ and search `AWebcamReader` and select it as the parent class. Name the new blueprint `BP_WebcamBillboard` and open it.

Within the viewport, add a cube static mesh component, and name it `Billboard`. This will be the component that the texture is rendered to. At the beginning of the game, we will want to create a dynamic material instance and set it to the billboard mesh. Under _Variables_, click the + button to add a new _Material Instance Dynamic_ called `DynamicMaterial`’ Drag the `Billboard` component onto the Event Graph, and create a _getter_ node. Drag out from this new getter and create a _Create Dynamic Material Instance_ node and connect the white execution wire to the transparent _BeginPlay_ event (or create one). This creates an special Unreal material instance that can be altered at runtime. However, we have not created this Unreal material!

Go back to your content browser, right click and create a new material. Call this material `M_Webcam` and open it. Click on the `M_Webcam` node and set the _Shading Mode_ to _Unlit_. Hold `T` and left click anywhere in the new graph to create a texture node. You will have to set the default texture to anything (I used `T_Ceramic_Tile_M`). Right click this node and convert it to a parameter. Call this parameter _Texture_ and connect its white `Float3` pin to the _Emissive Color_ pin on the `M_Webcam` node. Make sure the save the material!

Now, back in the `BP_WebcamBillboard` blueprint, select the `M_Webcam` as the _Source Material_ for the _Create Dynamic Material Instance_ node, and make sure the _Element Index_ is set to `0`. Drag out from the original billboard getter and create a _Set Material_ node. Set the _Material_ pin to the output of the _Create Dynamic Material Instance_ node, and again make sure the _Element Index_ is set to `0`. Finally, drag out the _DynamicMaterial_ variable we created earlier and create a setter. Connect the output of the _Create Dynamic Material Instance_ node to the _DynamicMaterial_ input pin to save a reference of this special material for later use.

We have dynamically set the material of our _Billboard_ mesh, and now we need to update its texture parameter each time a new frame is received. To do this, right click on the _Event Graph_ and create a _OnNextVideoFrame_ event. This event is called in the `AWebcamReader` actor whenever a new frame is read. Drag out the _DynamicMaterial_ variable and create a getter underneath the new event. Drag out from the getter and create a _Set Texture Parameter Value_ node. Set the _Parameter Name_ to _Texture_ (the name of the texture parameter in the `M_Webcam` material). Right click on the _Event Graph_ and type _VideoTexture_ to retrieve a reference to the webcam texture provided by the `AWebcamReader` parent class. Connect the output pin of this _VideoTexture_ reference to the _Value_ pin of the _Set Texture Parameter Value_ node. With that, the `BP_WebcamBillboard` is ready for use!


Drag the `BP_WebcamBillboard` blueprint from the content browser into your level. Orientate and position it however you like, and scale it to a similar scale of your images resolution (about `6.4`, `4.8`, and `0.5` for my webcam). Now, set the Webcam properties in the detail panel. (I used a _Camera ID_ of `0`, _Should Resize_ to `false`, and _Refresh Rate_ of `2.0`) Your _Camera ID_ will determine the camera that renders, it should be `0` unless you have more than one webcam. In the case of a laptop, `0` will probably be the integrated laptop, and `1+` will be any additional webcams. You can now press play to see the results!

![Picture](/.gitbook/assets/integratingopencvintounrealengine4/adding-the-webcam-to-a-level.png "Project Creation")

You can hold alt and move the object to duplicate it. Change its Camera ID to render a second webcam!

![Picture](/.gitbook/assets/integratingopencvintounrealengine4/adding-the-webcam-to-a-level2.png "Project Creation")

Good luck with your OpenCV / UE4 Projects! :)

## Final Notes

For packaged versions of your OpenCV projects, be sure that the executable has access to the dll's. For a windows build, this can be done by coppying the _opencv_world320.dll_ and _opencv_ffmpeg320_64.dll_ into the `WindowsNoEditor/YOURPROJECT` directory. (There are 2 executables for windows builds, but the one in the `YOURPROJECT` directory is the one that needs access to the dll's, regardless of which one you use to launch your project!)

UE4 and the C++ standard library do not play well together. This can cause annoying crashes, such as with the `cv::findContours` function (often during the `std::vector` destructors). Members of the community have gotten around this by wrapping `findContour` calls in a separate library, and including that in their UE4 projects with steps similar to the ones to include the OpenCV library in this tutorial.

The webcam reader shown in this tutorial is designed to be simple and easy to follow. However, it isn't very efficient, as all of the OpenCV reading code and UE4 dynamic texture code occurs on the game thread, potentially per-tick with high refresh rates! I would recommend separating these parts into another thread to increase performance. Rama's tutorial on multi-threading is a good start if you are unfamiliar with UE4 threads.

If any of these points are confusing, feel free to say so on my talk page and when I get the chance I will add an in-depth section to this tutorial! :)

-Ginku