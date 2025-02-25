# RazorLight

Use Razor to build templates from Files / EmbeddedResources / Strings / Database or your custom source outside of ASP.NET MVC. No redundant dependencies and workarounds in pair with excellent performance and **.NET Standard 2.0** and **.NET Core 3.0** support.

[![Build Status](https://travis-ci.org/toddams/RazorLight.svg?branch=master)](https://travis-ci.org/toddams/RazorLight)  [![NuGet Pre Release](https://img.shields.io/nuget/vpre/RazorLight.svg?maxAge=2592000?style=flat-square)](https://www.nuget.org/packages/RazorLight/) [![NuGet downloads](https://img.shields.io/nuget/dt/RazorLight.svg)](https://www.nuget.org/packages/RazorLight/) [![Join the chat at https://gitter.im/gitterHQ/gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/Razor-Light)

# Solidarity with Ukraine 
![Ukraine](https://upload.wikimedia.org/wikipedia/commons/thumb/4/49/Flag_of_Ukraine.svg/1200px-Flag_of_Ukraine.svg.png)
Dear friends, my name is Ivan, I am the guy who created this library. I live in Ukraine, and if you are reading this message - I really hope you and your family are safe and healthy. 24 February Russia invaded my country with a series of missle atacks across entire Ukraine, from East to West. They started with destroying military infrastructure, and so-called "special operation", as they call it, in fact is a full scale war against us. They planned for a 3 days Blitzkrieg, but as we are standing for 9 days already, you can clearly see how strong ukrainians are. Citizens of the cities are joining the Teritorial Defence, office workers, waiters, lawyers, software engineers, we are ready to defend our land. **Putin started to bomb civilians** with air strikes and missles from Russian and Belarus. **This is not about military infrastructure anymore**. People are duying! Tomorrow the war can knock to your house, but Ukraine is the border between civilized West and Russia. Please stay safe and thank you for your support!



**I am a volunteer** who is helping our Teritorial Defence (TO) to get radio communication, first aid kits, clothes, etc abroad. All these really important goods are not in stock in Ukraine already, so I am doing my best to find it anywhere in the Europe. We have logistics, we have transport across the border, we have everything. 


## I am begging for your help
Please, don't stand aside, we need your help. If you are able to donate - you are contributing to saving peace, **you are saving human lives**.
I am creating a discord channel, where I will post everything I buy, share the progress and keep a communication with you. Every penny I recieve will go to the Ukrainian Army and TO.


<details>
  <summary>EUR (Inside EU and SEPA)</summary>
Account holder: Ivan Balan Balan  
 
BIC: TRWIBEB1XXX  
IBAN: BE69 9672 7063 1578  
Address:  
Avenue Louise 54, Room S52
Brussels
1050
Belgium
</details>

<details>
    <Summary>EUR (Outside EU and SEPA)</Summary>
Account Holder: Ivan Balan Balan  

SWIFT/BIC: TRWIBEB1XXX  
IBAN: BE69 9672 7063 1578  
Address:  
Avenue Louise 54, Room S52
Brussels
1050
Belgium
</details>

<details>
    <summary>USD (Inside the US only)</summary>
Account holder: Ivan Balan Balan

Routing number: 084009519

Account number: 9600002966507311

Account type: Checking

Address:

19 W 24th Street
New York NY 10010
United States
</details>



# Table of contents
- [Quickstart](#quickstart)
- [Template sources](#template-sources)
  * [Files](#file-source)
  * [Embedded resources](#embeddedresource-source)
  * [Database (custom)](#custom-source)
- [Includes (aka Partial)](#includes-aka-partial-views)
- [Encoding](#encoding)
- [Additional metadata references](#additional-metadata-references)
- [Enable Intellisense support](#enable-intellisense-support)
- [FAQ](#faq)

# Quickstart
Install the nuget package using following command:

````
Install-Package RazorLight -Version 2.0.0-rc.6
````

The simplest scenario is to create a template from string. Each template must have a ````templateKey```` that is associated with it, so you can render the same template next time without recompilation.

snippet: simple

To render a compiled template:

snippet: RenderCompiledTemplate

# Template sources

RazorLight can resolve templates from any source, but there are a built-in providers that resolve template source from filesystem and embedded resources.

## File source

When resolving a template from filesystem, templateKey - is a relative path to the root folder, that you pass to RazorLightEngineBuilder.

snippet: FileSource

## EmbeddedResource source

For embedded resource, key - is a namespace and key of the embedded resource relative to root Type. Then root type namespace and templateKey will be combined into YourAssembly.NamespaceOfRootType.Templates.View.cshtml

snippet: EmbeddedResourceSource

## Custom source

If you store your templates in database - it is recommended to create custom RazorLightProject that is responsible for gettings templates source from it. The class will be used to get template source and ViewImports. RazorLight will use it to resolve Layouts, when you specify it inside the template.

````CSharp
var project = new EntityFrameworkRazorProject(new AppDbContext());
var engine = new RazorLightEngineBuilder()
              .UseProject(project)
              .UseMemoryCachingProvider()
              .Build();

// For key as a GUID
string result = await engine.CompileRenderAsync("6cc277d5-253e-48e0-8a9a-8fe3cae17e5b", new { Name = "John Doe" });

// Or integer
int templateKey = 322;
string result = await engine.CompileRenderAsync(templateKey.ToString(), new { Name = "John Doe" });
````

You can find a full sample [here](https://github.com/toddams/RazorLight/tree/master/samples/RazorLight.Samples)


# Includes (aka Partial views)

Include feature is useful when you have reusable parts of your templates you want to share between different views. Includes are an effective way of breaking up large templates into smaller components. They can reduce duplication of template content and allow elements to be reused. *This feature requires you to use the RazorLight Project system, otherwise there is no way to locate the partial.*

````CSharp
@model MyProject.TestViewModel
<div>
    Hello @Model.Title
</div>

@{ await IncludeAsync("SomeView.cshtml", Model); }
````
First argument takes a key of the template to resolve, second argument is a model of the view (can be null)

# Encoding
By the default RazorLight encodes Model values as HTML, but sometimes you want to output them as is. You can disable encoding for specific value using @Raw() function

````CSharp
/* With encoding (default) */

string template = "Render @Model.Tag";
string result = await engine.CompileRenderAsync("templateKey", template, new { Tag = "<html>&" });

Console.WriteLine(result); // Output: &lt;html&gt;&amp

/* Without encoding */

string template = "Render @Raw(Model.Tag)";
string result = await engine.CompileRenderAsync("templateKey", template, new { Tag = "<html>&" });

Console.WriteLine(result); // Output: <html>&
````
In order to disable encoding for the entire document - just set ````"DisableEncoding"```` variable to true
````html
@model TestViewModel
@{
    DisableEncoding = true;
}

<html>
    Hello @Model.Tag
</html>
````

# Enable Intellisense support
Visual Studio tooling knows nothing about RazorLight and assumes, that the view you are using - is a typical ASP.NET MVC template. In order to enable Intellisense for RazorLight templates, you should give Visual Studio a little hint about the base template class, that all your templates inherit implicitly

````CSharp
@using RazorLight
@inherits TemplatePage<MyModel>

<html>
    Your awesome template goes here, @Model.Name
</html>
````
____
![Intellisense](github/autocomplete.png)

# FAQ

## Coding Challenges (FAQ)

### How to use templates from memory without setting a project?

The short answer is, you have to set a project to use the memory caching provider.  The project doesn't have to do anything.  This is by design, as without a project system, RazorLight cannot locate partial views.

:x:
You used to be able to write:

```c#
var razorEngine = new RazorLightEngineBuilder()
.UseMemoryCachingProvider()
.Build();
```
... but this now throws an exception, saying, "`_razorLightProject cannot be null`".

:heavy_check_mark:
```c#
var razorEngine = new RazorLightEngineBuilder()
                .UseEmbeddedResourcesProject(typeof(AnyTypeInYourSolution)) // exception without this (or another project type)
                .UseMemoryCachingProvider()
                .Build();
```
Affects: RazorLight-2.0.0-beta1 and later.

Original Issue: https://github.com/toddams/RazorLight/issues/250

### How to embed an image in an email?

This isn't a RazorLight question, but please see [this StackOverflow answer](https://stackoverflow.com/a/32767496/1040437).

### How to embed css in an email?

This isn't a RazorLight question, but please look into PreMailer.Net.

## Compilation and Deployment Issues (FAQ)

Most problems with RazorLight deal with deploying it on a new machine, in a docker container, etc.  If it works fine in your development environment, read this list of problems to see if it matches yours.

### Additional metadata references
When RazorLight compiles your template - it loads all the assemblies from your entry assembly and creates MetadataReference from it. This is a default strategy and it works in 99% of the time. But sometimes compilation crashes with an exception message like "Can not find assembly My.Super.Assembly2000". In order to solve this problem you can pass additional metadata references to RazorLight.

````CSharp
var metadataReference = MetadataReference.CreateFromFile("path-to-your-assembly")

 var engine = new RazorLightEngineBuilder()
                .UseMemoryCachingProvider()
                .AddMetadataReferences(metadataReference)
                .Build();
````

### I'm getting errors after upgrading to ASP.NET Core 3.0 when using runtime compilation

Please see: https://docs.microsoft.com/en-us/aspnet/core/razor-pages/sdk?view=aspnetcore-3.1#use-the-razor-sdk

> Starting with ASP.NET Core 3.0, MVC Views or Razor Pages aren't served by default if the `RazorCompileOnBuild` or `RazorCompileOnPublish` MSBuild properties in the project file are disabled. Applications must add an explicit reference to the `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` package if the app relies on runtime compilation to process .cshtml files.


### I'm getting a Null Reference Exception after upgrading to RazorLight-2.0.0-beta2 or later.

The most common scenario is that some people were using RazorLight's ability to render raw strings as templates.  While this is still somewhat supported (you can't use advanced features like partial views), what is not supported (right now) is using the caching provider with raw strings.  A workaround is to use a dummy class.

### I'm getting "Cannot find compilation library" when I deploy this library on another server

Add these property groups to your **entry point csproj**.
It has to be the entry point project.  For example: ASP.NET Core web project, .NET Core Console project, etc.

````XML
  <PropertyGroup>
    <!-- This group contains project properties for RazorLight on .NET Core -->
    <PreserveCompilationContext>true</PreserveCompilationContext>
    <MvcRazorCompileOnPublish>false</MvcRazorCompileOnPublish>
    <MvcRazorExcludeRefAssembliesFromPublish>false</MvcRazorExcludeRefAssembliesFromPublish>
  </PropertyGroup>
````

### I'm getting "Can't load metadata reference from the entry assembly" exception

Set PreserveCompilationContext to true in your *.csproj file's PropertyGroup tag.

````XML
<PropertyGroup>
    ...
    <PreserveCompilationContext>true</PreserveCompilationContext>
</PropertyGroup>
````

Additionally, RazorLight allows you to specifically locate any `MetadataReference` you can't find, which could happen if you're running in SCD [(Self-Contained Deployment) mode](https://docs.microsoft.com/en-us/dotnet/core/deploying/), as the C# Compiler used by RazorLight [needs to be able to locate `mscorlib.dll`](https://github.com/toddams/RazorLight/issues/188#issuecomment-523418738).  This might be a useful trick if future versions of the .NET SDK tools ship with bad MSBuild targets that somehow don't "preserve compilation context" and you need an immediate fix while waiting for Microsoft support.

### I'm getting "Cannot find reference assembly 'Microsoft.AspNetCore.Antiforgery.dll'" exception on .NET Core App 3.0 or higher

By default, the 3.0 SDK avoids copying references to the build output.
Set `PreserveCompilationReferences` and `PreserveCompilationContext` to true in your *.csproj file's PropertyGroup tag.

````XML
<PropertyGroup>
    <PreserveCompilationReferences>true</PreserveCompilationReferences>
    <PreserveCompilationContext>true</PreserveCompilationContext>
</PropertyGroup>
````

For more information, see https://github.com/aspnet/AspNetCore/issues/14418#issuecomment-535107767 (which discusses the above flags) and https://github.com/microsoft/DockerTools/issues/217#issuecomment-549453362 (which discusses that Runtime Compilation feature was marked obsolete in ASP.NET Core 2.2, and removed from the default template in ASP.NET Core 3.0).

### RazorLight does not work properly on AWS Lambda or Azure Functions

Serverless solutions are not supported yet. However, for Azure Functions, some users have reported success on Azure Functions 3.0.3.  As of 6/3/2020, Azure Functions SDK team has acknowledged a [bug in Azure Functions `RemoveRuntimeDependencies` task](https://github.com/toddams/RazorLight/issues/306#issuecomment-636374491), affecting Azure Functions 3.0.4-3.0.6 releases.

For Azure Functions 3.0.4-3.0.5, the known workaround is to disable "Azure Functions dependency trimming".  To disable dependency trimming, add the following to your root / entrypoint project:

```xml
<PropertyGroup>
  <_FunctionsSkipCleanOutput>true</_FunctionsSkipCleanOutput>
</PropertyGroup>
```

In addition, Azure Functions has an open pull request outstanding to update `runtimeAssemblies.json`: https://github.com/Azure/azure-functions-vs-build-sdk/issues/422

## Unsupported Scenarios

### RazorLight does not work with ASP.NET Core Integration Testing

RazorLight is not currently designed to support such integration tests.  If you need to test your RazorLight tests, current recommendation is to simply create a project called `<YourCompanyName>.<YourProjectName>.Templating` and write your template rendering layer as a Domain Service, and write tests against that service.  Then, you can mock in your integration tests any dependencies on RazorLight.

If you happen to get this working, please let us know what you did.
