---
title: "Updating Existing Mac Apps"
description: "This document describes the steps that must be followed to update a Xamarin.Mac app from the Classic API to the Unified API."
ms.prod: xamarin
ms.assetid: 26673CC5-C1E5-4BAC-BEF4-9A386B296FD5
author: asb3993
ms.author: amburns
ms.date: 03/29/2017
---

# Updating Existing Mac Apps

Updating an existing app to use the Unified API requires changes to the project file itself as well as to the namespaces and APIs used in the application code.

## The Road to 64 Bits

The new Unified APIs are required to support 64 bit device architectures from a Xamarin.Mac application. As of February 1st, 2015 Apple requires that all new app submissions to the Mac App Store support 64 bit architectures.

Xamarin provides tooling for both Visual Studio for Mac and Visual Studio to automate the migration process from the Classic API to the Unified API or you can convert the project files manually. While the using the automatic tooling is highly suggested, this article will cover both methods.

### Before You Start...

Before you update your existing code to the Unified API, it is highly recommended that you eliminate all *compilation warnings*. Many *warnings* in the Classic API will become errors once you migrate to Unified. Fixing them before you start is easier because the compiler messages from the Classic API often provide hints on what to update.

## Automated Updating

Once the warnings have been fixed, select an existing Mac project in Visual Studio for Mac or Visual Studio and choose **Migrate to Xamarin.Mac Unified API** from the **Project** menu. For example:

![](updating-mac-apps-images/beta-tool1.png "Choose Migrate to Xamarin.Mac Unified API from the Project menu")

You'll need to agree to this warning before the automated migration will run (obviously you should ensure you have backups/source control before embarking on this adventure):

![](updating-mac-apps-images/migrate01.png "Agree to this warning before the automated migration will run")

There are two supported Target Framework types that can be selected when using the Unified API in a Xamarin.Mac application:

- **Xamarin.Mac Mobile Framework** - This is the same tuned .NET framework used by Xamarin.iOS and Xamarin.Android supporting a subset of the full **Desktop** framework. This is the recommended framework because it provides smaller average binaries due to superior linking behavior.
- **Xamarin.Mac .NET 4.5 Framework** - This framework is again, a subset of the **Desktop** framework. However, it trims off far less of the full **Desktop** framework than the **Mobile** framework and should _"just work"_ with most NuGet Packages or 3rd party libraries. This allows the developer to consume standard **Desktop** assemblies while still using a supported framework, but this option produces larger application bundles. This is the recommended framework where 3rd party .NET assemblies are being used that are not compatible with the **Xamarin.Mac Mobile Framework**. For a list of supported assemblies, please see our [Assemblies](~/cross-platform/internals/available-assemblies.md) documentation.

For detailed information on Target Frameworks and the implications of selecting a specific target for your Xamarin.Mac application, please see our [Target Frameworks](~/mac/platform/target-framework.md) documentation. 

The tool basically automates all the steps outlined in the **Update Manually** section presented below and is the suggested method of converting an existing Xamarin.Mac project to the Unified API.

## Steps to Update Manually

Again, once the warnings have been fixed, follow these steps to manually update Xamarin.Mac apps to use the new Unified API:

### 1. Update Project Type & Build Target

Change the project flavor in your **csproj** files from `42C0BBD9-55CE-4FC1-8D90-A7348ABAFB23` to `A3F8F2AB-B479-4A4A-A458-A89E7DC349F1`. Edit the **csproj** file in a text editor, replacing the first item in the `<ProjectTypeGuids>` element as shown:

![](updating-mac-apps-images/csproj.png "Edit the csproj file in a text editor, replacing the first item in the ProjectTypeGuids element as shown")

Change the **Import** element that contains `Xamarin.Mac.targets` to `Xamarin.Mac.CSharp.targets` as shown:

![](updating-mac-apps-images/csproj2.png "Change the Import element that contains Xamarin.Mac.targets to Xamarin.Mac.CSharp.targets as shown")

Add the following lines of code after the `<AssemblyName>` element:

```xml
<TargetFrameworkVersion>v2.0</TargetFrameworkVersion>
<TargetFrameworkIdentifier>Xamarin.Mac</TargetFrameworkIdentifier>

```

Example:

![](updating-mac-apps-images/csproj3.png "Add these lines of code after the <AssemblyName> element")

### 2. Update Project References

Expand the Mac application project's **References** node. It will initially show a *broken- **XamMac** reference similar to this screenshot (because we just changed the project type):

![](updating-mac-apps-images/references.png "It will initially show a broken- XamMac reference similar to this screenshot")

Click the **Gear Icon** beside the **XamMac** entry and select **Delete** to remove the broken reference.

Next, right-click on the **References** folder in the **Solution Explorer** and select **Edit References**. Scroll to the bottom of the list of references and place a check besides **Xamarin.Mac**.

![](updating-mac-apps-images/references2.png "Scroll to the bottom of the list of references and place a check besides Xamarin.Mac")

Press **OK** to save the project references changes.

### 3. Remove MonoMac from Namespaces

Remove the **MonoMac** prefix from namespaces in `using` statements or wherever a classname has been fully qualified (eg. `MonoMac.AppKit` becomes just `AppKit`).

### 4. Remap Types

[Native types](~/cross-platform/macios/nativetypes.md) have been introduced which replace some Types that were previously used, such as instances of `System.Drawing.RectangleF` with `CoreGraphics.CGRect` (for example). The full list of types can be found on the [native types](~/cross-platform/macios/nativetypes.md) page.

### 5. Fix Method Overrides

Some `AppKit` methods have had their signature changed to use the new [native types](~/cross-platform/macios/nativetypes.md) (such as `nint`). If custom subclasses override these methods the signatures will no longer match and will result in errors. Fix these method overrides by changing the subclass to match the new signature using native types. 

## Considerations

The following considerations should be taken into account when converting an existing Xamarin.Mac project from the Classic API to the new Unified API if that app relies on one or more Component or NuGet Package. 

### Components

Any component that you have included in your application will also need to be updated to the Unified API or you will get a conflict when you try to compile. For any included component, replace the current version with a new version from the Xamarin Component Store that supports the Unified API and do a clean build. Any component that has not yet been converted by the author, will display a 32 bit only warning in the component store.

### NuGet Support

While we contributed changes to NuGet to work with the Unified API support, there has not been a new release of NuGet, so we are evaluating how to get NuGet to recognize the new APIs. 

Until that time, just like the components, you'll need to switch any NuGet Package you have included in your project to a version that supports the Unified APIs and do a clean build afterwards.

> [!IMPORTANT]
> If you have an error in the form _"Error 3 Cannot include both 'monomac.dll' and 'Xamarin.Mac.dll' in the same Xamarin.Mac project - 'Xamarin.Mac.dll' is referenced explicitly, while 'monomac.dll' is referenced by 'xxx, Version=0.0.000, Culture=neutral, PublicKeyToken=null'"_ after converting your application to the Unified APIs, it is typically due to having either a component or NuGet Package in the project that has not been updated to the Unified API. You'll need to remove the existing component/NuGet, update to a version that supports the Unified APIs and do a clean build.

## Enabling 64 Bit Builds of Xamarin.Mac Apps

For a Xamarin.Mac mobile application that has been converted to the Unified API, the developer still needs to enable the building of the application for 64 bit machines from the app's Options. Please see the **Enabling 64 Bit Builds of Xamarin.Mac Apps** of the [32/64 bit Platform Considerations](~/cross-platform/macios/32-and-64/index.md) document for detailed instructions on enabling 64 bit builds.
	
## Finishing Up

Whether or not you choose to use the automatic or manual method to convert your Xamarin.Mac application from the Classic to the Unified APIs, there are several instances that will require further, manual intervention. Please see our [Tips for Updating Code to the Unified API](~/cross-platform/macios/unified/updating-tips.md) document for known issues and work arounds.

## Related Links

- [Tips for Updating Code to the Unified API](~/cross-platform/macios/unified/updating-tips.md)
- [Working with Native Types in Cross-Platform Apps](~/cross-platform/macios/native-types-cross-platform.md)
- [Classic vs Unified API differences](https://github.com/xamarin/release-notes-archive/blob/master/release-notes/ios/api_changes/classic-vs-unified-8.6.0/index.md)
