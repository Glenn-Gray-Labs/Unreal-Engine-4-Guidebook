---
description: Guide on String conversions (from/to) by Rama the legend
---

# String Conversions: FString to FName, FString to Int32, Float to FString

**Content**
- [Overview](#overview)
  * [Converting FString to FNames](#converting-fstring-to-fnames)
  * [std::string to FString](#std--string-to-fstring)
  * [FString to std::string](#fstring-to-std--string)
- [FCString Overview](#fcstring-overview)
  * [Converting FString to Numbers](#converting-fstring-to-numbers)
  * [FString to Integer](#fstring-to-integer)
  * [FString to Float](#fstring-to-float)
- [Float/Integer to FString](#float-integer-to-fstring)
- [UE4 Source Header References](#ue4-source-header-references)
- [Optimization Issues Concerning FNames](#optimization-issues-concerning-fnames)

### Overview

**Original Author: Rama**

Below are conversions for the following types:
1. FString to FName
2. std::string to FString
3. FString and FCString Overview
4. FString to Integer
5. FString to Float
6. Float/Integer to FString
7. UE4 C++ Source Header References
8. Optimization Issues Concerning FNames

All the header files I refer to in this tutorial are found in 
```
your UE4 install directory  / Engine / Source
```
you will probably want to do a search for them from this point :) 

#### Converting FString to FNames

Say we have
```cpp
FString TheString = "UE4_C++_IS_Awesome";
```
To convert this to an FName you do:
```cpp
FName ConvertedFString = FName(*TheString);
```

####  std::string to FString

```cpp
#include <string>
//....
std::string TestString = "Happy"; 
FString HappyString(TestString.c_str());
```

####  FString to std::string

```cpp
#include <string>
//....
FString UE4Str = "Flowers";
std::string MyStdString(TCHAR_TO_UTF8(*UE4Str));
```
You will find this particularly useful in cases other than float and int32!
C++ std::String::to_string
http://en.cppreference.com/w/cpp/string/basic_string/to_string

###  FCString Overview

#### Converting FString to Numbers

The * operator on FStrings returns their TCHAR* data which is what FCString functions use.
If you cant find the function you want in FStrings (UnrealString.h) then you should check out the FCString functions (CString.h)
I show how to convert from FString to FCString below:
Say we have
```cpp
FString TheString = "123.021";
```

####  FString to Integer

(note Atoi is unsafe; no way to indicate errors)
```cpp
int32 MyShinyNewInt = FCString::Atoi(*TheString);
```

####  FString to Float

```cpp
float MyShinyNewFloat = FCString::Atof(*TheString);
```
Note that Atoi and Atof are static functions, so you use the syntax FCString::TheFunction to call it :)

###  Float/Integer to FString

```cpp
FString NewString = FString::FromInt(YourInt);
FString VeryCleanString = FString::SanitizeFloat(YourFloat);
```
Static functions in the UnrealString.h :)

###  UE4 Source Header References

```cpp
CString.h
UnrealString.h
NameTypes.h
```
See CString.h for more details and other functions like
```cpp
atoi64 (string to int64)
Atod	(string to double precision float)
```
For a great deal of helpful functions you will also want to look at
UnrealString.h for direct manipulation of FStrings!

###  Optimization Issues Concerning FNames

FNames are inherently fast, but you could be forcing a hashmap lookup if you are accessing them in the wrong way. Look at the following code:
```cpp
if (ActorHasTag(TEXT("MyFNameActor_Tag")))
```
This code will take the character string "MyFNameActor_Tag" and then look it up in the FName hashmap.
Whereas this code doesn't need to do a string conversion:
```cpp
static const FName NAME_MyFNameActor(TEXT("MyFNameActor_Tag"));
if (ActorHasTag(NAME_MyFNameActor))
```
In our testing with UE4 4.14, the second method is nearly 100 times faster than using the string lookup. So please, always use the static const FName method over the TEXT() method.
For more info on FNames check out
```cpp
NameTypes.h
```
Enjoy!

## Authors

Original author: Rama &lt;3
Minor Authors: Kory
Captured from the epic wiki via the Wayback Machine. Reformatted by DarioMazzanti

