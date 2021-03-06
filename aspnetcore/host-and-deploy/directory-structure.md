---
title: ASP.NET Core dizin yapısı
author: rick-anderson
description: Yayımlanan ASP.NET Core uygulamalarının dizin yapısı hakkında bilgi edinin.
monikerRange: '>= aspnetcore-2.2'
ms.author: riande
ms.custom: mvc
ms.date: 04/09/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/directory-structure
ms.openlocfilehash: 29031556882dd471a5036b79dcb93a515bc98a33
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776402"
---
# <a name="aspnet-core-directory-structure"></a>ASP.NET Core dizin yapısı

::: moniker range=">= aspnetcore-3.0"

*Yayımla* dizini, uygulamanın [DotNet Publish](/dotnet/core/tools/dotnet-publish) komutu tarafından üretilen dağıtılabilir varlıkları içerir. Dizin şunları içerir:

* Uygulama dosyaları
* Yapılandırma dosyaları
* Statik varlıklar
* Paketler
* Çalışma zamanı (yalnızca[kendi içindeki dağıtım](/dotnet/core/deploying/#self-contained-deployments-scd) )

| Uygulama Türü | Dizin yapısı |
| -------- | ------------------- |
| [Çerçeveye bağımlı yürütülebilir (FDE)](/dotnet/core/deploying/#framework-dependent-executables-fde) | <ul><li>yayınlamanız&dagger;<ul><li>MVC&dagger; uygulamalarını görüntüler; Görünümler önceden derlenmiş değilse</li><li>Sayfalar&dagger; önceden derlenmiş değilse MVC veya Razor Pages uygulamaları</li><li>wwwroot&dagger;</li><li>\*. dll dosyaları</li><li>{ASSEMBLY NAME}. Deps. JSON</li><li>{Bütünleştirilmiş kod adı}. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı} {. Windows üzerinde uzantı}. exe uzantısı, macOS veya Linux üzerinde uzantı yok</li><li>{Bütünleştirilmiş kod adı}. pdb</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. pdb</li><li>{ASSEMBLY NAME}. runtimeconfig. JSON</li><li>Web. config (IIS dağıtımları)</li><li>createdump ([Linux createdump yardımcı programı](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/xplat-minidump-generation.md#configurationpolicy))</li><li>\*. so (Linux paylaşılan nesne kitaplığı)</li><li>\*. a (macOS Arşivi)</li><li>\*. dylib (macOS dinamik kitaplığı)</li></ul></li></ul> |
| [Kendi içinde dağıtım (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) | <ul><li>yayınlamanız&dagger;<ul><li>Görünümler&dagger; önceden derlenmiş değilse MVC uygulamalarını görüntüler</li><li>Sayfalar&dagger; önceden derlenmiş değilse MVC veya Razor Pages uygulamaları</li><li>wwwroot&dagger;</li><li>\*. dll dosyaları</li><li>{ASSEMBLY NAME}. Deps. JSON</li><li>{Bütünleştirilmiş kod adı}. dll</li><li>{Bütünleştirilmiş kod adı}. exe</li><li>{Bütünleştirilmiş kod adı}. pdb</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. pdb</li><li>{ASSEMBLY NAME}. runtimeconfig. JSON</li><li>Web. config (IIS dağıtımları)</li></ul></li></ul> |

&dagger;Bir dizini belirtir

*Yayımla* dizini, dağıtımın *uygulama temel yolu*olarak da adlandırılan *içerik kök yolunu*temsil eder. Sunucuda dağıtılan uygulamanın *Yayımlama* dizinine hangi ad verildiğinde, konumu sunucunun barındırılan uygulamanın fiziksel yolu olarak görev yapar.

Varsa, *Wwwroot* dizini yalnızca statik varlıkları içerir.

## <a name="additional-resources"></a>Ek kaynaklar

* [dotnet publish](/dotnet/core/tools/dotnet-publish)
* [.NET Core uygulama dağıtımı](/dotnet/core/deploying/)
* [Hedef çerçeveler](/dotnet/standard/frameworks)
* [.NET Core RID kataloğu](/dotnet/core/rid-catalog)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

*Yayımla* dizini, uygulamanın [DotNet Publish](/dotnet/core/tools/dotnet-publish) komutu tarafından üretilen dağıtılabilir varlıkları içerir. Dizin şunları içerir:

* Uygulama dosyaları
* Yapılandırma dosyaları
* Statik varlıklar
* Paketler
* Çalışma zamanı (yalnızca[kendi içindeki dağıtım](/dotnet/core/deploying/#self-contained-deployments-scd) )

| Uygulama Türü | Dizin yapısı |
| -------- | ------------------- |
| [Çerçeveye bağımlı yürütülebilir (FDE)](/dotnet/core/deploying/#framework-dependent-executables-fde) | <ul><li>yayınlamanız&dagger;<ul><li>MVC&dagger; uygulamalarını görüntüler; Görünümler önceden derlenmiş değilse</li><li>Sayfalar&dagger; önceden derlenmiş Razor değilse, MVC veya Pages uygulamaları</li><li>wwwroot&dagger;</li><li>\*. dll dosyaları</li><li>{ASSEMBLY NAME}. Deps. JSON</li><li>{Bütünleştirilmiş kod adı}. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı} {. Windows üzerinde uzantı}. exe uzantısı, macOS veya Linux üzerinde uzantı yok</li><li>{Bütünleştirilmiş kod adı}. pdb</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. pdb</li><li>{ASSEMBLY NAME}. runtimeconfig. JSON</li><li>Web. config (IIS dağıtımları)</li><li>createdump ([Linux createdump yardımcı programı](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/xplat-minidump-generation.md#configurationpolicy))</li><li>\*. so (Linux paylaşılan nesne kitaplığı)</li><li>\*. a (macOS Arşivi)</li><li>\*. dylib (macOS dinamik kitaplığı)</li></ul></li></ul> |
| [Kendi içinde dağıtım (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) | <ul><li>yayınlamanız&dagger;<ul><li>Görünümler&dagger; önceden derlenmiş değilse MVC uygulamalarını görüntüler</li><li>Sayfalar&dagger; önceden derlenmiş Razor değilse, MVC veya Pages uygulamaları</li><li>wwwroot&dagger;</li><li>\*. dll dosyaları</li><li>{ASSEMBLY NAME}. Deps. JSON</li><li>{Bütünleştirilmiş kod adı}. dll</li><li>{Bütünleştirilmiş kod adı}. exe</li><li>{Bütünleştirilmiş kod adı}. pdb</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. dll</li><li>{BÜTÜNLEŞTIRILMIŞ KOD ADı}. Views. pdb</li><li>{ASSEMBLY NAME}. runtimeconfig. JSON</li><li>Web. config (IIS dağıtımları)</li></ul></li></ul> |

&dagger;Bir dizini belirtir

*Yayımla* dizini, dağıtımın *uygulama temel yolu*olarak da adlandırılan *içerik kök yolunu*temsil eder. Sunucuda dağıtılan uygulamanın *Yayımlama* dizinine hangi ad verildiğinde, konumu sunucunun barındırılan uygulamanın fiziksel yolu olarak görev yapar.

Varsa, *Wwwroot* dizini yalnızca statik varlıkları içerir.

*Günlük* klasörü oluşturma, [ASP.NET Core modülü gelişmiş hata ayıklama günlüğü](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)için yararlıdır. `<handlerSetting>` Değere verilen yoldaki klasörler modül tarafından otomatik olarak oluşturulmaz ve modülün hata ayıklama günlüğünü yazmasına izin vermek için dağıtımda önceden var olmalıdır.

Aşağıdaki iki yaklaşımdan birini kullanarak dağıtım için bir *günlük* dizini oluşturulabilir:

* Aşağıdaki `<Target>` öğeyi proje dosyasına ekleyin:

   ```xml
   <Target Name="CreateLogsFolder" AfterTargets="Publish">
     <MakeDir Directories="$(PublishDir)Logs" 
              Condition="!Exists('$(PublishDir)Logs')" />
     <WriteLinesToFile File="$(PublishDir)Logs\.log" 
                       Lines="Generated file" 
                       Overwrite="True" 
                       Condition="!Exists('$(PublishDir)Logs\.log')" />
   </Target>
   ```

   `<MakeDir>` Öğesi yayımlanan çıktıda boş bir *Günlükler* klasörü oluşturur. Öğesi, klasörü oluşturmak `PublishDir` için hedef konumu belirlemede özelliğini kullanır. Web Dağıtımı gibi birkaç dağıtım yöntemi, dağıtım sırasında boş klasörleri atlar. `<WriteLinesToFile>` Öğesi *Günlükler* klasöründe, klasörün sunucuya dağıtımını garanti eden bir dosya oluşturur. Çalışan işleminin hedef klasöre yazma erişimi yoksa, bu yaklaşımı kullanarak klasör oluşturma işlemi başarısız olur.

* Dağıtımdaki sunucuda *günlük* dizinini fiziksel olarak oluşturun.

Dağıtım dizini için okuma/yürütme izinleri gerekir. *Günlükler* dizini için okuma/yazma izinleri gerekir. Dosyaların yazıldığı ek dizinler okuma/yazma izinleri gerektirir.

## <a name="additional-resources"></a>Ek kaynaklar

* [dotnet publish](/dotnet/core/tools/dotnet-publish)
* [.NET Core uygulama dağıtımı](/dotnet/core/deploying/)
* [Hedef çerçeveler](/dotnet/standard/frameworks)
* [.NET Core RID kataloğu](/dotnet/core/rid-catalog)

::: moniker-end
