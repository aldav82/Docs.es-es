---
title: Introducción a ASP.NET Core
author: rick-anderson
description: Consulte una introducción a ASP.NET Core, un marco multiplataforma de código abierto y de alto rendimiento que tiene como finalidad compilar modernas aplicaciones conectadas a Internet y basadas en la nube.
ms.author: riande
ms.date: 9/28/2018
uid: index
ms.openlocfilehash: 85f2aa04e4b6692dd03a63f14bcebdec97ee270e
ms.sourcegitcommit: 32f5ee0690604d451f61e9a5c28881c9fcf85738
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/29/2018
ms.locfileid: "47454783"
---
# <a name="introduction-to-aspnet-core"></a>Introducción a ASP.NET Core

Por [Daniel Roth](https://github.com/danroth27), [Rick Anderson](https://twitter.com/RickAndMSFT) y [Shaun Luttin](https://twitter.com/dicshaunary)

ASP.NET Core es un marco multiplataforma de [código abierto](https://github.com/aspnet/home) y de alto rendimiento que tiene como finalidad compilar modernas aplicaciones conectadas a Internet y basadas en la nube. Con ASP.NET Core puede hacer lo siguiente:

* Compilar servicios y aplicaciones web, aplicaciones de [IoT](https://www.microsoft.com/internet-of-things/) y back-ends móviles.
* Usar sus herramientas de desarrollo favoritas en Windows, macOS y Linux.
* Efectuar implementaciones locales y en la nube.
* Ejecutarlo en [.NET Core o en .NET Framework](https://docs.microsoft.com/dotnet/articles/standard/choosing-core-framework-server).

## <a name="why-use-aspnet-core"></a>¿Por qué debería usar ASP.NET Core?

Millones de desarrolladores han usado [ASP.NET 4.x](https://docs.microsoft.com/aspnet/overview) (y siguen usándolo) para crear aplicaciones web. ASP.NET Core es un nuevo diseño de ASP.NET 4.x que cuenta con cambios en la arquitectura que dan como resultado un marco más sencillo y modular.

ASP.NET Core ofrece las siguientes ventajas:

* Un caso unificado para crear API web y una interfaz de usuario web.
* Integración de [marcos del lado cliente modernos](xref:client-side/index) y flujos de trabajo de desarrollo.
* Un [sistema de configuración](xref:fundamentals/configuration/index) basado en el entorno y preparado para la nube.
* [Inserción de dependencias](xref:fundamentals/dependency-injection) integrada.
* Una canalización de solicitudes HTTP ligera, modular y de [alto rendimiento](https://github.com/aspnet/benchmarks).
* Capacidad de hospedarse en [IIS](xref:host-and-deploy/iis/index), [Nginx](xref:host-and-deploy/linux-nginx), [Apache](xref:host-and-deploy/linux-apache), [Docker](xref:host-and-deploy/docker/index) o de autohospedarse en su propio proceso.
* Control de versiones de aplicaciones en paralelo con [.NET Core](https://docs.microsoft.com/dotnet/articles/standard/choosing-core-framework-server) como destino.
* Herramientas que simplifican el desarrollo web moderno.
* Capacidad para compilarse y ejecutarse en Windows, macOS y Linux.
* De código abierto y [centrado en la comunidad](https://live.asp.net/).

## <a name="build-web-apis-and-web-ui-using-aspnet-core-mvc"></a>Creación de API web e interfaces de usuario web mediante ASP.NET Core MVC

ASP.NET Core MVC proporciona características para crear [API web](xref:tutorials/index#build-web-apis) y [aplicaciones web](xref:tutorials/index#build-web-apps):

* El [patrón Modelo-Vista-Controlador (MVC)](xref:mvc/overview) permite que se [puedan hacer pruebas](xref:test/index) en las API web y en las aplicaciones web.
* [Páginas de Razor](xref:razor-pages/index) (novedad de ASP.NET Core 2.0) es un modelo de programación basado en páginas que facilita la compilación de interfaces de usuario web y hace que sea más productiva.
* El [marcado de Razor](xref:mvc/views/razor) proporciona una sintaxis productiva para las [páginas de Razor](xref:razor-pages/index) y las [vistas de MVC](xref:mvc/views/overview).
* Los [asistentes de etiquetas](xref:mvc/views/tag-helpers/intro) permiten que el código de servidor participe en la creación y la representación de elementos HTML en archivos de Razor.
* La compatibilidad integrada para [varios formatos de datos y la negociación de contenidos](xref:web-api/advanced/formatting) permite que las API web lleguen a una amplia gama de clientes, como los exploradores y los dispositivos móviles.
* El [enlace de modelo](xref:mvc/models/model-binding) asigna automáticamente datos de solicitudes HTTP a parámetros de método de acción.
* La [validación de modelos](xref:mvc/models/validation) efectúa una validación del lado cliente y del lado servidor de forma automática.

## <a name="client-side-development"></a>Desarrollo del lado cliente

ASP.NET Core se integra perfectamente con bibliotecas y marcos de trabajo populares del lado cliente, que incluyen [Angular](xref:spa/angular), [React](xref:spa/react) y [Bootstrap](xref:client-side/bootstrap). Para obtener más información, vea [Desarrollo del lado cliente](xref:client-side/index).

## <a name="aspnet-core-targeting-net-framework"></a>ASP.NET Core con .NET Framework como destino

ASP.NET Core puede tener como destino .NET Core o .NET Framework. Las aplicaciones de ASP.NET Core que tienen como destino .NET Framework no son multiplataforma, sino que solo se ejecutan en Windows. No está previsto eliminar la compatibilidad con .NET Framework como destino en ASP.NET Core. Por lo general, ASP.NET Core está formado por bibliotecas de [.NET Standard](/dotnet/standard/net-standard). Las aplicaciones escritas con .NET Standard 2.0 se ejecutan en cualquier lugar en el que se admita .NET Standard 2.0.

El uso de .NET Core como destino cuenta con varias ventajas que van en aumento con cada versión. Entre las ventajas del uso de .NET Core en vez de .NET Framework se incluyen las siguientes:

* Multiplataforma. Ejecución en macOS, Linux y Windows.
* Rendimiento mejorado
* Control de versiones en paralelo.
* Nuevas API.
* Abrir origen

Estamos trabajando intensamente para cerrar la brecha de API entre .NET Framework y .NET Core. El [paquete de compatibilidad de Windows](/dotnet/core/porting/windows-compat-pack) ha permitido que miles de API solo de Windows estén disponibles en .NET Core. Estas API no estaban disponibles en .NET Core 1.x.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los siguientes recursos:

* [Introducción a las páginas de Razor](xref:tutorials/razor-pages/razor-pages-start)
* [Tutoriales de ASP.NET Core](xref:tutorials/index)
* <xref:tutorials/publish-to-azure-webapp-using-vs>
* [Conceptos básicos de ASP.NET Core](xref:fundamentals/index)
* [La reunión semanal de la comunidad de ASP.NET](https://live.asp.net/) trata el progreso y los planes del equipo. Incluye nuevos blogs y nuevo software de terceros.
