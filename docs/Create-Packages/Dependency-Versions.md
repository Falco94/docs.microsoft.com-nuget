---
# required metadata

title: Dependency Versions in NuGet | Microsoft Docs
author: kraigb
ms.author: kraigb
manager: ghogen
ms.date: 5/4/2017
ms.topic: article
ms.prod: nuget
#ms.service:
ms.technology: null
ms.assetid: ae6c6212-67e9-4968-9585-e265407fd9c8

# optional metadata

description: Exact details on specifying version numbers and ranges for other packages upon which a NuGet package depends, and how dependencies are installed.
keywords: versioning, NuGet package dependencies, NuGet dependency versions, NuGet version numbers, NuGet package version, version ranges, version specifications, normalized version numbers
#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer:
- karann
- unnir
#ms.suite:
#ms.tgt_pltfrm:
#ms.custom:

---
# Specifying dependency versions for NuGet Packages

*For .NET Core projects using NuGet 4.0+, see [Package References in Project Files](../consume-packages/package-references-in-project-files.md) for declaring dependencies.*

When you [create a NuGet package](../create-packages/creating-a-package.md), you can specify dependencies for your package in the `<dependencies>` node of the `.nuspec` file, where each dependency is listed with a `<dependency>` tag:

```xml
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
    <metadata>
        <!-- ... -->

        <dependencies>
            <dependency id="Newtonsoft.Json" version="9.0" />
            <dependency id="EntityFramework" version="6.1.0" />
        </dependencies>
    </metadata>
</package>
```

Dependencies are installed whenever the dependent package is installed, [reinstalled](../consume-packages/reinstalling-and-updating-packages.md), or included in a [package restore](../consume-packages/package-restore.md). NuGet chooses the exact version of the installed dependency by using the value of the `version` attribute specified for that dependency as described in the following sections.

- [Version ranges](#version-ranges)
- [Normalized version numbers](#normalized-version-numbers)
- [Dependency updates during package install](#dependency-updates-during-package-install).

> **Best practice**
>
> Developers most commonly specify a minimum version, which is a version number without adornments as shown above, like 6.1.0. This allows NuGet to install that version or later.


For additional details on how dependencies are installed, see [Reinstalling and updating packages](../consume-packages/reinstalling-and-updating-packages.md) and [Dependency resolution](../consume-packages/dependency-resolution.md).


## Version ranges

NuGet supports using interval notation for specifying version ranges, summarized as follows:

| Notation | Applied rule | Description |
|----------|--------------|-------------|
| 1.0 | 1.0 ≤ x | Minimum version, inclusive |
| (1.0,) | 1.0 < x | Minimum version, exclusive |
| [1.0] | x == 1.0 | Exact version match |
| (,1.0] | x ≤ 1.0 | Maximum version, inclusive |
| (,1.0) | x < 1.0 | Maximum version, exclusive |
| [1.0,2.0] | 1.0 ≤ x ≤ 2.0 | Exact range, inclusive |
| (1.0,2.0) | 1.0 < x < 2.0 | Exact range, exclusive |
| [1.0,2.0) | 1.0 ≤ x < 2.0 | Mixed inclusive minimum and exclusive maximum version |
| (1.0)    | invalid | invalid |

> [!Note]
> Pre-release versions are not included when resolving version ranges. Pre-release versions are included when using a wildcard (*) with a floating version number. That is, *[1.0,2.0]* will not include 2.0-beta, but *2.0-** will. However, due to issues [4513](https://github.com/NuGet/Home/issues/4513) and [4859](https://github.com/NuGet/Home/issues/4859), this resolves the lowest matching version, not the latest.


A few examples:

```xml
<!-- Accepts any version 6.1 and above -->
<dependency id="ExamplePackage" version="6.1" />

<!-- Accepts any version above, but not include 4.1.3. This might be
      used to guarantee a dependency with a specific bug fix. -->
<dependency id="ExamplePackage" version="(4.1.3,)" />

<!-- Accepts any version up below 5.x, which might be used to prevent
      pulling in a later version of a dependency that changed its interface.
      However, this form is not recommended because it can be difficult to
      determine the lowest version.  -->
<dependency id="ExamplePackage" version="(,5.0)" />

<!-- Accepts any 1.x or 2.x version, but no 0.x or 3.x and higher versions -->
<dependency id="ExamplePackage" version="[1,3)" />

<!-- Accepts 1.3.2 up to 1.4.x, but not 1.5 and higher. -->
<dependency id="ExamplePackage" version="[1.3.2,1.5)" />
```

<br/>

> [!Important]
> Always specify a version or version range for package dependencies. NuGet.org will reject packages that lack dependency versions. In addition, without a version, NuGet 2.8.x and earlier will use the latest package version, whereas NuGet 3.x and later will use the lowest package version. Specifying versions avoids this uncertainty. 

## Normalized version numbers

> [!Note]
> This is a breaking change for NuGet 3.4 and later.


When obtaining packages from a repository during install, reinstall, or restore operations, NuGet 3.4 and later will treat version numbers as follows:

- Leading zeroes are removed from version numbers:

    ```
    1.00 is treated as 1.0
    1.01.1 is treated as 1.1.1
    1.00.0.1 is treated as 1.0.0.1
    ```

- A zero in the fourth part of the version number will be omitted

    ```
    1.0.0.0 is treated as 1.0.0
    1.0.01.0 is treated as 1.0.1
    ```

This normalization does not affect the version numbers in the packages themselves; it affects only how NuGet matches versions.

However, NuGet repositories must treat these values in the same way as NuGet to prevent package version duplication. Thus a repository that contains v1.0 of a package should not also host v1.0.0 as a separate and different package.

## Dependency updates during package install

With NuGet 2.4.x and earlier, when a package is installed whose dependency already exists in the project, the dependency is updated to the latest version that satisfies the version constraints, even if the existing version also satisfies those constraints.

For example, consider package A that depends on package B and specifies 1.0 for the version number. The source repository contains both versions 1.0, 1.1, and 1.2 of package B. If A is installed in a project that already contains B version 1.0, then B is updated to version 1.2.

With NuGet 2.5 and later, if a dependency version is already satisfied, the dependency will not be updated during other package installations.

In the same example above, installing package A into a project with NuGet 2.5 and later will leave package B 1.0 in the project, as it already satisfies the version constraint. However, if package A had requests version 1.1 or higher of B, then B 1.2 would be installed.
