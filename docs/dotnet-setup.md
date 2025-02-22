---
id: dotnet-setup
title: .NET SDK Instrumentation
sidebar_label: .NET
---

This page will dive into the nitty gritty details on installing Rookout under various configurations.  
If you are encountering any difficulties with deploying Rookout, this is the place to look.

## .NET

The [.NET SDK](https://www.nuget.org/packages/Rookout) provides the ability to fetch debug data from a running application in real time.  

It can easily be installed as a [NuGet package](https://www.nuget.org/packages/Rookout).

## Setup

Start the SDK within your application, by adding the following to your *main* method or your application's entry point:

<!--DOCUSAURUS_CODE_TABS-->
<!--C#-->
```cs
using Rook;
namespace Program
{
    class Program
    {
        static int Main(string[] args)
        {
            Rook.RookOptions options = new Rook.RookOptions() 
            {
                token = "[Your Rookout Token]",
                labels = new Dictionary<string, string> { { "env", "dev" } }
            };
            Rook.API.Start(options);
    
            // ...
        }
    }
}
```
<!--VB.NET-->
```vs
Imports Rook

Module Program
    Sub Main(args As String())
        Dim opts = New RookOptions()
        opts.token = "[Your Rookout Token]"
        opts.labels = New Dictionary(Of String, String)()
        opts.labels.Add("env", "dev")
        Rook.API.Start(opts)
    
        //......
    
    End Sub
End Module
```
<!--F#-->
```fs
open System
open Rook
open System.Collections.Generic

[<EntryPoint>]
let main argv =
    let labels = new Dictionary<string, string>()
    labels.Add("env", "dev")
  
    let opt = Rook.RookOptions(token="[Your Rookout Token]", labels=labels)
    Rook.API.Start(opt)

    // .....

```
<!--END_DOCUSAURUS_CODE_TABS-->
<div class="rookout-org-info"></div>

Check out the [debug information](#debug-information), [source information](#source-information), and [packaging-sources](#packaging-sources) sections for recommendations on how to configure the build process.


## SDK API

### start

```cs
public static void Start(RookOptions opts)
```

The Start method is used to initialize the SDK in the background and accepts a `RookOptions` object with the following attributes:

| Argument &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Environment Variable &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Default Value | Description |
| ------------ | ----------------------- | ------------- | ----------- |
| `token` | `ROOKOUT_TOKEN` | None | The Rookout token for your organization. Should be left empty if you are using a Rookout ETL Controller |
| `labels` | `ROOKOUT_LABELS` | {} | A dictionary of key:value labels for your application instances. Use `k:v,k:v` format for environment variables |
| `git_commit` | `ROOKOUT_COMMIT` | None | String that indicates your git commit or a branch name |
| `git_origin` | `ROOKOUT_REMOTE_ORIGIN` | None | String that indicates your git remote origin |
| `host` | `ROOKOUT_CONTROLLER_HOST` | None | If you are using a Rookout ETL Controller, this is the hostname for it |
| `port` | `ROOKOUT_CONTROLLER_PORT` | None | If you are using a Rookout ETL Controller, this is the port for it |
| `proxy` | `ROOKOUT_PROXY` | None | URL to proxy server
| `debug` | `ROOKOUT_DEBUG` | False | Set to `True` to increase log level to debug |

## Test connectivity

To make sure the SDK was properly installed and test your configuration (environment variables only), download and run TestConnectivity:
* [Windows .Net Framework](https://get.rookout.com/test_connectivity_windows_x64_framework.zip)
* [.Net Core](https://get.rookout.com/test_connectivity_core_x64.zip)
    
Unix: 
* Core 2.x: `chmod +x ./rookout_test_core_2.x.sh && ./rookout_test_core_2.x.sh`
* Core 3.x: `chmod +x ./rookout_test_core_3.x.sh && ./rookout_test_core_3.x.sh`

Windows: Simply run rookout_test_core_2.x.bat or rookout_test_core_3.x.bat respectively.

## Project Requirements

### Debug type

Rookout requires your application to be built and deployed with debug information in the form of `.pdb` files.

In your project’s settings, set the “debug type” to `portable` like so:

```xml
<DebugType>portable</DebugType>
```

While other “debug types” such as `full` and `pdbonly` may work (but are not recommended), the `embedded` type is not supported at all.

For further reading: https://devblogs.microsoft.com/devops/understanding-symbol-files-and-visual-studios-symbol-settings/

### Optimizations

Disabling compiler optimizations `<Optimize>false</Optimize>` will further improve the debugging experience at a small cost to the application performance.

### Multi-Project Solutions

To support multi-projects Solutions its recommended to add the following `Directory.Build.props` file to your Root folder:
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">

  <ItemGroup>
    <PackageReference Include="Rookout" Version="0.1.*" />
  </ItemGroup>

</Project>
```

### Dynamic library loading

To be able to debug libraries loaded using [`AppDomain.Load(Byte[], Byte[])`](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain.load?view=netcore-3.1#System_AppDomain_Load_System_Byte___System_Byte___) make sure to load those binaries into Rookout using:

```cs
Rook.API.LoadAssembly(Assembly a, byte[] pdb, byte[] assembly)
```

## Source information

To enable automatic source fetching, information about the source control must be specified.

### Environment Variables or Start Parameters

Use the environment variables or start parameters as described above in the API section. 

### Git Folder

If the .git folder is present at the root of your project, and no environment variables / start parameters are set, then the source information will be taken from it.

### MSBuildGitHash Package

Use the MSBuildGitHash package to embed the Git information to your application binary.

Note that for this method, the git binary is required to be installed on the build machine.

After installing the [MSBuildGitHash NuGet package](https://www.nuget.org/packages/MSBuildGitHash) add the following line in the .csproj file:

```xml
    <MSBuildGitHashCommand>git config --get remote.origin.url %26%26 git rev-parse HEAD</MSBuildGitHashCommand>
```

## Supported Versions

| Implementation      | Versions              |
| ------------------  | -------------         |
| **.NET Framework**  | 4.5, 4.6, 4.7, 4.8    |
| **.NET Core**       | 2.1, 2.2, 3.0, 3.1    |
| **.NET**            | 5.0                   |

## Supported Languages

The following languages are officially supported: C#, VB.NET and F#.

## IIS support

We currently support IIS 8.0 and above.

If the environment you are trying to debug is not mentioned in the list above, be sure to let us know: {@inject: supportEmail}


## Serverless and PaaS deployments

### Integrating with AWS Lambda

When integrating Rookout into an application running on AWS Lambda, you should explicitly flush the collected information once lambda execution concludes.  
Rookout provides an easy to use wrapper - wrap your code with `using (Rook.API.StartLambda(options))` as in the example below:

```cs
using Rook;

namespace LambdaExample
{
    public class Function
    {

        public string FunctionHandler(string input, ILambdaContext context)
        {
            Rook.RookOptions options = new Rook.RookOptions()
            {
                labels = new Dictionary<string, string> { { "env", "lambda" } }
            };
            using (Rook.API.StartLambda(options))
            {
                /// Your code
                return "Hello World";
            }
        }
    }
}
```

On .NET Core 3 or newer, you can also use `await using` instead of just `using`.

**Note:** Adding the Rookout SDK will slow down your Serverless cold-start times. Please make sure your timeout is no less than 20 seconds.

Refer to the [SDK API](#sdk-api-1) for the available optional options

## Packaging Sources

To make sure you are collecting data from the source line where you have set the breakpoint, include your source files within your library.
```xml
    <EmbedAllSources>true</EmbedAllSources>
```
