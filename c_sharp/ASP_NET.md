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
