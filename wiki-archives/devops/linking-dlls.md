---
description: This wiki article was written by Original Author ZkarmaKun (talk); Updated & Improved by F3NR1S (talk), XenoEgger, Darkgaze; Converted by jfaw
---

# Linking DLLs

## Overview

This tutorial explains how to link / bind your own [DLL](https://en.wikipedia.org/wiki/Dynamic-link_library) to Unreal Engine 4 and how to use your DLL's methods for visual scripting in a [Blueprint Function Library](https://docs.unrealengine.com/latest/INT/Programming/BlueprintFunctionLibraries/). 



## Creating a C++ DLL

This article originally centered on binding the DLL but here is also a brief explanation on how to build DLLs in different [IDEs](https://en.wikipedia.org/wiki/Integrated_development_environment). 



### Visual Studio Community 2015

- Create a new project: _menu bar -> File -> New -> Project..._

    1. In the New Project window on the left select _Installed -> Templates -> Visual C++ -> Win32._
    2. Select _Win32 Project_ in the middle.
    3. Name: the project _CreateAndLinkDLLTut_ and the Solution name: _CreateAndLinkDLLTutSol_.
    4. Click _OK_. 

- In the next window _Win32 Application Wizard - CreateAndLinkDLLTut_ click _Next_.

    1. In the **Application Settings** select
        - _Application type: -> DLL_
        - _Additional options: -> Empty project_
    2. Click _Finish_. 

- On the left side of Visual Studio in the Solution Explorer make sure that CreateAndLinkDLLTut is selected.

    1. Click _main menu -> Project -> Add Class..._
    2. In the _Add Class_ window select _Installed -> Visual C++ -> C++ ->_ on the left side and _C++ Class_ in the middle then click _Add_.
    3. In the _Generic C++ Class Wizard_ window fill in _CreateAndLinkDLLfile_ into the _Class name: input_ field. Click _Finish_.

- On the left side in the _Solution Explorer_ select the file _CreateAndLinkDLLfile.h_ and copy & paste the following code. Replace all automatically generated code.

```cpp
#pragma once  

#define DLL_EXPORT __declspec(dllexport)	//shortens __declspec(dllexport) to DLL_EXPORT

#ifdef __cplusplus		//if C++ is used convert it to C to prevent C++'s name mangling of method names
extern "C"
{
#endif

	bool DLL_EXPORT getInvertedBool(bool boolState);
	int DLL_EXPORT getIntPlusPlus(int lastInt);
	float DLL_EXPORT getCircleArea(float radius);
	char DLL_EXPORT *getCharArray(char* parameterText);
	float DLL_EXPORT *getVector4( float x, float y, float z, float w);

#ifdef __cplusplus
}
#endif
```

- Then select the file _CreateAndLinkDLLfile.cpp_ and copy & paste the following code. Replace all automatically generated code.

```cpp
#pragma once

#include "string.h"
#include "CreateAndLinkDLLFile.h"


//Exported method that invertes a given boolean.
bool getInvertedBool(bool boolState)
{
	return bool(!boolState);
}

//Exported method that iterates a given int value.
int getIntPlusPlus(int lastInt)
{
	return int(++lastInt);
}

//Exported method that calculates the are of a circle by a given radius.
float getCircleArea(float radius)
{
	return float(3.1416f * (radius * radius));
}

//Exported method that adds a parameter text to an additional text and returns them combined.
char *getCharArray(char* parameterText)
{
	char* additionalText = " world!";
	
	if (strlen(parameterText) + strlen(additionalText) + 1 > 256)
	{
		return "Error: Maximum size of the char array is 256 chars.";
	}

	char combinedText[256] = "";
	
	strcpy_s( combinedText, 256, parameterText);
	strcat_s( combinedText, 256, additionalText);
	
	return ( char* )combinedText;
}

//Exported method that adds a vector4 to a given vector4 and returns the sum.
float *getVector4( float x, float y, float z, float w )
{
	float* modifiedVector4 = new float[4];
	
	modifiedVector4[0] = x + 1.0F;
	modifiedVector4[1] = y + 2.0F;
	modifiedVector4[2] = z + 3.0F;
	modifiedVector4[3] = w + 4.0F;

	return ( float* )modifiedVector4;
}
```

- Save with _menu bar -> File -> Save All_.
- Set the proper build options for your 64-bit DLL: In the menu bar select **Release** as _Solution Configuration_ and **x64** as _Solution Platform_. ( If you use a 32-bit Windows system please select **x86** instead of x64. )
- Build the DLL with _menu bar -> Build -> Build CreateAndLinkDLLTut_. The Output at the bottom should show a message like _========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========_.
- The 64-bit DLL was created in the folder _.../CreateAndLinkDLLTutSol/x64/Release/_ and is called _CreateAndLinkDLLTut.dll_. ( The 32-bit DLL was created in the folder _.../CreateAndLinkDLLTutSol/Release/_ and is called _CreateAndLinkDLLTut.dll_. [You won't be able to bind the DLL if the platforms are different.](https://answers.unrealengine.com/questions/30927/does-it-make-a-difference-if-you-load-a-custom-win.html) )



## Unreal Engine Project

- On **Unreal Engine**, create a new project: _New Project -> C++ -> Basic Code_.
- Name the project _CreateAndLinkDLLProj_ and create it.
- Open **Windows Explorer**.

    1. Go to the main folder of your created UE4 project.
    2. Add a folder called [Plugins](https://docs.unrealengine.com/latest/INT/Programming/Plugins/index.html#pluginfolders).
    3. In the Plugins folder create an other folder called _MyTutorialDLLs_.
    4. Copy and paste the DLL _CreateAndLinkDLLTut.dll_ you have created earlier into the folder _MyTutorialDLLs_. 

- Add a new C++ class to your project in **Unreal Editor**.
- Choose the _Blueprint Function Library_ as the base class.
- Name your blueprint function library _CreateAndLinkDLLTutBFL_.
- If **Visual Studio** does not open it automatically, open it by double clicking _CreateAndLinkDLLTutBFL_ in the UE4 content browser.
- Open the _CreateAndLinkDLLTutBFL.h_ and _CreateAndLinkDLLTutBFL.cpp_ files.
- Select the file _CreateAndLinkDLLTutBFL.h_ and copy & paste the following code:

```cpp
#pragma once

#include "Kismet/BlueprintFunctionLibrary.h"
#include "CreateAndLinkDLLTutBFL.generated.h"


UCLASS()
class CREATEANDLINKDLLPROJ_API UCreateAndLinkDLLTutBFL : public UBlueprintFunctionLibrary
{
	GENERATED_BODY()
	
public:

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool importDLL( FString folder, FString name);


	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool importMethodGetInvertedBool( );

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool importMethodGetIntPlusPlus( );

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool importMethodGetCircleArea( );

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool importMethodGetCharArray( );

	UFUNCTION( BlueprintCallable, Category = "My DLL Library" )
	static bool importMethodGetVector4( );
	

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static bool getInvertedBoolFromDll(bool boolState);

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static int getIntPlusPlusFromDll(int lastInt);

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static float getCircleAreaFromDll(float radius);

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static FString getCharArrayFromDll(FString parameterText);

	UFUNCTION( BlueprintCallable, Category = "My DLL Library" )
	static FVector4 getVector4FromDll( FVector4 vector4 );
	

	UFUNCTION(BlueprintCallable, Category = "My DLL Library")
	static void freeDLL();
};
```

- Select the file _CreateAndLinkDLLTutBFL.cpp_ and copy & paste the following code:

```cpp
#include "CreateAndLinkDLLProj.h"
#include "CreateAndLinkDLLTutBFL.h"

typedef bool(*_getInvertedBool)(bool boolState); // Declare a method to store the DLL method getInvertedBool.
typedef int(*_getIntPlusPlus)(int lastInt); // Declare a method to store the DLL method getIntPlusPlus.
typedef float(*_getCircleArea)(float radius); // Declare a method to store the DLL method getCircleArea.
typedef char*(*_getCharArray)(char* parameterText); // Declare a method to store the DLL method getCharArray.
typedef float*(*_getVector4)(float x, float y, float z, float w); // Declare a method to store the DLL method getVector4.

_getInvertedBool m_getInvertedBoolFromDll;
_getIntPlusPlus m_getIntPlusPlusFromDll;
_getCircleArea m_getCircleAreaFromDll;
_getCharArray m_getCharArrayFromDll;
_getVector4 m_getVector4FromDll;

void *v_dllHandle;


#pragma region Load DLL

// Method to import a DLL.
bool UCreateAndLinkDLLTutBFL::importDLL(FString folder, FString name)
{
	FString filePath = *FPaths::GamePluginsDir() + folder + "/" + name;
	
	if (FPaths::FileExists(filePath))
	{
		v_dllHandle = FPlatformProcess::GetDllHandle(*filePath); // Retrieve the DLL.
		if (v_dllHandle != NULL)
		{
			return true;
		}
	}
	return false;	// Return an error.
}
#pragma endregion Load DLL

#pragma region Import Methods

// Imports the method getInvertedBool from the DLL.
bool UCreateAndLinkDLLTutBFL::importMethodGetInvertedBool()
{
	if (v_dllHandle != NULL)
	{
		m_getInvertedBoolFromDll = NULL;
		FString procName = "getInvertedBool";	// Needs to be the exact name of the DLL method.
		m_getInvertedBoolFromDll = (_getInvertedBool)FPlatformProcess::GetDllExport(v_dllHandle, *procName);
		if (m_getInvertedBoolFromDll != NULL)
		{
			return true;
		}
	}
	return false;	// Return an error.
}

// Imports the method getIntPlusPlus from the DLL.
bool UCreateAndLinkDLLTutBFL::importMethodGetIntPlusPlus()
{
	if (v_dllHandle != NULL)
	{
		m_getIntPlusPlusFromDll = NULL;
		FString procName = "getIntPlusPlus";	// Needs to be the exact name of the DLL method.
		m_getIntPlusPlusFromDll = (_getIntPlusPlus)FPlatformProcess::GetDllExport(v_dllHandle, *procName);
		if (m_getIntPlusPlusFromDll != NULL)
		{
			return true;
		}
	}
	return false;	// Return an error.
}

// Imports the method getCircleArea from the DLL.
bool UCreateAndLinkDLLTutBFL::importMethodGetCircleArea()
{
	if (v_dllHandle != NULL)
	{
		m_getCircleAreaFromDll = NULL;
		FString procName = "getCircleArea";	// Needs to be the exact name of the DLL method.
		m_getCircleAreaFromDll = (_getCircleArea)FPlatformProcess::GetDllExport(v_dllHandle, *procName);
		if (m_getCircleAreaFromDll != NULL)
		{
			return true;
		}
	}
	return false;	// Return an error.
}

// Imports the method getCharArray from the DLL.
bool UCreateAndLinkDLLTutBFL::importMethodGetCharArray()
{
	if (v_dllHandle != NULL)
	{
		m_getCharArrayFromDll = NULL;
		FString procName = "getCharArray";	// Needs to be the exact name of the DLL method.
		m_getCharArrayFromDll = (_getCharArray)FPlatformProcess::GetDllExport(v_dllHandle, *procName);
		if (m_getCharArrayFromDll != NULL)
		{
			return true;
		}
	}
	return false;	// Return an error.
}

// Imports the method getVector4 from the DLL.
bool UCreateAndLinkDLLTutBFL::importMethodGetVector4( )
{
	if( v_dllHandle != NULL )
	{
		m_getVector4FromDll = NULL;
		FString procName = "getVector4";	// Needs to be the exact name of the DLL method.
		m_getVector4FromDll = ( _getVector4 ) FPlatformProcess::GetDllExport( v_dllHandle, *procName );
		if( m_getVector4FromDll != NULL )
		{
			return true;
		}
	}
	return false;	// Return an error.
}

#pragma endregion Import Methods

#pragma region Method Calls

// Calls the method getInvertedBoolFromDll that was imported from the DLL.
bool UCreateAndLinkDLLTutBFL::getInvertedBoolFromDll(bool boolState)
{
	if (m_getInvertedBoolFromDll != NULL)
	{
		bool out = bool(m_getInvertedBoolFromDll(boolState)); // Call the DLL method with arguments corresponding to the exact signature and return type of the method.
		return out;
	}
	return boolState;	// Return an error.
}

// Calls the method m_getIntPlusPlusFromDll that was imported from the DLL.
int UCreateAndLinkDLLTutBFL::getIntPlusPlusFromDll(int lastInt)
{
	if (m_getIntPlusPlusFromDll != NULL)
	{
		int out = int(m_getIntPlusPlusFromDll(lastInt)); // Call the DLL method with arguments corresponding to the exact signature and return type of the method.
		return out;
	}
	return -32202;	// Return an error.
}

// Calls the method m_getCircleAreaFromDll that was imported from the DLL.
float UCreateAndLinkDLLTutBFL::getCircleAreaFromDll(float radius)
{
	if (m_getCircleAreaFromDll != NULL)
	{
		float out = float(m_getCircleAreaFromDll(radius)); // Call the DLL method with arguments corresponding to the exact signature and return type of the method.
		return out;
	}
	return -32202.0F;	// Return an error.
}

// Calls the method m_getCharArrayFromDLL that was imported from the DLL.
FString UCreateAndLinkDLLTutBFL::getCharArrayFromDll(FString parameterText)
{
	if (m_getCharArrayFromDll != NULL)
	{
		char* parameterChar = TCHAR_TO_ANSI(*parameterText);

		char* returnChar = m_getCharArrayFromDll(parameterChar);

		return (ANSI_TO_TCHAR(returnChar));
	}
	return "Error: Method getCharArray was probabey not imported yet!";	// Return an error.
}

// Calls the method m_getVector4FromDll that was imported from the DLL.
FVector4 UCreateAndLinkDLLTutBFL::getVector4FromDll( FVector4 vector4 )
{
	if( m_getVector4FromDll != NULL )
	{
		float* vector4Array = m_getVector4FromDll( vector4.X, vector4.Y, vector4.Z, vector4.W );

		return FVector4( vector4Array[0], vector4Array[1], vector4Array[2], vector4Array[3] );
	}
	return FVector4( -32202.0F, -32202.0F, -32202.0F, -32202.0F );	// Return an error.
}
#pragma endregion Method Calls


#pragma region Unload DLL

// If you love something  set it free.
void UCreateAndLinkDLLTutBFL::freeDLL()
{
	if (v_dllHandle != NULL)
	{
		m_getInvertedBoolFromDll = NULL;
		m_getIntPlusPlusFromDll = NULL;
		m_getCircleAreaFromDll = NULL;
		m_getCharArrayFromDll = NULL;
		m_getVector4FromDll = NULL;

		FPlatformProcess::FreeDllHandle(v_dllHandle);
		v_dllHandle = NULL;
	}
}
#pragma endregion Unload DLL
```

- Save with _menu bar -> File -> Save All_.



## Creating the Blueprint

- First hit the Compile button PD CompileButton.PNG of the Unreal Editor to compile the code you've added and saved in Visual Studio before.
- In Unreal Editor add a new Blueprint Class called _BP_DllTest_ and open it. ([How to create a blueprint class](https://docs.unrealengine.com/latest/INT/Engine/Blueprints/UserGuide/Types/ClassBlueprint/Creation/index.html))
- Select the _Event Graph_ and add the following nodes construct ( Click it and click it again to download it! ). 
    - **Important note**: If you don't see the functions in the dropdown, try compiling from Visual Studio and then reopening UE4. If this doesn't work, close UE4, remove Binaries folder and Intermediate folder (but avoid deleting Intermediate/Project Files). You will be prompted to rebuild the project.

![Picture](/.gitbook/assets/linkingdlls/blueprint-graph1.png "Project Creation")

- Then compile and save the blueprint and drag & drop it into the level.
- The result should look like this:

![Picture](/.gitbook/assets/linkingdlls/result1.png "Project Creation")



## Project Source Code

You can download the final [Visual Studio Solution of the DLL](https://github.com/XenoEgger/CreateAndLinkDLLTutSol) and the [Unreal Engine 4 project](https://github.com/XenoEgger/CreateAndLinkDLLProj) from GitHub. ( You may need to rebuild the DLL and copy it to your UE4 _Plugins_ folder. ) 



## Final Words

- You can use any DLL from C code or C++ or other languages.
- You can use unmanaged or managed (CLR, .Net Framework) code from a project in your solution or external.
- Most issues arise from differences in the signature of the DLL function and the type definition in the Unreal Project.
- Automatic packaging of third party DLL is not yet supported, you will need to package the DLL, the DLL folder and the plugin folder as well, which is not created in a package by default at this time.
- Be mindful of load times of DLL, it may slow down your project.
- Be mindful of processing time of your DLL, your project loses execution control inside the DLL, until it returns, it may be expensive to perform some operations.
- To go further with this tutorial:
    - C++ with proper class, namespace and name mangling.
    - DLL with multithreading and callback example.
    
