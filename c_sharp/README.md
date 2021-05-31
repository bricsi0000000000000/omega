- [Casual c](#casual-c)
  - [`IEnumerable` vs `List`](#ienumerable-vs-list)
  - [Interesting things](#interesting-things)
- [Entity Framework](#entity-framework)
  - [Data Annotations](#data-annotations)
- [ASP.NET](#aspnet)
  - [Life cycle](#life-cycle)
    - [PreInit](#preinit)
    - [Init](#init)
    - [InitComplete](#initcomplete)
    - [OnPreLoad](#onpreload)
    - [Load](#load)
    - [Control PostBack Event(s)](#control-postback-events)
    - [LoadComplete](#loadcomplete)
    - [OnPreRender](#onprerender)
    - [OnSaveStateComplete](#onsavestatecomplete)
    - [Render Method](#render-method)
    - [UnLoad](#unload)
    - [Example](#example)
- [WPF](#wpf)
  - [Inherit `UserControl` from `class`](#inherit-usercontrol-from-class)

# Casual c#

> [Source](https://stackoverflow.com/questions/3628425/ienumerable-vs-list-what-to-use-how-do-they-work#3628705)

## `IEnumerable` vs `List`

`IEnumerable` describes behavior, while **`List` is an implementation** of that behavior.

When you use `IEnumerable`, you give the compiler a chance to **defer work until later**, possibly optimizing along the way. If you use `ToList()` you force the compiler to reify the results right away.

*"Stacking"* **LINQ** expressions, use `IEnumerable`, because by only specifying the behavior you give **LINQ** a chance to defer evaluation and possibly optimize the program.

**LINQ** doesn't generate the SQL to query the database until you enumerate it.

```cs
public IEnumerable<Animals> AllSpotted()
{
  return from a in Zoo.Animals
         where a.coat.HasSpots == true
         select a;
}

public IEnumerable<Animals> Feline(IEnumerable<Animals> sample)
{
  return from a in sample
         where a.race.Family == "Felidae"
         select a;
}

public IEnumerable<Animals> Canine(IEnumerable<Animals> sample)
{
  return from a in sample
         where a.race.Family == "Canidae"
         select a;
}
```

Now you have a method that selects an initial sample (`AllSpotted`), plus some filters. So now you can do this:

```cs
var Leopards = Feline(AllSpotted());
var Hyenas = Canine(AllSpotted());
```

Is it faster to use `List` over `IEnumerable`?

- Only if you want to prevent a query from being executed more than once.

But is it better overall?

- In the above, Leopards and Hyenas get converted into **single SQL queries each**, and the **database only returns the rows that are relevant**.
- If we had returned a `List` from `AllSpotted()`, then **it may run slower** because **the database could return far more data than is actually needed**, and we waste cycles doing the filtering in the client.

In a program, **it may be better to defer converting your query to a list until the very end**, so if I'm going to enumerate through Leopards and Hyenas more than once, I'd do this:

```cs
List<Animals> Leopards = Feline(AllSpotted()).ToList();
List<Animals> Hyenas = Canine(AllSpotted()).ToList();
```

## Interesting things

`null` becomes before anything else.

# Entity Framework

Create migration: `dotnet ef migrations add InitialCreate`

Create database and schema: `dotnet ef database update`

## Data Annotations

> [Source](https://entityframework.net/data-annotations)

| Attribute         | Description                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------- |
| Key               | To make the corresponding column a primary pey (`PK`) column in the database.                       |
| ForeignKey        | Specifies that the property is the foreign key in a relationship.                                 |
| Required          | To make the corresponding column a `NOT NULL` column in a database table.                           |
| MinLength         | Specify a minimum number of characters or bytes for the column that the property should map to.   |
| MaxLength         | Specify a maximum number of characters or bytes for the column that the property should map to.   |
| StringLength      | Sets the maximum allowed length of the property value.                                            |
| Table             | Create a table with a specified name in Table attribute for a given domain class.                 |
| Column            | Create a column with a specified name in Column attribute for a given property in a domain class. |
| ConcurrencyCheck  | Specifies that the property is included in concurrency checks.                                    |
| Timestamp         | Specifies the data type of the database column as rowversion.                                     |
| NotMapped         | Applied to properties or classes that are to be excluded from database mapping.                   |
| InverseProperty   | Specifies the inverse of a navigation property.                                                   |
| ComplexType       | Complex Types cannot be tracked on their own but they are tracked as part of an entity.           |
| DatabaseGenerated | Specifies how the database generates values for the property.                                     |

# ASP.NET

## Life cycle

> [Source](https://www.c-sharpcorner.com/UploadFile/8911c4/page-life-cycle-with-examples-in-Asp-Net/)

![life cycle](images/ASP.NET%20Page%20Life%20Cycle.png)

### PreInit

1. Check the `IsPostBack` property to determine whether this is the first time the page is being processed.
2. Create or re-create dynamic controls.
3. Set a master page dynamically.
4. Set the Theme property dynamically.

If the request is a postback then the values of the controls have not yet been restored from the view state. If you set a control property at this stage, its value might be overwritten in the next event.

```cs
protected void Page_PreInit(object sender, EventArgs e)
{

}
```

### Init

1. This event fires after each control has been initialized.
2. Each control's UniqueID is set and any skin settings have been applied.
3. Use this event to **read or initialize control properties**.
4. The "Init" event is fired first for the bottom-most control in the hierarchy, and then fired up the hierarchy until it is for the page itself.

```cs
protected void Page_Init(object sender, EventArgs e)
{
  
}
```

### InitComplete

1. Until now the viewstate values are not yet loaded, hence you can use this event to make changes to the view state that you want to ensure are persisted after the next postback.
2. Raised by the Page object.
3. Use this event for **processing tasks that require all initialization to be complete**.

```cs
protected void Page_InitComplete(object sender, EventArgs e)
{
  
}
```

### OnPreLoad

1. Raised after the page loads view state for itself and all controls, and after it processes postback data that is included with the Request instance.
2. Before the Page instance raises this event, it loads view state for itself and all controls, and then processes any postback data included with the Request instance.
3. Loads ViewState: ViewState data are loaded to controls.
4. Loads Postback data: Postback data are now handed to the page controls.

```cs
protected void Page_OnPreLoad(object sender, EventArgs e)
{
  
}
```

### Load

1. The Page object **calls** the **`OnLoad`** method on the Page object, and **then recursively does the same for each child control until the page and all controls are loaded**. The `Load` event of individual controls occurs after the `Load` event of the page.
2. This is the **first place** in the page lifecycle that all **values are restored**.
3. **Most code checks the value of `IsPostBack` to avoid unnecessarily resetting state**.
4. You may also call `Validate` and check the value of `IsValid` in this method.
5. You can also create dynamic controls in this method.
6. Use the `OnLoad` event method to **set properties in control**s and **establish database connections**.

```cs
protected void Page_Load(object sender, EventArgs e)
{
  
}
```

### Control PostBack Event(s)

1. ASP.NET calls **any events on the page or its controls** that caused the PostBack to occur.
2. Use these events to **handle specific control events**, such as a `Button` control's **`Click` event** or a `TextBox` control's **`TextChanged` event**.
3. In a postback request, if the page contains validator controls, check the `IsValid` property of the Page and of individual validation controls before performing any processing.

*This is just an example of a control event. Here it is the button click event that caused the postback.*

```cs
protected void Button1_Click(object sender, EventArgs e)
{
  
}
```

### LoadComplete

1. Raised at the end of the event-handling stage.
2. Use this event **for tasks that require that all other controls on the page be loaded**.

```cs
protected void Page_LoadComplete(object sender, EventArgs e)
{
  
}
```

### OnPreRender

1. Raised after the Page object has created all controls that are required in order to render the page, including child controls of composite controls.
2. The Page object **raises the `PreRender` event on the Page object**, and **then recursively does the same for each child control**.
3. The `PreRender` event **of individual controls occurs after the `PreRender` event of the page**.
4. Allows final changes to the page or its control.
5. This event **takes place before saving `ViewState`**, so **any changes made here are saved**.
6. For example: After this event, you cannot change any property of a button or change any viewstate value.
7. Each data bound control whose `DataSourceID` property is set calls its `DataBind` method.
8. **Use the event to make final changes to the contents of the page or its controls**.

```cs
protected override void OnPreRender(EventArgs e)
{
  
}
```

### OnSaveStateComplete

1. Raised **after view state and control state have been saved for the page and for all controls**.
2. Before this event occurs, `ViewState` has been saved for the page and for all controls.
3. **Any changes to the page or controls at this point will be ignored**.
4. Use this event **perform tasks that require the view state to be saved, but that do not make any changes to controls**.

```cs
protected override void OnSaveStateComplete(EventArgs e)
{
  
}
```

### Render Method

1. This is a **method** of the page object and its controls *(and **not an event**)*.
2. The `Render` method **generates** the **client-side HTML**, **Dynamic Hypertext Markup Language** (DHTML), and **script that are necessary to properly display a control** at the browser.

### UnLoad

1. This event is used **for cleanup code**.
2. At this point, **all processing has occurred** and **it is safe to dispose of any remaining objects**, **including the Page object**.
3. Cleanup can be performed on:
   - Instances of classes, in other words objects
   - Closing opened files
   - Closing database connections
4. This event **occurs for each control** and **then for the page**.
5. During the unload stage, **the page and its controls have been rendered**, so you **cannot make further changes to the response stream**.
6. If you attempt to call a method such as the `Response.Write` method then the page will throw an exception.

```cs
protected void Page_UnLoad(object sender, EventArgs e)
{
  
}
```

### Example

```cs
public partial class PageLiftCycle: System.Web.UI.Page
{ 
  protected void Page_PreInit(object sender, EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "PreInit";  
  }  

  protected void Page_Init(object sender, EventArgs e)
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "Init";  
  }  

  protected void Page_InitComplete(object sender, EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "InitComplete";  
  }  

  protected override void OnPreLoad(EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    //If the page is post back, then label control values will be loaded from view state.  
    //E.g: If you string str = label.Text, then str will contain viewstate values.  
    label.Text += "<br/>" + "PreLoad";  
  }  

  protected void Page_Load(object sender, EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "Load";  
  }

  protected void SubmitButton_Click(object sender, EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "SubmitButton_Click";  
  }

  protected void Page_LoadComplete(object sender, EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "LoadComplete";  
  }

  protected override void OnPreRender(EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    label.Text += "<br/>" + "PreRender";  
  }

  protected override void OnSaveStateComplete(EventArgs e) 
  {  
    //Work and It will assign the values to label.  
    //But "SaveStateComplete" values will not be available during post back. i.e. View state.  
    label.Text += "<br/>" + "SaveStateComplete";  
  }

  protected void Page_UnLoad(object sender, EventArgs e) 
  {  
    //Work and it will not effect label contrl, view stae and post back data.  
    label.Text += "<br/>" + "UnLoad";  
  }  
}   
```

The first time the Page Load is output:

- PreInit
- Init
- InitComplete
- PreLoad
- Load
- LoadComplete
- PreRender
- SaveStateComplete

When you click on the Submit Button output:

- PreInit
- Init
- InitComplete
- PreLoad
- Load
- LoadComplete
- PreRender
- **PreLoad**
- **Load**
- **SubmitButton_Click**
- **LoadComplete**
- **PreRender**
- SaveStateComplete

During the first time of the page load with `EnableViewState="false"`:

- PreInit
- Init
- InitComplete
- PreLoad
- Load
- LoadComplete
- PreRender
- SaveStateComplete

When you click on the Submit Button output with `EnableViewState="false"`:

- PreInit
- Init
- InitComplete
- PreLoad
- Load
- **SubmitButton_Click**
- LoadComplete
- PreRender
- SaveStateComplete

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
