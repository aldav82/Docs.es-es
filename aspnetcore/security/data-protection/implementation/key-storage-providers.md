---
title: Proveedores de almacenamiento de claves en ASP.NET Core
author: rick-anderson
description: Obtenga información acerca de los proveedores de almacenamiento de claves en ASP.NET Core y cómo configurar ubicaciones de almacenamiento de claves.
ms.author: riande
ms.date: 07/16/2018
uid: security/data-protection/implementation/key-storage-providers
ms.openlocfilehash: 0e64a65ab1d65efa9f2e4d36a23663b607f206d7
ms.sourcegitcommit: 9bdba90b2c97a4016188434657194b2d7027d6e3
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/27/2018
ms.locfileid: "47402073"
---
# <a name="key-storage-providers-in-aspnet-core"></a>Proveedores de almacenamiento de claves en ASP.NET Core

El sistema de protección de datos [emplea un mecanismo de detección predeterminada](xref:security/data-protection/configuration/default-settings) para determinar dónde se deben conservar las claves criptográficas. El desarrollador puede invalidar el mecanismo de detección predeterminado y especificar manualmente la ubicación.

> [!WARNING]
> Si especifica una ubicación de persistencia de clave explícita, el sistema de protección de datos anula el registro el cifrado de clave predeterminado en el mecanismo de rest, por lo que las claves ya no se cifran en reposo. Se recomienda que, además [especifica un mecanismo de cifrado de claves explícitas](xref:security/data-protection/implementation/key-encryption-at-rest) para las implementaciones de producción.

## <a name="file-system"></a>Sistema de archivos

Para configurar un repositorio clave basada en el sistema de archivos, llame a la [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem) rutina de configuración como se muestra a continuación. Proporcione un [DirectoryInfo](/dotnet/api/system.io.directoryinfo) que apunta al repositorio donde se deben almacenar las claves:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"c:\temp-keys\"));
}
```

El `DirectoryInfo` puede apuntar a un directorio en el equipo local, o puede señalar a una carpeta en un recurso compartido de red. Si señala a un directorio en el equipo local (y el escenario es que solo las aplicaciones en el equipo local requieren acceso a usar este repositorio), considere el uso de [DPAPI de Windows](xref:security/data-protection/implementation/key-encryption-at-rest) (en Windows) para cifrar las claves en reposo. De lo contrario, considere el uso de un [certificado X.509](xref:security/data-protection/implementation/key-encryption-at-rest) para cifrar las claves en reposo.

## <a name="azure-and-redis"></a>Azure y Redis

El [Microsoft.AspNetCore.DataProtection.AzureStorage](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.AzureStorage/) y [Microsoft.AspNetCore.DataProtection.Redis](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Redis/) paquetes permiten almacenar claves de protección de datos en Azure Storage o en una caché en Redis. Las claves se pueden compartir entre varias instancias de una aplicación web. Las aplicaciones pueden compartir las cookies de autenticación o la protección de CSRF en varios servidores. Para configurar el proveedor de almacenamiento de blobs de Azure, llame a uno de los [PersistKeysToAzureBlobStorage](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.persistkeystoazureblobstorage) sobrecargas:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blob URI including SAS token>"));
}
```

Para configurar Redis, llame a uno de los [PersistKeysToRedis](/dotnet/api/microsoft.aspnetcore.dataprotection.redisdataprotectionbuilderextensions.persistkeystoredis) sobrecargas:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToRedis(redis, "DataProtection-Keys");
}
```

Para obtener más información, vea los temas siguientes:

* [StackExchange.Redis ConnectionMultiplexer](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Basics.md)
* [Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#connect-to-the-cache)
* [ejemplos de ASPNET/DataProtection](https://github.com/aspnet/DataProtection/tree/master/samples)

## <a name="registry"></a>Registro

**Solo se aplica a las implementaciones de Windows.**

A veces, la aplicación podría no tener acceso de escritura al sistema de archivos. Considere un escenario donde se ejecuta una aplicación como una cuenta de servicio virtual (como *w3wp.exe*de identidad del grupo de aplicación). En estos casos, el administrador puede aprovisionar una clave del registro que es accesible mediante la identidad de la cuenta de servicio. Llame a la [PersistKeysToRegistry](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystoregistry) método de extensión tal como se muestra a continuación. Proporcione un [RegistryKey](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.registryxmlrepository.registrykey) apunta a la ubicación donde se deben almacenar las claves criptográficas:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToRegistry(Registry.CurrentUser.OpenSubKey(@"SOFTWARE\Sample\keys"));
}
```

> [!IMPORTANT]
> Se recomienda usar [DPAPI de Windows](xref:security/data-protection/implementation/key-encryption-at-rest) para cifrar las claves en reposo.

::: moniker range=">= aspnetcore-2.2"

## <a name="entity-framework-core"></a>Entity Framework Core

El [Microsoft.AspNetCore.DataProtection.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.EntityFrameworkCore/) paquete proporciona un mecanismo para almacenar las claves de protección de datos a una base de datos mediante Entity Framework Core. El `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` paquete NuGet debe agregarse al archivo de proyecto, no es parte de la [Microsoft.AspNetCore.App metapaquete](xref:fundamentals/metapackage-app).

Con este paquete, las claves se pueden compartir entre varias instancias de una aplicación web.

Para configurar el proveedor de EF Core, llame a la [ `PersistKeysToDbContext<TContext>` ](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcoredataprotectionextensions.persistkeystodbcontext) método:

[!code-csharp[Main](key-storage-providers/sample/Startup.cs?name=snippet&highlight=13-15)]

El parámetro genérico, `TContext`, debe heredar de [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) y [IDataProtectionKeyContext](/dotnet/api/microsoft.aspnetcore.dataprotection.entityframeworkcore.idataprotectionkeycontext):

[!code-csharp[Main](key-storage-providers/sample/MyKeysContext.cs)]

::: moniker-end

## <a name="custom-key-repository"></a>Repositorio de claves personalizado

Si los mecanismos en el equipo no son adecuados, el desarrollador puede especificar su propio mecanismo de persistencia de clave al proporcionar una personalizada [IXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.ixmlrepository).
