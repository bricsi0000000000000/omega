- [WPF](#wpf)
  - [Inherit `UserControl` from `class`](#inherit-usercontrol-from-class)

# WPF

## Inherit `UserControl` from `class`

> [Source](https://docs.microsoft.com/en-us/answers/questions/26046/wpf-usercontrol-and-inheritance-how-to.html)

Base class:

```cs
public abstract class PlotTelemetry : UserControl
{

}
```

Derived class:

```cs
public partial class LiveTelemetry : PlotTelemetry
{

}
```

```xml
<local:PlotTelemetry x:Class="LogicLayer.Menus.Live.LiveTelemetry">

</local:PlotTelemetry>
```

Change `UserControl` to `local:PlotTelemetry` inside `XAML`.

If it is not working, try reload project, or reopen Visual Studio.
