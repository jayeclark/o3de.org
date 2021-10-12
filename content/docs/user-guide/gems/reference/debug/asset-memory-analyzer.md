---
linkTitle: Asset Memory Analyzer
title: Asset Memory Analyzer Gem
description: The Asset Memory Analyzer Gem provides tools to profile asset memory usage in Open 3D Engine (O3DE) through ImGUI (Immediate Mode Graphical User Interface).
toc: true
---

Resource management is critical, particularly on target platforms where memory is limited. In any given project, the model, texture, animation, and audio resource files that make up the project's assets use the bulk of the memory allocated to run the project. **Asset Memory Analyzer** shows how memory is allocated to assets as your project runs. It is an indispensable tool to balance memory usage and get the best performance for your project.

Asset Memory Analyzer is an **Open 3D Engine (O3DE)** Gem that displays a table of heap and VRAM memory allocations per asset through the **ImGUI** (Immediate Mode Graphical User Interface) overlay. In addition to live display of memory allocations for assets loaded in the project, Asset Memory Analyzer can export allocation data to `JSON` and `CSV` files.

## Enable Asset Memory Analyzer

To use Asset Memory Analyzer, enable asset scope tracking, configure and compile a profile build of your project, and enable asset memory analysis.

1. Add the Asset Memory Analyzer and ImGUI Gems to your project.

1. Configure and build your project in profile mode.

1. Set the CVAR `assetmem_enabled=1` via the **Editor Console** to enable asset memory analysis. To make the setting persistent, add `assetmem_enabled=1` to an appropriate config file, such as `/dev/system_windows_pc.cfg`.

## View Live Asset Memory Analysis with ImGUI

To view live asset memory allocation, enable the ImGUI overlay when testing your project and choose Asset Memory Analyzer in the ImGUI window.

1. In **O3DE Editor**, press **Ctrl+G** or press the **Play** button to run your project.

1. Press the **Home** key to open the ImGUI overlay window.

1. Choose Asset Memory Analyzer from the top of the ImGUI overlay window.

![The ImGUI overlay displaying Asset Memory Analyzer.](/images/user-guide/gems/assetmemoryanalyzer/ui-asset-memory-analyzer-A.png)

Each recorded asset is displayed along with the number of allocations and total size in kilobytes for both heap and VRAM. Use the selections at the top of the ImGUI overlay window to sort the table by heap size, heap count, VRAM size, VRAM count, or alphabetically by asset label.

{{< note >}}
The asset label is the full path for each asset from the root of the project folder.
{{< /note >}}

Click the **Arrow** to the left of an asset to expand it and view individual allocations and references belonging to the asset.

![Expanded view of an asset in the Asset Memory Analyzer.](/images/user-guide/gems/assetmemoryanalyzer/ui-asset-memory-analyzer-B.png)

## Export an Asset Memory Analysis Snapshot to a File 

Snapshots of asset memory allocation can be exported to `JSON` or `CSV` files through three methods:

* Click Asset Memory Analyzer in the ImGUI overlay window and choose **Export JSON** or **Export CSV**.
* 

![Snapshot output options for the Asset Memory Analyzer.](/images/user-guide/gems/assetmemoryanalyzer/ui-asset-memory-analyzer-C.png)

* Use the console commands `assetmem_export_json` or `assetmem_export_csv` in  Editor Console to generate the file.

* Call `ExportJSONFile` or `ExportCSVFile` on the `AssetMemoryAnalyzerRequestBus` in C++, with `nullptr` as the parameter, to generate the file in the default location.

    ```c++
    EBUS_EVENT(AssetMemoryAnalyzerRequestBus, ExportJSONFile, nullptr);
    ```

The snapshot file is created in the project log directory located at `/Cache/ProjectName/pc/user/log` and named with the time and date of the snapshot.

{{< note >}}
Due to the limitations of the `CSV` format, only a top-level overview of assets will be written to `CSV`, *without* the hierarchical drill-down available for `JSON` files or the ImGUI overlay window.
{{< /note >}}

## View a JSON Asset Memory Analysis Snapshot in a Browser

`JSON` snapshots can be viewed in a browser with a web viewer provided with O3DE. The web viewer is located at `/Gems/AssetMemoryAnalyzer/www/AssetMemoryViewer/index.html`. Open the `index.html` file and drag-and-drop the `JSON` file onto the page, or click on the target area to browse to it. This displays the contents of the file in an expandable table.

![The Asset Memory Viewer displaying a JSON snapshot in a browser.](/images/user-guide/gems/assetmemoryanalyzer/ui-asset-memory-analyzer-D.png)

{{< note >}}
Chromium based browsers are most reliable.
{{< /note >}}

The table can be sorted by any column. The columns give a breakdown by multiple categories:

* **Heap** allocations and **VRAM** allocations
* **Local** summary (not including any sub-assets), and **Total** summary (including all sub-assets)
* **Number** of allocations and **Kilobytes** allocated

Expanding assets will display individual allocations belonging to the asset and sub-assets that were loaded through references.

The fields at the top of the table can be used to filter the assets by their label on a number of conditions, including regular expressions.

![The Asset Memory Viewer filtering a JSON snapshot in a browser by keyword.](/images/user-guide/gems/assetmemoryanalyzer/ui-asset-memory-analyzer-E.png)

## Instrumenting Code for Asset Memory Analysis

### Initial Loading of an Asset

Asset Memory Analyzer traps allocations (heap and VRAM) that occur during a slice of code execution or scope when an asset is active for recording. When a system begins loading a new asset, it should use the `AZ_ASSET_NAMED_SCOPE` macro to demarcate the C++ scope where that asset might be making allocations. For example:

```c++
#include <AzCore/Debug/AssetMemoryDriller.h>

Foo* LoadMyFooAsset(const char* name)
{
    AZ_ASSET_NAMED_SCOPE("Foo: %s", name);
    Foo* result = aznew Foo(name);  // The call to aznew will be recorded as associated with the asset "Foo: <name>"

    return result;  // When this function exits, the asset will no longer be in scope, and subsequent allocations will not be recorded
}
```

### Subsequent Processing of an Asset

When the asset is being updated, otherwise processed, or is being handed off to a different thread, it should use the `AZ_ASSET_ATTACH_TO_SCOPE` macro with a pointer that was allocated and tracked by the initial asset. This will associate any further allocations with the same asset until the scope of the declaration closes.

```c++
#include <AzCore/Debug/AssetMemoryDriller.h>

void UpdateAllFoos(const AZStd::vector<Foo*>& allFoos)
{
    for (Foo* foo : allFoos)
    {
        AZ_ASSET_ATTACH_TO_SCOPE(foo);  // Subsequent allocations in this scope will associate with any asset that was in scope when foo was allocated
        UpdateFoo(foo);
    }
}

void UpdateFoo(Foo* foo)
{
    aznew Bar;  // This automatically gets recorded with the owning asset for foo
    AZStd::thread doThreadedWork([foo]()
    {
        // Work being done on a different thread means we need to reattach to the owning asset
        AZ_ASSET_ATTACH_TO_SCOPE(foo);
        aznew Bar;  // This will now be recorded under the owning asset for foo
    });
    doThreadedWork.join();
}
```

You can attempt to attach to any pointer that was created while that asset was in scope, *or even any portion of memory that was allocated to it*.

For instance, the following code is valid:

```c++
#include <AzCore/Debug/AssetMemoryDriller.h>

struct Baz
{
    int a;
    char* b;
    double c;
};

Baz* CreateBaz(const char* name)
{
    AZ_ASSET_NAMED_SCOPE(name);
    Baz* baz = aznew Baz;  // bar is associated with the named asset
    return baz;
}

void TestScopes()
{
    Baz* baz = CreateBaz("My test baz");

    {
        AZ_ASSET_ATTACH_TO_SCOPE(&baz->c);  // This works, even though "c" didn't have its own allocation
        baz->b = aznew char[32];  // This allocation will be recorded under the asset "My test baz"
    }
}
```

An original pointer to an object that was allocated within a scope is *not* required in order to attach to it. This makes it possible to attach across systems to objects that have been defined with multiple inheritance.

## EBus Processing of and Asset

EBus handlers can automatically attempt to attach to a scope for each handler receiving an event. This works when the handler was allocated as part of an asset.

If the handler was created while an asset was in scope, you can modify an EBus as follows:

```c++
#include <AzCore/Debug/AssetMemoryDriller.h>

class MyEvents : public AZ::EBusTraits
{
    // Process individual events by first attempting to attach to the asset that owns the handler
    template<typename Bus>
    using EventProcessingPolicy = Debug::AssetTrackingEventProcessingPolicy<Bus>;

    // Regular Ebus definitions
    virtual void MyFunction() = 0;
};
```

Some O3DE EBuses use this feature, such as the `TickBus`. If you find others that should use it, please add them! You should not default to using this `EventProcessingPolicy` if it is not applicable.

Instrumentation does create some overhead, which can negatively affect your project's performance.

Creating a new named scope requires:

* Function calls
* An environment lookup
* Locking a mutex
* Two hashtable lookups
* Thread-local modificationss

Attaching to an existing scope requires:

* Function calls
* An environment lookup
* Locking a mutex
* A lookup in a large red-black tree
* Thread-local modifications

Creating a scope is generally a small cost, but significant enough that you should not use the `AZ_ASSET_ATTACH_TO_SCOPE` macro or use the `AssetTrackingEventProcessingPolicy` on an EBus when not tracking assets.

When asset tracking is disabled (`AZ_ANALYZE_ASSET_MEMORY` is undefined or `0`), there's no cost to instrumentation of scopes. This is default in performance builds.

{{< note >}}
Because `Debug` builds disable optimization, EBuses that use the `AssetTrackingEventProcessingPolicy` will still have the indirection of a function call for each instrumented EBus handler.
{{< /note >}}
