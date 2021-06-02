- [Web API](#web-api)
  - [Return complete XML response from ASP .NET Core Web API](#return-complete-xml-response-from-asp-net-core-web-api)

# Web API

## Return complete XML response from ASP .NET Core Web API

> [Source](https://exceptionshub.com/c-return-complete-xml-response-from-asp-net-core-web-api-method.html)

```cs
public void ConfigureServices(IServiceCollection services)
{
  services.AddControllers();
  services.AddMvc(options =>
  {
    options.Filters.Add(new ProducesAttribute("text/xml"));
    options.OutputFormatters.Add(new XmlSerializerOutputFormatter(new XmlWriterSettings
    {
      OmitXmlDeclaration = false
    }));
  }).AddXmlSerializerFormatters();
}
```
