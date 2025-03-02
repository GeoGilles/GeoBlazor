﻿# GeoBlazor

[View the live demo site!](https://dy-blazor-samples-server.azurewebsites.net/)

[Join our Discord Server!](https://discord.gg/hcmbPzn4VW)

This project wraps the [ArcGIS Javascript API](https://developers.arcgis.com/javascript/latest/) in a Blazor templating framework.
It generates a nuget package that can be imported and consumed from any Blazor application, without directly interacting with javascript.

In addition to "hiding" the javascript implementation, the goal is also to make a simple, component-based system for declaring a map and view. For example:

```html
<MapView Longitude="_longitude" Latitude="_latitude" Zoom="11" Style="height: 600px; width: 100%;">
    <Map ArcGISDefaultBasemap="arcgis-topographic">
        <GraphicsLayer>
            <Graphic>
                <Point Longitude="_longitude" Latitude="_latitude"/>
                <SimpleMarkerSymbol Color="@(new MapColor(226, 119, 40))">
                    <Outline Color="@(new MapColor(255, 255, 255))" Width="1"/>
                </SimpleMarkerSymbol>
            </Graphic>
            <Graphic>
                <PolyLine Paths="@_mapPaths"/>
                <SimpleLineSymbol Color="@(new MapColor(226, 119, 40))" Width="2"/>
            </Graphic>
            <Graphic>
                <Polygon Rings="@_mapRings"/>
                <SimpleFillSymbol Color="@(new MapColor(227, 139, 79, 0.8))">
                    <Outline Color="@(new MapColor(255, 255, 255))" Width="1"/>
                </SimpleFillSymbol>
                <Attributes Name="This is a Title" Description="And a Description"/>
                <PopupTemplate Title="{Name}" Content="{Description}"/>
            </Graphic>
        </GraphicsLayer>
    </Map>
</MapView>
```

for a 2D map with a default ArcGIS basemap, or

```html
<SceneView Longitude="_longitude" Latitude="_latitude" Zoom="11" Style="height: 600px; width: 100%;" ZIndex="2000" Tilt="76">
    <Map Ground="world-elevation">
        <Basemap>
            <PortalItem Id="f35ef07c9ed24020aadd65c8a65d3754" />
        </Basemap>
        <GraphicsLayer>            <Graphic>
                <Point Longitude="_longitude" Latitude="_latitude"/>
                <SimpleMarkerSymbol Color="@(new MapColor(226, 119, 40))">
                    <Outline Color="@(new MapColor(255, 255, 255))" Width="1"/>
                </SimpleMarkerSymbol>
            </Graphic>
            <Graphic>
                <PolyLine Paths="@_mapPaths"/>
                <SimpleLineSymbol Color="@(new MapColor(226, 119, 40))" Width="2"/>
            </Graphic>
            <Graphic>
                <Polygon Rings="@_mapRings"/>
                <SimpleFillSymbol Color="@(new MapColor(227, 139, 79, 0.8))">
                    <Outline Color="@(new MapColor(255, 255, 255))" Width="1"/>
                </SimpleFillSymbol>
                <Attributes Name="This is a Title" Description="And a Description"/>
                <PopupTemplate Title="{Name}" Content="{Description}"/>
            </Graphic>
        </GraphicsLayer>
    </Map>
</SceneView>
```

for a 3D map with a basemap loaded from a `PortalId`.

## [Using the Library](UsingTheAPI.md)

### Known Limitations/"Gotchas"

- All classes inheriting from `MapComponent`, implement `IAsyncDisposable`. If you use these components in Razor markdown,
  it handles disposal for you. However, there may be situations (e.g., adding a new graphic on the fly),
  where you want to instantiate one of these components in C# code. Be aware that if you do this, you need to call
  `DisposeAsync` yourself when you are done with the object.
- Directly calling a `HashSet` collection like  `graphicLayer.Add(graphic)` or `MapView.Widgets.Add(widget)` does not 
  work currently to register these components with the JavaScript API. Instead, use 
  `graphicLayer.RegisterChildComponent(graphic)` or `MapView.RegisterChildComponent(widget)`.
- While `JsRuntime` calls are very efficient, some "real-time" events, like handling a mouse pointer moving,
  may have enough latency to notice on a round-trip to Blazor/C# and back to JavaScript. In these situations, it is 
  our recommendation to instead write custom JavaScript code, which can handle the event completely client-side. 
  See `DisplayProjection.razor` in the `...Sample.Shared` library for an example of writing your own JavaScript that interacts
  with GeoBlazor.

## Build Requirements

For the Asp.NET projects, including the core library, you can run on the latest [.NET 6 SDK](https://dotnet.microsoft.com/en-us/download).

For the Maui sample project, you need the latest [_preview_ of Visual Studio](https://visualstudio.microsoft.com/vs/preview/).

If you have not installed node.js on your system, you will need to download and install it in order for the npm scripts to run. Please [select
the appropriate installer for your system](https://nodejs.org/en/download/).

If you have not installed powershell on your system (macOS and Linux users), or if you would like the latest version of Powershell for Windows, you will need to install Powershell and then change the "execution policies" on the system.
-Complete installation instructions for Powershell [can be found here]
(https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.2).

Because Geoblazor uses an unsigned, local powershell script to copy files in the `Sample.Shared` project, you need to allow unsigned scripts to be run in Powershell.
-The procedure to change the "execution policies" and set them to `RemoteSigned` are found here:
https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2#change-the-execution-policy

## Projects

### dymaptic.GeoBlazor.Core

- The core logic library

### dymaptic.GeoBlazor.Core.Sample.Shared

- A razor class library for sample applications
- All sample pages are based on the [ArcGIS for Javascript API Tutorials](https://developers.arcgis.com/javascript/latest/).

### dymaptic.GeoBlazor.Core.Sample.Server

- Asp.NET Core Blazor Server application sample
- `dotnet run --project .\samples\dymaptic.GeoBlazor.Core.Sample.Server\dymaptic.GeoBlazor.Core.Sample.Server.csproj`
- Runs on kestrel or via IIS
- Serves pages via SignalR/Websockets
- Can be loaded with a `usersecrets` file to provide the ArcGIS Api Key.

### dymaptic.GeoBlazor.Core.Sample.Wasm

- `dotnet run --project .\samples\dymaptic.GeoBlazor.Core.Sample.Wasm\dymaptic.GeoBlazor.Core.Sample.Wasm.csproj`
- Runs Blazor in Web Assembly on the client browser
- No safe storage for API key, users must input an api key or sign in from the browser

### dymaptic.GeoBlazor.Core.Sample.Maui

- Cross-platform mobile and desktop application
- Should be run from Visual Studio Preview. Command Line support appears to be limited at this time.
- Android and Windows versions tested

### dymaptic.GeoBlazor.Interactive (not included in open source repo)

- Extended application features - coming soon!
  - Custom renderers (e.g. image icons), see [Feature Layers (demo)](https://blazor.dymaptic.com/feature-layers)
  - Custom popups (e.g. charts, tables), see [Popups (demo)](https://blazor.dymaptic.com/popups)
  - Advanced widgets (e.g. sketch, track), see [Sketch Query (demo)](https://blazor.dymaptic.com/sketch-query)
  - Custom layers (e.g. GeoJSON Layer), see [Projection (demo)](https://blazor.dymaptic.com/projection)
  - Advanced event handling (e.g., pointer move, sketch events, search events), see above examples
  - GeometryEngine direct calls, see [Calculate Geometries (demo)](https://blazor.dymaptic.com/calculate-geometries)
  - ArcGIS Rest direct calls, see [Demographic Data (demo)](https://blazor.dymaptic.com/demographic-data)
- Please contact info@dymaptic.com to discuss licensing these advanced features!
