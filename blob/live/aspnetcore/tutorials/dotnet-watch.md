---
title: Desarrollar aplicaciones de ASP.NET Core con dotnet watch
author: rick-anderson
description: "Este tutorial muestra cómo instalar y usar la herramienta de monitor de archivos (dotnet watch) de la CLI de .NET Core en una aplicación de ASP.NET Core."
keywords: ASP.NET Core, usar dotnet watch
ms.author: riande
manager: wpickett
ms.date: 10/05/2017
ms.topic: article
ms.assetid: 563ffb3f-d369-4aa5-bf0a-7300b4e7832c
ms.technology: aspnet
ms.prod: asp.net-core
uid: tutorials/dotnet-watch
ms.openlocfilehash: 1b26beaa41f4b38e0cfd2c8300cb521a3dcce47d
ms.sourcegitcommit: 281f0c614543a6c3db565ea4655b70fe49b61d84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/03/2018
---
# <a name="developing-aspnet-core-apps-using-dotnet-watch"></a><span data-ttu-id="41d94-104">Desarrollar aplicaciones de ASP.NET Core con dotnet watch</span><span class="sxs-lookup"><span data-stu-id="41d94-104">Developing ASP.NET Core apps using dotnet watch</span></span>

<span data-ttu-id="41d94-105">Por [Rick Anderson](https://twitter.com/RickAndMSFT) y [Victor Hurdugaci](https://twitter.com/victorhurdugaci)</span><span class="sxs-lookup"><span data-stu-id="41d94-105">By [Rick Anderson](https://twitter.com/RickAndMSFT) and [Victor Hurdugaci](https://twitter.com/victorhurdugaci)</span></span>

<span data-ttu-id="41d94-106">`dotnet watch` es una herramienta que ejecuta un comando de la [CLI de .NET Core](/dotnet/core/tools) cuando se modifican los archivos de código fuente.</span><span class="sxs-lookup"><span data-stu-id="41d94-106">`dotnet watch` is a tool that runs a [.NET Core CLI](/dotnet/core/tools) command when source files change.</span></span> <span data-ttu-id="41d94-107">Por ejemplo, un cambio en un archivo puede desencadenar una compilación, una ejecución de prueba o una implementación.</span><span class="sxs-lookup"><span data-stu-id="41d94-107">For example, a file change can trigger compilation, test execution, or deployment.</span></span>

<span data-ttu-id="41d94-108">En este tutorial usaremos una aplicación de API web existente con dos puntos de conexión: uno que devuelva una suma y otro que devuelva un producto.</span><span class="sxs-lookup"><span data-stu-id="41d94-108">In this tutorial, we use an existing Web API app with two endpoints: one that returns a sum and one that returns a product.</span></span> <span data-ttu-id="41d94-109">El método del producto contiene un error que corregiremos en este mismo tutorial.</span><span class="sxs-lookup"><span data-stu-id="41d94-109">The product method contains a bug that we'll fix as part of this tutorial.</span></span>

<span data-ttu-id="41d94-110">Descargue la [aplicación de ejemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/dotnet-watch/sample).</span><span class="sxs-lookup"><span data-stu-id="41d94-110">Download the [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/dotnet-watch/sample).</span></span> <span data-ttu-id="41d94-111">Contiene dos proyectos: *WebApp* (una API web de ASP.NET Core) y *WebAppTests* (pruebas unitarias para la API web).</span><span class="sxs-lookup"><span data-stu-id="41d94-111">It contains two projects: *WebApp* (an ASP.NET Core Web API) and *WebAppTests* (unit tests for the Web API).</span></span>

<span data-ttu-id="41d94-112">En un shell de comandos, vaya a la carpeta *WebApp* y ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="41d94-112">In a command shell, navigate to the *WebApp* folder and run the following command:</span></span>

```console
dotnet run
```

<span data-ttu-id="41d94-113">La salida de la consola muestra mensajes similares al siguiente (indicando que la aplicación se ejecuta y espera solicitudes):</span><span class="sxs-lookup"><span data-stu-id="41d94-113">The console output shows messages similar to the following (indicating that the app is running and awaiting requests):</span></span>

```console
$ dotnet run
Hosting environment: Development
Content root path: C:/Docs/aspnetcore/tutorials/dotnet-watch/sample/WebApp
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

<span data-ttu-id="41d94-114">En un explorador web, vaya a `http://localhost:<port number>/api/math/sum?a=4&b=5`.</span><span class="sxs-lookup"><span data-stu-id="41d94-114">In a web browser, navigate to `http://localhost:<port number>/api/math/sum?a=4&b=5`.</span></span> <span data-ttu-id="41d94-115">Debería ver el resultado de `9`.</span><span class="sxs-lookup"><span data-stu-id="41d94-115">You should see the result of `9`.</span></span>

<span data-ttu-id="41d94-116">Navegue a la API del producto (`http://localhost:<port number>/api/math/product?a=4&b=5`).</span><span class="sxs-lookup"><span data-stu-id="41d94-116">Navigate to the product API (`http://localhost:<port number>/api/math/product?a=4&b=5`).</span></span> <span data-ttu-id="41d94-117">Devuelve `9`, no `20` tal como se esperaría.</span><span class="sxs-lookup"><span data-stu-id="41d94-117">It returns `9`, not `20` as you'd expect.</span></span> <span data-ttu-id="41d94-118">Lo corregiremos más adelante en el tutorial.</span><span class="sxs-lookup"><span data-stu-id="41d94-118">We'll fix that later in the tutorial.</span></span>

## <a name="add-dotnet-watch-to-a-project"></a><span data-ttu-id="41d94-119">Agregar `dotnet watch` a un proyecto</span><span class="sxs-lookup"><span data-stu-id="41d94-119">Add `dotnet watch` to a project</span></span>

1. <span data-ttu-id="41d94-120">Agregue una referencia de paquete `Microsoft.DotNet.Watcher.Tools` al archivo *.csproj*:</span><span class="sxs-lookup"><span data-stu-id="41d94-120">Add a `Microsoft.DotNet.Watcher.Tools` package reference to the *.csproj* file:</span></span>

    ```xml
    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
    </ItemGroup> 
    ```

1. <span data-ttu-id="41d94-121">Instale el paquete `Microsoft.DotNet.Watcher.Tools` mediante la ejecución del comando siguiente:</span><span class="sxs-lookup"><span data-stu-id="41d94-121">Install the `Microsoft.DotNet.Watcher.Tools` package by running the following command:</span></span>
    
    ```console
    dotnet restore
    ```

## <a name="running-net-core-cli-commands-using-dotnet-watch"></a><span data-ttu-id="41d94-122">Ejecución de los comandos de la CLI de .NET Core mediante `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="41d94-122">Running .NET Core CLI commands using `dotnet watch`</span></span>

<span data-ttu-id="41d94-123">Cualquier [comando de la CLI de .NET Core](/dotnet/core/tools#cli-commands) se puede ejecutar con `dotnet watch`.</span><span class="sxs-lookup"><span data-stu-id="41d94-123">Any [.NET Core CLI command](/dotnet/core/tools#cli-commands) can be run with `dotnet watch`.</span></span> <span data-ttu-id="41d94-124">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="41d94-124">For example:</span></span>

| <span data-ttu-id="41d94-125">Comando</span><span class="sxs-lookup"><span data-stu-id="41d94-125">Command</span></span> | <span data-ttu-id="41d94-126">Comando con watch</span><span class="sxs-lookup"><span data-stu-id="41d94-126">Command with watch</span></span> |
| ---- | ----- |
| <span data-ttu-id="41d94-127">dotnet run</span><span class="sxs-lookup"><span data-stu-id="41d94-127">dotnet run</span></span> | <span data-ttu-id="41d94-128">dotnet watch run</span><span class="sxs-lookup"><span data-stu-id="41d94-128">dotnet watch run</span></span> |
| <span data-ttu-id="41d94-129">dotnet run -f netcoreapp2.0</span><span class="sxs-lookup"><span data-stu-id="41d94-129">dotnet run -f netcoreapp2.0</span></span> | <span data-ttu-id="41d94-130">dotnet watch run -f netcoreapp2.0</span><span class="sxs-lookup"><span data-stu-id="41d94-130">dotnet watch run -f netcoreapp2.0</span></span> |
| <span data-ttu-id="41d94-131">dotnet run -f netcoreapp2.0 -- --arg1</span><span class="sxs-lookup"><span data-stu-id="41d94-131">dotnet run -f netcoreapp2.0 -- --arg1</span></span> | <span data-ttu-id="41d94-132">dotnet watch run -f netcoreapp2.0 -- --arg1</span><span class="sxs-lookup"><span data-stu-id="41d94-132">dotnet watch run -f netcoreapp2.0 -- --arg1</span></span> |
| <span data-ttu-id="41d94-133">dotnet test</span><span class="sxs-lookup"><span data-stu-id="41d94-133">dotnet test</span></span> | <span data-ttu-id="41d94-134">dotnet watch test</span><span class="sxs-lookup"><span data-stu-id="41d94-134">dotnet watch test</span></span> |

<span data-ttu-id="41d94-135">Ejecute `dotnet watch run` en la carpeta *WebApp*.</span><span class="sxs-lookup"><span data-stu-id="41d94-135">Run `dotnet watch run` in the *WebApp* folder.</span></span> <span data-ttu-id="41d94-136">La salida de la consola indica que se ha iniciado `watch`.</span><span class="sxs-lookup"><span data-stu-id="41d94-136">The console output indicates `watch` has started.</span></span>

## <a name="making-changes-with-dotnet-watch"></a><span data-ttu-id="41d94-137">Efectuar cambios con `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="41d94-137">Making changes with `dotnet watch`</span></span>

<span data-ttu-id="41d94-138">Asegúrese de que `dotnet watch` se está ejecutando.</span><span class="sxs-lookup"><span data-stu-id="41d94-138">Make sure `dotnet watch` is running.</span></span>

<span data-ttu-id="41d94-139">Corrija el error en el método `Product` de *MathController.cs* para que devuelva el producto y no la suma:</span><span class="sxs-lookup"><span data-stu-id="41d94-139">Fix the bug in the `Product` method of *MathController.cs* so it returns the product and not the sum:</span></span>

```csharp
public static int Product(int a, int b)
{
  return a * b;
} 
```

<span data-ttu-id="41d94-140">Guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="41d94-140">Save the file.</span></span> <span data-ttu-id="41d94-141">La salida de la consola muestra que `dotnet watch` ha detectado un cambio de archivo y ha reiniciado la aplicación.</span><span class="sxs-lookup"><span data-stu-id="41d94-141">The console output indicates that `dotnet watch` detected a file change and restarted the app.</span></span>

<span data-ttu-id="41d94-142">Compruebe que `http://localhost:<port number>/api/math/product?a=4&b=5` devuelve el resultado correcto.</span><span class="sxs-lookup"><span data-stu-id="41d94-142">Verify `http://localhost:<port number>/api/math/product?a=4&b=5` returns the correct result.</span></span>

## <a name="running-tests-using-dotnet-watch"></a><span data-ttu-id="41d94-143">Ejecutar pruebas con `dotnet watch`</span><span class="sxs-lookup"><span data-stu-id="41d94-143">Running tests using `dotnet watch`</span></span>

1. <span data-ttu-id="41d94-144">Vuelva a cambiar el método `Product` de *MathController.cs* para devolver la suma y guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="41d94-144">Change the `Product` method of *MathController.cs* back to returning the sum and save the file.</span></span>
1. <span data-ttu-id="41d94-145">En un shell de comandos, desplácese hasta la carpeta *WebAppTests*.</span><span class="sxs-lookup"><span data-stu-id="41d94-145">In a command shell, navigate to the *WebAppTests* folder.</span></span>
1. <span data-ttu-id="41d94-146">Ejecute `dotnet restore`.</span><span class="sxs-lookup"><span data-stu-id="41d94-146">Run `dotnet restore`.</span></span>
1. <span data-ttu-id="41d94-147">Ejecute `dotnet watch test`.</span><span class="sxs-lookup"><span data-stu-id="41d94-147">Run `dotnet watch test`.</span></span> <span data-ttu-id="41d94-148">La salida que indica que se ha producido un error en una prueba y que el monitor espera cambios de archivos:</span><span class="sxs-lookup"><span data-stu-id="41d94-148">Its output indicates that a test failed and that watcher is awaiting file changes:</span></span>

     ```console
     Total tests: 2. Passed: 1. Failed: 1. Skipped: 0.
     Test Run Failed.
     ```

1. <span data-ttu-id="41d94-149">Corrija el código del método `Product` para que devuelva el producto.</span><span class="sxs-lookup"><span data-stu-id="41d94-149">Fix the `Product` method code so it returns the product.</span></span> <span data-ttu-id="41d94-150">Guarde el archivo.</span><span class="sxs-lookup"><span data-stu-id="41d94-150">Save the file.</span></span>

<span data-ttu-id="41d94-151">`dotnet watch` detecta el cambio de archivo y vuelve a ejecutar las pruebas.</span><span class="sxs-lookup"><span data-stu-id="41d94-151">`dotnet watch` detects the file change and reruns the tests.</span></span> <span data-ttu-id="41d94-152">La salida de la consola indica que se han superado las pruebas.</span><span class="sxs-lookup"><span data-stu-id="41d94-152">The console output indicates the tests passed.</span></span>

## <a name="dotnet-watch-in-github"></a><span data-ttu-id="41d94-153">dotnet-watch en GitHub</span><span class="sxs-lookup"><span data-stu-id="41d94-153">dotnet-watch in GitHub</span></span>

<span data-ttu-id="41d94-154">dotnet-watch forma parte del [repositorio de DotNetTools](https://github.com/aspnet/DotNetTools/tree/dev/src/dotnet-watch) de GitHub.</span><span class="sxs-lookup"><span data-stu-id="41d94-154">dotnet-watch is part of the GitHub [DotNetTools repository](https://github.com/aspnet/DotNetTools/tree/dev/src/dotnet-watch).</span></span>

<span data-ttu-id="41d94-155">En la [sección MSBuild](https://github.com/aspnet/DotNetTools/tree/dev/src/dotnet-watch#msbuild) del [archivo Léame de dotnet-watch](https://github.com/aspnet/DotNetTools/blob/dev/src/dotnet-watch/README.md) se describe cómo se puede configurar dotnet-watch desde el archivo de proyecto de MSBuild que se está inspeccionando.</span><span class="sxs-lookup"><span data-stu-id="41d94-155">The [MSBuild section](https://github.com/aspnet/DotNetTools/tree/dev/src/dotnet-watch#msbuild) of the [dotnet-watch ReadMe](https://github.com/aspnet/DotNetTools/blob/dev/src/dotnet-watch/README.md) explains how dotnet-watch can be configured from the MSBuild project file being watched.</span></span> <span data-ttu-id="41d94-156">El [archivo Léame de dotnet-watch](https://github.com/aspnet/DotNetTools/blob/dev/src/dotnet-watch/README.md) contiene información de dotnet-watch que no se trata en este tutorial.</span><span class="sxs-lookup"><span data-stu-id="41d94-156">The [dotnet-watch ReadMe](https://github.com/aspnet/DotNetTools/blob/dev/src/dotnet-watch/README.md) contains information on dotnet-watch not covered in this tutorial.</span></span>