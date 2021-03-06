---
author: stevewhims
description: An introduction to C++/WinRT&mdash;a standard C++ language projection for WinRT APIs.
title: Introduction to C++/WinRT
ms.author: stwhi
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp, standard, c++, cpp, winrt, projection
ms.localizationpriority: medium
---

# Introduction to C++/WinRT
> [!NOTE]
> **Some information relates to pre-released product which may be substantially modified before it’s commercially released. Microsoft makes no warranties, express or implied, with respect to the information provided here.**

Introduced in Windows 10, version 1803, the Windows SDK now includes C++/WinRT.

C++/WinRT is an entirely standard modern C++17 language projection for Windows Runtime (WinRT) APIs, implemented solely in header files, and designed to provide you with first-class access to the modern Windows API. With C++/WinRT, you can author and consume WinRT APIs using any standards-compliant C++17 compiler.

## Language projections
WinRT is based on Component Object Model (COM) APIs, and it's designed to be accessed through *language projections*. A projection hides the COM details, and provides a more natural programming experience for a given language.

### The C++/WinRT language projection in the Windows UWP API reference content
When you're browsing [Windows UWP APIs](https://docs.microsoft.com/uwp/api/), click the **Language** combo box in the upper right, and select **C++/WinRT** to view API syntax blocks as they appear in the C++/WinRT language projection.

## SDK support for C++/WinRT
As of Windows 10, version 1803, the Windows SDK contains the C++/WinRT projection Windows namespace headers and tools. One important tool is `cppwinrt.exe` which, from a `.winmd`, generates source code files that project the metadata into C++/WinRT. Windows Metadata (`.winmd`) files provide a canonical way of describing a WinRT API surface. From Windows Metadata, `cppwinrt.exe` generates a standard C++ library that fully describes&mdash;or *projects*&mdash;the API surface; whether they're Windows APIs, or third-party Windows Runtime Component APIs. `Cppwinrt.exe` plays an important role in your development workflow for both consuming first- and third-party APIs and also for authoring your own APIs in components.

## Visual Studio support for C++/WinRT
Because C++/WinRT uses features from the C++17 standard, it needs project property **C/C++** > **Language** > **ISO C++17 Standard (/std:c++17)**. You might also want to set **Conformance mode: Yes (/permissive-)**, which further constrains your code to be standards-compliant.

Another project property to be aware of is **C/C++** > **General** > **Treat Warnings As Errors**. Set this to **Yes(/WX)** or **No (/WX-)** to taste. Source files generated by the `cppwinrt.exe` tool can generate warnings until you add your implementation to them.

> [!NOTE]
> 
> **The C++/WinRT Visual Studio Extension (VSIX) is not yet released.**
> 
> For C++/WinRT project templates, as well as C++/WinRT MSBuild properties and targets, download and install the C++/WinRT Visual Studio Extension (VSIX) from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/). You'll need Visual Studio 2017 Version 15.6, or later. You can then create a new project in Visual Studio, or you can convert an existing project by adding the `<CppWinRTProject>true</CppWinRTProject>` property to its `.vcxproj` file, inside Project > PropertyGroup. Once you've added that property, you'll get C++/WinRT MSBuild support for the project, including invoking the `cppwinrt.exe` tool.

These are the Visual Studio project templates for C++/WinRT.

### Windows Console Application (C++/WinRT)
A project template for a C++/WinRT client application for Windows Desktop, with a console user-interface.

### Blank App (C++/WinRT)
A project template for a Universal Windows Platform (UWP) app that has a XAML user-interface.

Visual Studio provides XAML compiler support to generate implementation and header stubs from the Interface Definition Language (IDL) (`.idl`) file that sits behind each XAML markup file. In an IDL file, define any local runtime classes that you want to reference in your app's XAML pages, and then build the project once to generate implementation templates in `Generated Files`, and stub type definitions in `Generated Files\sources`. Then use those the stub type definitions for reference to implement your local runtime classes. We recommend that you declare each runtime class in its own IDL file.

### Core App (C++/WinRT)
A project template for a Universal Windows Platform (UWP) app that doesn't use XAML.

Instead, it uses the C++/WinRT projection Windows namespace header for the Windows.ApplicationModel.Core namespace. After building and running, click on an empty space to add a colored square; then click on a colored square to drag it.

### Windows Runtime Component (C++/WinRT)
A project template for a component; typically for consumption from a Universal Windows Platform (UWP).

This template demonstrates the `midl.exe` > `cppwinrt.exe` toolchain, where Windows Metadata (`.winmd`) is generated from IDL, and then implementation and header stubs are generated from the Windows Metadata.

In an IDL file, define the runtime classes in your component, their default interface, and any other interfaces they implement. Build the project once to generate `module.g.cpp`, `module.h.cpp`, implementation templates in `Generated Files`, and stub type definitions in `Generated Files\sources`. Then use those the stub type definitions for reference to implement the runtime classes in your component. We recommend that you declare each runtime class in its own IDL file.

Bundle the built Windows Runtime Component binary and its `.winmd` with the UWP app consuming them.

## A C++/WinRT quick-start
Create a new **Windows Console Application (C++/WinRT)** project. Edit `main.cpp` to look like this.

```cppwinrt
// main.cpp

#include "pch.h"
#include <iostream>
#include "winrt/Windows.Foundation.h"
#include "winrt/Windows.Web.Syndication.h"

using namespace winrt;
using namespace Windows::Foundation;
using namespace Windows::Web::Syndication;

int main()
{
	init_apartment();

	Uri rssFeedUri{ L"https://blogs.windows.com/feed" };
	SyndicationClient syndicationClient;
	SyndicationFeed syndicationFeed = syndicationClient.RetrieveFeedAsync(rssFeedUri).get();
	for (const SyndicationItem syndicationItem : syndicationFeed.Items())
	{
		hstring titleAsHstring = syndicationItem.Title().Text();
		std::wcout << titleAsHstring.c_str() << std::endl;
	}
}
```

The included headers `winrt/Windows.Foundation.h` and `winrt/Windows.Web.Syndication.h` are in the SDK, inside the folder `%ProgramFiles(x86)%\Windows Kits\10\Include<WindowsTargetPlatformVersion>\cppwinrt\winrt\`. Visual Studio includes that path in its *IncludePath* macro. The headers contain Windows APIs projected into C++/WinRT. Whenever you want to use types from the Windows namespaces, include the corresponding C++/WinRT projection Windows namespace headers like this. The `using namespace` directives are optional, but convenient.

All the projected types are in the C++/WinRT root namespace **winrt**. Both [C++/CX](/cpp/cppcx/visual-c-language-reference-c-cx) and the Windows SDK declare types in the root namespace **Windows**. These distinct namespaces let you migrate from C++/CX to C++/WinRT at your own pace.

[**SyndicationClient::RetrieveFeedAsync**](/uwp/api/windows.web.syndication.syndicationclient.retrievefeedasync) is an example of an asynchronous WinRT function. The code example receives an asynchronous operation object from **RetrieveFeedAsync**, and it calls **get** on that object to block the calling thread and wait for the results. For more about concurrency, and for non-blocking techniques, see [Concurrency and asynchronous operations with C++/WinRT](concurrency.md).

[**SyndicationFeed.Items**](/uwp/api/windows.web.syndication.syndicationfeed.items) is a range, defined by the iterators returned from **begin** and **end** functions (or their constant, reverse, and constant-reverse variants). Because of this, you can enumerate **Items** with either a range-based `for` statement, or with the **std::for_each** template function.

The code then gets the feed's title text, as a [**winrt::hstring**](/uwp/cpp-ref-for-winrt/hstring) object (see [String handling in C++/WinRT](strings.md)). The **hstring** is then output, via **c_str**, which will look familiar to you if you've used strings from the C++ Standard Library.

As you can see, C++/WinRT encourages modern, and class-like, C++ expressions such as `syndicationItem.Title().Text()`. This is a different, and cleaner programming style from traditional COM programming. You don't need to explicitly initialize COM (**init_apartment** does that for you), work with COM pointers, nor handle HRESULT return codes. C++/WinRT converts error HRESULTs to exceptions for a natural and modern programming style.

## Custom types in the C++/WinRT projection
You can use standard C++ language features and [Standard C++ data types and C++/WinRT](std-cpp-data-types.md)&mdash;including some C++ Standard Library data types&mdash;in your C++/WinRT programming. But you'll also become aware of some custom data types in the projection, and you can choose to use them. For example, we used [**winrt::hstring**](/uwp/cpp-ref-for-winrt/hstring) in the quick start above.

[**winrt::com_array**](/uwp/cpp-ref-for-winrt/com-array) is another type that you're likely to use at some point. But you're less likely to directly use a type such as [**winrt::array_view**](/uwp/cpp-ref-for-winrt/array-view). Or you may choose not to use it so that you won't have any code to change if and when an equivalent type appears in the C++ Standard Library.

There are also types that you might see if you closely study the C++/WinRT projection Windows namespace headers. An example is **winrt::param::hstring**. These exist only for efficiency reasons, and you should not use them in your code.

## Important APIs
* [winrt namespace (C++/WinRT)](/uwp/cpp-ref-for-winrt/winrt)
* [SyndicationClient::RetrieveFeedAsync](/uwp/api/windows.web.syndication.syndicationclient.retrievefeedasync)
* [SyndicationFeed.Items](/uwp/api/windows.web.syndication.syndicationfeed.items)
* [winrt::hstring](/uwp/cpp-ref-for-winrt/hstring)

## Related topics
* [C++/CX](/cpp/cppcx/visual-c-language-reference-c-cx)
* [Windows UWP APIs](https://docs.microsoft.com/uwp/api/)
* [Visual Studio Marketplace](https://marketplace.visualstudio.com/)
* [String handling in C++/WinRT](strings.md)