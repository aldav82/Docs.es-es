---
title: "Implementación de servidor web de HTTP.sys en ASP.NET Core"
author: rick-anderson
description: "Presenta HTTP.sys, un servidor web para ASP.NET Core en Windows. Basado en el controlador Http.Sys en modo kernel, HTTP.sys es una alternativa a Kestrel que puede usarse para una conexión directa a Internet sin IIS."
ms.author: riande
manager: wpickett
ms.date: 08/07/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/servers/httpsys
ms.openlocfilehash: 60301e1e3eb96f51e86ef9f8be61f5fd8a4c009c
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/19/2018
---
# <a name="httpsys-web-server-implementation-in-aspnet-core"></a><span data-ttu-id="100c7-104">Implementación de servidor web de HTTP.sys en ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="100c7-104">HTTP.sys web server implementation in ASP.NET Core</span></span>

<span data-ttu-id="100c7-105">Por [Tom Dykstra](https://github.com/tdykstra) y [Chris Ross](https://github.com/Tratcher)</span><span class="sxs-lookup"><span data-stu-id="100c7-105">By [Tom Dykstra](https://github.com/tdykstra) and [Chris Ross](https://github.com/Tratcher)</span></span>

> [!NOTE]
> <span data-ttu-id="100c7-106">En este tema se aplica solo a ASP.NET Core 2.0 y versiones posteriores.</span><span class="sxs-lookup"><span data-stu-id="100c7-106">This topic applies only to ASP.NET Core 2.0 and later.</span></span> <span data-ttu-id="100c7-107">En versiones anteriores de ASP.NET Core, se denomina HTTP.sys [WebListener](xref:fundamentals/servers/weblistener).</span><span class="sxs-lookup"><span data-stu-id="100c7-107">In earlier versions of ASP.NET Core, HTTP.sys is named [WebListener](xref:fundamentals/servers/weblistener).</span></span>

<span data-ttu-id="100c7-108">HTTP.sys es un [servidor web de ASP.NET Core](index.md) que sólo se ejecuta en Windows.</span><span class="sxs-lookup"><span data-stu-id="100c7-108">HTTP.sys is a [web server for ASP.NET Core](index.md) that runs only on Windows.</span></span> <span data-ttu-id="100c7-109">Se basa en el [controlador de modo de núcleo de Http.Sys](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx).</span><span class="sxs-lookup"><span data-stu-id="100c7-109">It's built on the [Http.Sys kernel mode driver](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx).</span></span> <span data-ttu-id="100c7-110">HTTP.sys es una alternativa a [Kestrel](kestrel.md) que ofrece algunas características que no Kestel.</span><span class="sxs-lookup"><span data-stu-id="100c7-110">HTTP.sys is an alternative to [Kestrel](kestrel.md) that offers some features that Kestel doesn't.</span></span> <span data-ttu-id="100c7-111">**HTTP.sys no puede utilizarse con IIS o IIS Express, ya no es compatible con la [módulo principal de ASP.NET](aspnet-core-module.md).**</span><span class="sxs-lookup"><span data-stu-id="100c7-111">**HTTP.sys can't be used with IIS or IIS Express, as it's incompatible with the [ASP.NET Core Module](aspnet-core-module.md).**</span></span>

<span data-ttu-id="100c7-112">HTTP.sys es compatible con las siguientes características:</span><span class="sxs-lookup"><span data-stu-id="100c7-112">HTTP.sys supports the following features:</span></span>

- [<span data-ttu-id="100c7-113">Autenticación de Windows</span><span class="sxs-lookup"><span data-stu-id="100c7-113">Windows Authentication</span></span>](xref:security/authentication/windowsauth)
- <span data-ttu-id="100c7-114">Uso compartido de puertos</span><span class="sxs-lookup"><span data-stu-id="100c7-114">Port sharing</span></span>
- <span data-ttu-id="100c7-115">HTTPS con SNI</span><span class="sxs-lookup"><span data-stu-id="100c7-115">HTTPS with SNI</span></span>
- <span data-ttu-id="100c7-116">HTTP/2 a través de TLS (Windows 10)</span><span class="sxs-lookup"><span data-stu-id="100c7-116">HTTP/2 over TLS (Windows 10)</span></span>
- <span data-ttu-id="100c7-117">Transmisión de archivos directa</span><span class="sxs-lookup"><span data-stu-id="100c7-117">Direct file transmission</span></span>
- <span data-ttu-id="100c7-118">Las respuestas en caché</span><span class="sxs-lookup"><span data-stu-id="100c7-118">Response caching</span></span>
- <span data-ttu-id="100c7-119">WebSockets (Windows 8)</span><span class="sxs-lookup"><span data-stu-id="100c7-119">WebSockets (Windows 8)</span></span>

<span data-ttu-id="100c7-120">Versiones admitidas de Windows:</span><span class="sxs-lookup"><span data-stu-id="100c7-120">Supported Windows versions:</span></span>

- <span data-ttu-id="100c7-121">Windows 7 y Windows Server 2008 R2 y versiones posteriores</span><span class="sxs-lookup"><span data-stu-id="100c7-121">Windows 7 and Windows Server 2008 R2 and later</span></span>

<span data-ttu-id="100c7-122">[Vea o descargue el código de ejemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/sample) ([cómo descargarlo](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="100c7-122">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="when-to-use-httpsys"></a><span data-ttu-id="100c7-123">Cuándo utilizar HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="100c7-123">When to use HTTP.sys</span></span>

<span data-ttu-id="100c7-124">HTTP.sys es útil para las implementaciones donde es necesario exponer el servidor directamente a Internet sin usar IIS.</span><span class="sxs-lookup"><span data-stu-id="100c7-124">HTTP.sys is useful for deployments where you need to expose the server directly to the Internet without using IIS.</span></span>

![HTTP.sys se comunica directamente con Internet](httpsys/_static/httpsys-to-internet.png)

<span data-ttu-id="100c7-126">Dado que se basa en Http.Sys, HTTP.sys no requiere un servidor proxy inverso para la protección frente a ataques.</span><span class="sxs-lookup"><span data-stu-id="100c7-126">Because it's built on Http.Sys, HTTP.sys doesn't require a reverse proxy server for protection against attacks.</span></span> <span data-ttu-id="100c7-127">Http.Sys es una tecnología consolidada que le protege contra muchos tipos de ataques y proporciona la solidez, la seguridad y la escalabilidad de un servidor web con todas las características.</span><span class="sxs-lookup"><span data-stu-id="100c7-127">Http.Sys is mature technology that protects against many kinds of attacks and provides the robustness, security, and scalability of a full-featured web server.</span></span> <span data-ttu-id="100c7-128">El propio IIS se ejecuta como un agente de escucha HTTP sobre Http.Sys.</span><span class="sxs-lookup"><span data-stu-id="100c7-128">IIS itself runs as an HTTP listener on top of Http.Sys.</span></span> 

<span data-ttu-id="100c7-129">HTTP.sys es una buena elección para implementaciones internas cuando necesite una característica no está disponible en Kestrel, como la autenticación de Windows.</span><span class="sxs-lookup"><span data-stu-id="100c7-129">HTTP.sys is a good choice for internal deployments when you need a feature not available in Kestrel, such as Windows authentication.</span></span>

![HTTP.sys se comunica directamente con la red interna](httpsys/_static/httpsys-to-internal.png)

## <a name="how-to-use-httpsys"></a><span data-ttu-id="100c7-131">Cómo usar HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="100c7-131">How to use HTTP.sys</span></span>

<span data-ttu-id="100c7-132">Este es un resumen de tareas de instalación para el sistema operativo del host y la aplicación de ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="100c7-132">Here's an overview of setup tasks for the host OS and your ASP.NET Core application.</span></span>

### <a name="configure-windows-server"></a><span data-ttu-id="100c7-133">Configurar Windows Server</span><span class="sxs-lookup"><span data-stu-id="100c7-133">Configure Windows Server</span></span>

* <span data-ttu-id="100c7-134">Instale la versión de .NET que requiera la aplicación, como [.NET Core](https://www.microsoft.com/net/download/core) o [.NET Framework](https://www.microsoft.com/net/download/framework).</span><span class="sxs-lookup"><span data-stu-id="100c7-134">Install the version of .NET that your application requires, such as [.NET Core](https://www.microsoft.com/net/download/core) or [.NET Framework](https://www.microsoft.com/net/download/framework).</span></span>

* <span data-ttu-id="100c7-135">Registrar previamente los prefijos de dirección URL para enlazar con HTTP.sys y configurar los certificados de SSL</span><span class="sxs-lookup"><span data-stu-id="100c7-135">Preregister URL prefixes to bind to HTTP.sys, and set up SSL certificates</span></span>

   <span data-ttu-id="100c7-136">Si no registrar previamente los prefijos de dirección URL en Windows, tendrá que ejecutar la aplicación con privilegios de administrador.</span><span class="sxs-lookup"><span data-stu-id="100c7-136">If you don't preregister URL prefixes in Windows, you have to run your application with administrator privileges.</span></span> <span data-ttu-id="100c7-137">La única excepción es si enlazará a localhost mediante HTTP (HTTPS no) con un número de puerto mayor que 1024; en ese caso, no se requiere privilegios de administrador.</span><span class="sxs-lookup"><span data-stu-id="100c7-137">The only exception is if you bind to localhost using HTTP (not HTTPS) with a port number greater than 1024; in that case, administrator privileges aren't required.</span></span>

   <span data-ttu-id="100c7-138">Para obtener más información, consulte [cómo registrar previamente prefijos y configurar SSL](#preregister-url-prefixes-and-configure-ssl) más adelante en este artículo.</span><span class="sxs-lookup"><span data-stu-id="100c7-138">For details, see [How to preregister prefixes and configure SSL](#preregister-url-prefixes-and-configure-ssl) later in this article.</span></span>

* <span data-ttu-id="100c7-139">Abrir puertos del firewall para permitir el tráfico llegar a HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="100c7-139">Open firewall ports to allow traffic to reach HTTP.sys.</span></span>

   <span data-ttu-id="100c7-140">Puede usar *netsh.exe* o [cmdlets de PowerShell](https://technet.microsoft.com/library/jj554906).</span><span class="sxs-lookup"><span data-stu-id="100c7-140">You can use *netsh.exe* or [PowerShell cmdlets](https://technet.microsoft.com/library/jj554906).</span></span>

<span data-ttu-id="100c7-141">También hay [configuración del registro de Http.Sys](https://support.microsoft.com/kb/820129).</span><span class="sxs-lookup"><span data-stu-id="100c7-141">There are also [Http.Sys registry settings](https://support.microsoft.com/kb/820129).</span></span>

### <a name="configure-your-aspnet-core-application-to-use-httpsys"></a><span data-ttu-id="100c7-142">Configurar la aplicación de ASP.NET Core para usar HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="100c7-142">Configure your ASP.NET Core application to use HTTP.sys</span></span>

* <span data-ttu-id="100c7-143">No hay ninguna instalación de paquete es necesaria si usa el [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage.</span><span class="sxs-lookup"><span data-stu-id="100c7-143">No package install is needed if you use the [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage.</span></span> <span data-ttu-id="100c7-144">El [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) paquete se incluye en el metapackage.</span><span class="sxs-lookup"><span data-stu-id="100c7-144">The [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) package is included in the metapackage.</span></span>

* <span data-ttu-id="100c7-145">Llame a la `UseHttpSys` método de extensión `WebHostBuilder` en su `Main` método especifica cualquier [HTTP.sys opciones](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs) que necesite, como se muestra en el ejemplo siguiente:</span><span class="sxs-lookup"><span data-stu-id="100c7-145">Call the `UseHttpSys` extension method on `WebHostBuilder` in your `Main` method, specifying any [HTTP.sys options](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs) that you need, as shown in the following example:</span></span>

  [!code-csharp[](httpsys/sample/Program.cs?name=snippet_Main&highlight=11-19)]

### <a name="configure-httpsys-options"></a><span data-ttu-id="100c7-146">Configurar las opciones de HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="100c7-146">Configure HTTP.sys options</span></span>

<span data-ttu-id="100c7-147">Estos son algunos de los límites que se pueden configurar y configuración de HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="100c7-147">Here are some of the HTTP.sys settings and limits that you can configure.</span></span>

<span data-ttu-id="100c7-148">**Conexiones de cliente máximo**</span><span class="sxs-lookup"><span data-stu-id="100c7-148">**Maximum client connections**</span></span>

<span data-ttu-id="100c7-149">Se puede establecer el número máximo de conexiones simultáneas de TCP abiertos en toda la aplicación con el siguiente código en *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="100c7-149">The maximum number of concurrent open TCP connections can be set for the entire application with the following code in *Program.cs*:</span></span>

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Options&highlight=5)]

<span data-ttu-id="100c7-150">El número máximo de conexiones es un número ilimitado de forma predeterminada (null).</span><span class="sxs-lookup"><span data-stu-id="100c7-150">The maximum number of connections is unlimited (null) by default.</span></span>

<span data-ttu-id="100c7-151">**Tamaño del cuerpo de solicitud máximo**</span><span class="sxs-lookup"><span data-stu-id="100c7-151">**Maximum request body size**</span></span>

<span data-ttu-id="100c7-152">El tamaño predeterminado del cuerpo de solicitud máximo es: 30.000.000 bytes, que es aproximadamente 28,6 MB.</span><span class="sxs-lookup"><span data-stu-id="100c7-152">The default maximum request body size is 30,000,000 bytes, which is approximately 28.6MB.</span></span>

<span data-ttu-id="100c7-153">La manera recomendada para invalidar el límite de una aplicación de MVC de ASP.NET Core es usar el [RequestSizeLimit](https://github.com/aspnet/Mvc/blob/rel/2.0.0/src/Microsoft.AspNetCore.Mvc.Core/RequestSizeLimitAttribute.cs) atributo en un método de acción:</span><span class="sxs-lookup"><span data-stu-id="100c7-153">The recommended way to override the limit in an ASP.NET Core MVC app is to use the [RequestSizeLimit](https://github.com/aspnet/Mvc/blob/rel/2.0.0/src/Microsoft.AspNetCore.Mvc.Core/RequestSizeLimitAttribute.cs) attribute on an action method:</span></span>

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

<span data-ttu-id="100c7-154">Este es un ejemplo que muestra cómo configurar la restricción para toda la aplicación, todas las solicitudes:</span><span class="sxs-lookup"><span data-stu-id="100c7-154">Here's an example that shows how to configure the constraint for the entire application, every request:</span></span>

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Options&highlight=6)]

<span data-ttu-id="100c7-155">Puede invalidar la configuración en una solicitud específica en *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="100c7-155">You can override the setting on a specific request in *Startup.cs*:</span></span>

[!code-csharp[](httpsys/sample/Startup.cs?name=snippet_Configure&highlight=9-10)]
 
<span data-ttu-id="100c7-156">Se produce una excepción si intenta configurar el límite de una solicitud después de que la aplicación se ha empezado la lectura de la solicitud.</span><span class="sxs-lookup"><span data-stu-id="100c7-156">An exception is thrown if you try to configure the limit on a request after the application has started reading the request.</span></span> <span data-ttu-id="100c7-157">Hay un `IsReadOnly` propiedad que indica si la `MaxRequestBodySize` propiedad está en estado de solo lectura, lo que significa es demasiado tarde para configurar el límite.</span><span class="sxs-lookup"><span data-stu-id="100c7-157">There's an `IsReadOnly` property that tells you if the `MaxRequestBodySize` property is in read-only state, meaning it's too late to configure the limit.</span></span>

<span data-ttu-id="100c7-158">Para obtener información sobre otras opciones de HTTP.sys, vea [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="100c7-158">For information about other HTTP.sys options, see [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs).</span></span> 

### <a name="configure-urls-and-ports-to-listen-on"></a><span data-ttu-id="100c7-159">Configurar los puertos y las direcciones URL para escuchar en</span><span class="sxs-lookup"><span data-stu-id="100c7-159">Configure URLs and ports to listen on</span></span> 

<span data-ttu-id="100c7-160">De forma predeterminada, ASP.NET Core enlaza a `http://localhost:5000`.</span><span class="sxs-lookup"><span data-stu-id="100c7-160">By default ASP.NET Core binds to `http://localhost:5000`.</span></span> <span data-ttu-id="100c7-161">Para configurar los puertos y los prefijos de dirección URL, puede usar el `UseUrls` método de extensión, el `urls` argumento de línea de comandos, la variable de entorno ASPNETCORE_URLS, o la `UrlPrefixes` propiedad [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs).</span><span class="sxs-lookup"><span data-stu-id="100c7-161">To configure URL prefixes and ports, you can use the `UseUrls` extension method, the `urls` command-line argument, the ASPNETCORE_URLS environment variable, or the `UrlPrefixes` property on [HttpSysOptions](https://github.com/aspnet/HttpSysServer/blob/rel/2.0.0/src/Microsoft.AspNetCore.Server.HttpSys/HttpSysOptions.cs).</span></span> <span data-ttu-id="100c7-162">El siguiente ejemplo de código usa `UrlPrefixes`.</span><span class="sxs-lookup"><span data-stu-id="100c7-162">The following code example uses `UrlPrefixes`.</span></span>

[!code-csharp[](httpsys/sample/Program.cs?name=snippet_Main&highlight=17)]

<span data-ttu-id="100c7-163">Una ventaja de `UrlPrefixes` es que se obtiene un mensaje de error inmediatamente si se intenta agregar un prefijo con el formato incorrecto.</span><span class="sxs-lookup"><span data-stu-id="100c7-163">An advantage of `UrlPrefixes` is that you get an error message immediately if you try to add a prefix that is formatted wrong.</span></span> <span data-ttu-id="100c7-164">Una ventaja de `UseUrls` (compartida con `urls` y ASPNETCORE_URLS) es que resulta más sencillo alternar entre Kestrel y HTTP.sys.</span><span class="sxs-lookup"><span data-stu-id="100c7-164">An advantage of `UseUrls` (shared with `urls` and ASPNETCORE_URLS) is that you can more easily switch between Kestrel and HTTP.sys.</span></span>

<span data-ttu-id="100c7-165">Si utiliza ambos `UseUrls` (o `urls` o ASPNETCORE_URLS) y `UrlPrefixes`, la configuración de `UrlPrefixes` invalidar las de `UseUrls`.</span><span class="sxs-lookup"><span data-stu-id="100c7-165">If you use both `UseUrls` (or `urls` or ASPNETCORE_URLS) and `UrlPrefixes`, the settings in `UrlPrefixes` override the ones in `UseUrls`.</span></span> <span data-ttu-id="100c7-166">Para más información, vea [Hospedaje](xref:fundamentals/hosting).</span><span class="sxs-lookup"><span data-stu-id="100c7-166">For more information, see [Hosting](xref:fundamentals/hosting).</span></span>

<span data-ttu-id="100c7-167">HTTP.sys usa el [formatos de cadena HTTP Server API UrlPrefix](https://msdn.microsoft.com/library/windows/desktop/aa364698.aspx).</span><span class="sxs-lookup"><span data-stu-id="100c7-167">HTTP.sys uses the [HTTP Server API UrlPrefix string formats](https://msdn.microsoft.com/library/windows/desktop/aa364698.aspx).</span></span>

> [!NOTE]
> <span data-ttu-id="100c7-168">Asegúrese de especificar las mismas cadenas de prefijo en `UseUrls` o `UrlPrefixes` que registrar previamente en el servidor.</span><span class="sxs-lookup"><span data-stu-id="100c7-168">Make sure that you specify the same prefix strings in `UseUrls` or `UrlPrefixes` that you preregister on the server.</span></span> 

### <a name="dont-use-iis"></a><span data-ttu-id="100c7-169">No use IIS</span><span class="sxs-lookup"><span data-stu-id="100c7-169">Don't use IIS</span></span>

<span data-ttu-id="100c7-170">Asegúrese de que la aplicación no está configurada para ejecutar IIS o IIS Express.</span><span class="sxs-lookup"><span data-stu-id="100c7-170">Make sure your application isn't configured to run IIS or IIS Express.</span></span>

<span data-ttu-id="100c7-171">En Visual Studio, el perfil de inicio predeterminado es de IIS Express.</span><span class="sxs-lookup"><span data-stu-id="100c7-171">In Visual Studio, the default launch profile is for IIS Express.</span></span> <span data-ttu-id="100c7-172">Para ejecutar el proyecto como una aplicación de consola, cambie manualmente el perfil seleccionado, como se muestra en la siguiente captura de pantalla.</span><span class="sxs-lookup"><span data-stu-id="100c7-172">To run the project as a console application, manually change the selected profile, as shown in the following screen shot.</span></span>

![Seleccione el perfil de aplicación de consola](httpsys/_static/vs-choose-profile.png)

## <a name="preregister-url-prefixes-and-configure-ssl"></a><span data-ttu-id="100c7-174">Registrar previamente prefijos de dirección URL y la configuración de SSL</span><span class="sxs-lookup"><span data-stu-id="100c7-174">Preregister URL prefixes and configure SSL</span></span>

<span data-ttu-id="100c7-175">IIS y HTTP.sys se basan en el controlador de modo kernel de subyacente Http.Sys para escuchar las solicitudes e inicial del procesamiento.</span><span class="sxs-lookup"><span data-stu-id="100c7-175">Both IIS and HTTP.sys rely on the underlying Http.Sys kernel mode driver to listen for requests and do initial processing.</span></span> <span data-ttu-id="100c7-176">En IIS, la interfaz de usuario de administración ofrece una forma es relativamente fácil de configurar todos los elementos.</span><span class="sxs-lookup"><span data-stu-id="100c7-176">In IIS, the management UI gives you a relatively easy way to configure everything.</span></span> <span data-ttu-id="100c7-177">Sin embargo, debe configurar Http.Sys usted mismo.</span><span class="sxs-lookup"><span data-stu-id="100c7-177">However, you need to configure Http.Sys yourself.</span></span> <span data-ttu-id="100c7-178">La herramienta integrada para hacer que es *netsh.exe*.</span><span class="sxs-lookup"><span data-stu-id="100c7-178">The built-in tool for doing that is *netsh.exe*.</span></span> 

<span data-ttu-id="100c7-179">Con *netsh.exe* puede reservar prefijos de dirección URL y asignar los certificados SSL.</span><span class="sxs-lookup"><span data-stu-id="100c7-179">With *netsh.exe* you can reserve URL prefixes and assign SSL certificates.</span></span> <span data-ttu-id="100c7-180">La herramienta requiere privilegios administrativos.</span><span class="sxs-lookup"><span data-stu-id="100c7-180">The tool requires administrative privileges.</span></span>

<span data-ttu-id="100c7-181">En el ejemplo siguiente se muestra el mínimo necesario para reservar los prefijos de dirección URL para los puertos 80 y 443:</span><span class="sxs-lookup"><span data-stu-id="100c7-181">The following example shows the minimum needed to reserve URL prefixes for ports 80 and 443:</span></span>

```console
netsh http add urlacl url=http://+:80/ user=Users
netsh http add urlacl url=https://+:443/ user=Users
```

<span data-ttu-id="100c7-182">En el ejemplo siguiente se muestra cómo asignar un certificado SSL:</span><span class="sxs-lookup"><span data-stu-id="100c7-182">The following example shows how to assign an SSL certificate:</span></span>

```console
netsh http add sslcert ipport=0.0.0.0:443 certhash=MyCertHash_Here appid={00000000-0000-0000-0000-000000000000}"
```

<span data-ttu-id="100c7-183">Aquí está la documentación de referencia de *netsh.exe*:</span><span class="sxs-lookup"><span data-stu-id="100c7-183">Here is the reference documentation for *netsh.exe*:</span></span>

* [<span data-ttu-id="100c7-184">Comandos Netsh para hipertexto transferir protocolo (HTTP)</span><span class="sxs-lookup"><span data-stu-id="100c7-184">Netsh Commands for Hypertext Transfer Protocol (HTTP)</span></span>](https://technet.microsoft.com/library/cc725882.aspx)
* [<span data-ttu-id="100c7-185">Cadenas de UrlPrefix</span><span class="sxs-lookup"><span data-stu-id="100c7-185">UrlPrefix Strings</span></span>](https://msdn.microsoft.com/library/windows/desktop/aa364698.aspx)

<span data-ttu-id="100c7-186">Los siguientes recursos proporcionan instrucciones detalladas para varios escenarios.</span><span class="sxs-lookup"><span data-stu-id="100c7-186">The following resources provide detailed instructions for several scenarios.</span></span> <span data-ttu-id="100c7-187">Artículos que hacen referencia a HttpListener se aplican por igual a HTTP.sys, tal y como se basan en Http.Sys.</span><span class="sxs-lookup"><span data-stu-id="100c7-187">Articles that refer to HttpListener apply equally to HTTP.sys, as both are based on Http.Sys.</span></span>

* [<span data-ttu-id="100c7-188">Configuración de un puerto con un certificado SSL</span><span class="sxs-lookup"><span data-stu-id="100c7-188">How to: Configure a Port with an SSL Certificate</span></span>](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/how-to-configure-a-port-with-an-ssl-certificate)
* <span data-ttu-id="100c7-189">[Comunicación HTTPS - HttpListener en función de hospedaje y la certificación de cliente](http://sunshaking.blogspot.com/2012/11/https-communication-httplistener-based.html) esto es un blog de terceros y es bastante antiguo pero aún tiene información útil.</span><span class="sxs-lookup"><span data-stu-id="100c7-189">[HTTPS Communication - HttpListener based Hosting and Client Certification](http://sunshaking.blogspot.com/2012/11/https-communication-httplistener-based.html) This is a third-party blog and is fairly old but still has useful information.</span></span>
* <span data-ttu-id="100c7-190">[Cómo: Tutorial utilizando HttpListener o servidor Http de código no administrado (C++) como un servidor Simple SSL](https://blogs.msdn.microsoft.com/jpsanders/2009/09/29/how-to-walkthrough-using-httplistener-or-http-server-unmanaged-code-c-as-an-ssl-simple-server/) Esto también es un blog anterior con información útil.</span><span class="sxs-lookup"><span data-stu-id="100c7-190">[How To: Walkthrough Using HttpListener or Http Server unmanaged code (C++) as an SSL Simple Server](https://blogs.msdn.microsoft.com/jpsanders/2009/09/29/how-to-walkthrough-using-httplistener-or-http-server-unmanaged-code-c-as-an-ssl-simple-server/) This too is an older blog with useful information.</span></span>

<span data-ttu-id="100c7-191">Estas son algunas herramientas de terceros que pueden ser más fáciles de usar que los *netsh.exe* línea de comandos.</span><span class="sxs-lookup"><span data-stu-id="100c7-191">Here are some third-party tools that can be easier to use than the *netsh.exe* command line.</span></span> <span data-ttu-id="100c7-192">Estos no son proporcionados por o están aprobados por Microsoft.</span><span class="sxs-lookup"><span data-stu-id="100c7-192">These are not provided by or endorsed by Microsoft.</span></span> <span data-ttu-id="100c7-193">Las herramientas se ejecutan como administrador de forma predeterminada, ya que *netsh.exe* propio requiere privilegios de administrador.</span><span class="sxs-lookup"><span data-stu-id="100c7-193">The tools run as administrator by default, since *netsh.exe* itself requires administrator privileges.</span></span>

* <span data-ttu-id="100c7-194">[http.sys Manager](http://httpsysmanager.codeplex.com/) proporciona la interfaz de usuario de la lista y configurar certificados SSL y opciones, las reservas de prefijo y listas de confianza de certificados.</span><span class="sxs-lookup"><span data-stu-id="100c7-194">[http.sys Manager](http://httpsysmanager.codeplex.com/) provides UI for listing and configuring SSL certificates and options, prefix reservations, and certificate trust lists.</span></span> 
* <span data-ttu-id="100c7-195">[HttpConfig](http://www.stevestechspot.com/ABetterHttpcfg.aspx) permite mostrar o configurar certificados SSL y los prefijos de dirección URL.</span><span class="sxs-lookup"><span data-stu-id="100c7-195">[HttpConfig](http://www.stevestechspot.com/ABetterHttpcfg.aspx) lets you list or configure SSL certificates and URL prefixes.</span></span> <span data-ttu-id="100c7-196">La interfaz de usuario es más refinada que http.sys Manager y expone unas cuantas más opciones de configuración, pero en caso contrario, proporciona una funcionalidad similar.</span><span class="sxs-lookup"><span data-stu-id="100c7-196">The UI is more refined than http.sys Manager and exposes a few more configuration options, but otherwise it provides similar functionality.</span></span> <span data-ttu-id="100c7-197">No se puede crear una nueva lista de confianza de certificados (CTL), pero puede asignar los existentes.</span><span class="sxs-lookup"><span data-stu-id="100c7-197">It cannot create a new certificate trust list (CTL), but can assign existing ones.</span></span>

[!INCLUDE[How to make an SSL cert](../../includes/make-ssl-cert.md)]

## <a name="next-steps"></a><span data-ttu-id="100c7-198">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="100c7-198">Next steps</span></span>

<span data-ttu-id="100c7-199">Para obtener más información, vea los siguientes recursos:</span><span class="sxs-lookup"><span data-stu-id="100c7-199">For more information, see the following resources:</span></span>

* [<span data-ttu-id="100c7-200">Aplicación de ejemplo de este artículo</span><span class="sxs-lookup"><span data-stu-id="100c7-200">Sample app for this article</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/sample)
* [<span data-ttu-id="100c7-201">Código fuente de HTTP.sys</span><span class="sxs-lookup"><span data-stu-id="100c7-201">HTTP.sys source code</span></span>](https://github.com/aspnet/HttpSysServer/)
* [<span data-ttu-id="100c7-202">Hospedar aplicaciones de WPF</span><span class="sxs-lookup"><span data-stu-id="100c7-202">Hosting</span></span>](xref:fundamentals/hosting)