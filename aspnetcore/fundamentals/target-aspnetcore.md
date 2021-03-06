---
title: Bir sınıf kitaplığında ASP.NET Core API 'Leri kullanma
author: scottaddie
description: Bir sınıf kitaplığında ASP.NET Core API 'Leri kullanmayı öğrenin.
ms.author: scaddie
ms.custom: mvc
ms.date: 12/16/2019
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/target-aspnetcore
ms.openlocfilehash: 85c0d850922b7118b101126c09b208b0db420f7e
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776493"
---
# <a name="use-aspnet-core-apis-in-a-class-library"></a>Bir sınıf kitaplığında ASP.NET Core API 'Leri kullanma

[Scott Ade](https://github.com/scottaddie) tarafından

Bu belge bir sınıf kitaplığında ASP.NET Core API 'Leri kullanmaya yönelik rehberlik sağlar. Diğer tüm kitaplık Kılavuzu için bkz. [Açık Kaynak Kitaplığı Kılavuzu](/dotnet/standard/library-guidance/).

## <a name="determine-which-aspnet-core-versions-to-support"></a>Hangi ASP.NET Core sürümlerinin destekleyeceğini belirleme

ASP.NET Core [.NET Core destek ilkesine](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)uyar. Kitaplıkta hangi ASP.NET Core sürümlerinin destekleyeceği belirlenirken destek ilkesine başvurun. Bir kitaplık:

* *Uzun süreli destek* (LTS) olarak sınıflandırılan tüm ASP.NET Core sürümlerini desteklemeye yönelik bir çaba oluşturun.
* *Yaşam süresi* (EOL) olarak sınıflandırılan ASP.NET Core sürümlerini desteklemeye yönelik değildir.

ASP.NET Core önizleme sürümleri kullanılabilir hale getirilirse, [ASPNET/Duyurular](https://github.com/aspnet/Announcements/issues) GitHub deposunda son değişiklikler gönderilir. Çerçeve özellikleri geliştirildiği için kitaplıkların Uyumluluk testi yürütülür.

## <a name="use-the-aspnet-core-shared-framework"></a>ASP.NET Core paylaşılan çerçevesini kullanma

.NET Core 3,0 sürümü sayesinde, birçok ASP.NET Core derlemesi artık NuGet 'e paket olarak yayımlanmamıştır. Bunun yerine, derlemeler .NET Core SDK ve çalışma zamanı `Microsoft.AspNetCore.App` yükleyicilerle birlikte yüklenen paylaşılan çerçeveye dahil edilmiştir. Artık yayımlanmayan paketlerin listesi için bkz. [artık kullanılmayan paket başvurularını kaldır](xref:migration/22-to-30#remove-obsolete-package-references).

.NET Core 3,0 itibariyle, `Microsoft.NET.Sdk.Web` MSBuild SDK 'sını kullanan projeler, paylaşılan çerçeveye örtük olarak başvurur. `Microsoft.NET.Sdk` Veya `Microsoft.NET.Sdk.Razor` SDK kullanan projeler, paylaşılan çerçevede ASP.NET Core apı 'leri kullanmak için ASP.NET Core başvurmalıdır.

ASP.NET Core başvurmak için, proje dosyanıza aşağıdaki `<FrameworkReference>` öğeyi ekleyin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj?highlight=8)]

ASP.NET Core buna bu şekilde başvurulması yalnızca .NET Core 3. x 'i hedefleyen projeler için desteklenir.

## <a name="include-blazor-extensibility"></a>Blazor genişletilebilirliği Ekle

Blazor, WebAssembly (ıSSTREAM) ve sunucu [barındırma modellerini](xref:blazor/hosting-models)destekler. Belirli bir neden olmadığı müddetçe, [Razor bileşenleri](xref:blazor/components) Kitaplığı hem barındırma modellerini desteklemelidir. Razor bileşenleri Kitaplığı [Microsoft. net. SDK. Razor SDK 'sını](xref:razor-pages/sdk)kullanmalıdır.

### <a name="support-both-hosting-models"></a>Barındırma modellerini destekler

Hem [Blazor Server](xref:blazor/hosting-models#blazor-server) hem de [Blazor InStream](xref:blazor/hosting-models#blazor-webassembly) projelerinden Razor bileşeni tüketimini desteklemek için, düzenleyiciniz için aşağıdaki yönergeleri kullanın.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**Razor sınıf kitaplığı** proje şablonunu kullanın. Şablonun **destek sayfaları ve görünümleri** onay kutusu seçili olmalıdır.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Tümleşik terminalde aşağıdaki komutu çalıştırın:

```dotnetcli
dotnet new razorclasslib
```

# <a name="visual-studio-for-mac"></a>[Mac için Visual Studio](#tab/visual-studio-mac)

**Razor sınıf kitaplığı** proje şablonunu kullanın.

---

Şablondan oluşturulan proje aşağıdaki işlemleri yapar:

* 2,0 .NET Standard hedefler.
* `RazorLangVersion` Özelliğini olarak `3.0`ayarlar. `3.0`, .NET Core 3. x için varsayılan değerdir.
* Aşağıdaki paket başvurularını ekler:
  * [Microsoft. AspNetCore. Components](https://www.nuget.org/packages/Microsoft.AspNetCore.Components)
  * [Microsoft. AspNetCore. components. Web](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Web)

Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-components-library.csproj)]

### <a name="support-a-specific-hosting-model"></a>Belirli bir barındırma modelini destekleme

Tek bir Blazor barındırma modelini desteklemek çok daha az yaygındır. Örnek olarak, yalnızca [Blazor Server](xref:blazor/hosting-models#blazor-server) projelerinden gelen Razor bileşeni tüketimini desteklemek için:

* Hedef .NET Core 3. x.
* Paylaşılan çerçeve `<FrameworkReference>` için bir öğe ekleyin.

Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-components-library.csproj)]

Razor bileşenleri içeren kitaplıklar hakkında daha fazla bilgi için bkz. [ASP.NET Core Razor bileşenleri sınıf kitaplıkları](xref:blazor/class-libraries).

## <a name="include-mvc-extensibility"></a>MVC genişletilebilirliği Ekle

Bu bölümde şunları içeren kitaplıklara ilişkin öneriler özetlenmektedir:

* [Razor görünümleri veya Razor Pages](#razor-views-or-razor-pages)
* [Etiket Yardımcıları](#tag-helpers)
* [Görünüm bileşenleri](#view-components)

Bu bölüm, MVC 'nin birden çok sürümünü desteklemek için Çoklu hedefleme ile açıklanmamaktadır. Birden çok ASP.NET Core sürümünün desteklenmesi hakkında rehberlik için bkz. [çoklu ASP.NET Core sürümlerini](#support-multiple-aspnet-core-versions)destekleme.

### <a name="razor-views-or-razor-pages"></a>Razor görünümleri veya Razor Pages

[Razor görünümlerini](xref:mvc/views/overview) veya [Razor Pages](xref:razor-pages/index) Içeren bir proje, [Microsoft. net. SDK. Razor SDK 'sını](xref:razor-pages/sdk)kullanmalıdır.

Proje .NET Core 3. x 'i hedefliyorsa şunları gerektirir:

* Olarak `AddRazorSupportForMvc` `true`ayarlanan MSBuild özelliği.
* Paylaşılan `<FrameworkReference>` çerçeve için bir öğe.

**Razor sınıf kitaplığı** proje şablonu, .NET Core 3. x 'i hedefleyen projeler için önceki gereksinimleri karşılar. Düzenleyiciniz için aşağıdaki yönergeleri kullanın.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

**Razor sınıf kitaplığı** proje şablonunu kullanın. Şablonun **destek sayfaları ve görünümleri** onay kutusu seçilmelidir.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Tümleşik terminalde aşağıdaki komutu çalıştırın:

```dotnetcli
dotnet new razorclasslib -s
```

# <a name="visual-studio-for-mac"></a>[Mac için Visual Studio](#tab/visual-studio-mac)

Şu anda proje şablonu desteği yok.

---

Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-views-pages-library.csproj)]

Proje bunun yerine .NET Standard hedefliyorsa, bir [Microsoft. AspNetCore. Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc) paket başvurusu gerekir. `Microsoft.AspNetCore.Mvc` Paket ASP.NET Core 3,0 ' de paylaşılan çerçeveye taşındı ve bu nedenle artık yayımlanmamıştır. Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-views-pages-library.csproj?highlight=8)]

### <a name="tag-helpers"></a>Etiket Yardımcıları

[Etiket Yardımcıları](xref:mvc/views/tag-helpers/intro) içeren bir projenin `Microsoft.NET.Sdk` SDK kullanması gerekir. .NET Core 3. x ' i hedefliyorsanız, `<FrameworkReference>` paylaşılan çerçeve için bir öğe ekleyin. Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

.NET Standard hedefliyorsanız (ASP.NET Core 3. x sürümünden önceki sürümleri desteklemek için), [Microsoft. AspNetCore. MvcRazor](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor)öğesine bir paket başvurusu ekleyin. `Microsoft.AspNetCore.Mvc.Razor` Paket, paylaşılan çerçeveye taşındı ve bu nedenle artık yayınlanmadı. Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-tag-helpers-library.csproj)]

### <a name="view-components"></a>Görünüm bileşenleri

[Görünüm bileşenlerini](xref:mvc/views/view-components) içeren bir projenin `Microsoft.NET.Sdk` SDK kullanması gerekir. .NET Core 3. x ' i hedefliyorsanız, `<FrameworkReference>` paylaşılan çerçeve için bir öğe ekleyin. Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

.NET Standard hedefliyorsanız (ASP.NET Core 3. x sürümünden önceki sürümleri desteklemek için), [Microsoft. AspNetCore. Mvc. ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures)öğesine bir paket başvurusu ekleyin. `Microsoft.AspNetCore.Mvc.ViewFeatures` Paket, paylaşılan çerçeveye taşındı ve bu nedenle artık yayınlanmadı. Örneğin:

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-view-components-library.csproj)]

## <a name="support-multiple-aspnet-core-versions"></a>Çoklu ASP.NET Core sürümlerini destekleme

ASP.NET Core birden çok çeşidini destekleyen bir kitaplığı yazmak için Çoklu hedefleme gereklidir. Etiket Yardımcıları kitaplığı 'nın aşağıdaki ASP.NET Core türevlerini desteklemesi gereken bir senaryo düşünün:

* ASP.NET Core 2,1 hedefleme .NET Framework 4.6.1
* .NET Core 2. x 'i hedefleyen 2. x ASP.NET Core
* ASP.NET Core 3. x .NET Core 3. x 'i hedefleme

Aşağıdaki proje dosyası bu değişkenleri `TargetFrameworks` özelliği aracılığıyla destekler:

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

Önceki proje dosyası ile:

* `Markdig` Paket tüm tüketiciler için eklenir.
* [Microsoft. AspNetCore. MvcRazor ](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) öğesine bir başvuru. .NET Framework 4.6.1 veya üzeri ya da .NET Core 2. x ' i hedefleyen tüketiciler için eklenmiştir. Paketin 2.1.0 sürümü, geriye dönük uyumluluk nedeniyle ASP.NET Core 2,2 ile birlikte çalışmaktadır.
* Paylaşılan çerçeveye .NET Core 3. x 'i hedefleyen tüketiciler için başvurulur. `Microsoft.AspNetCore.Mvc.Razor` Paket, paylaşılan çerçeveye dahildir.

Alternatif olarak, .NET Standard 2,0 hem .NET Core 2,1 hem de .NET Framework 4.6.1 hedeflemek yerine hedeflenebilir:

[!code-xml[](target-aspnetcore/samples/multi-tfm/alternative-tag-helpers-library.csproj?highlight=4)]

Önceki proje dosyası ile aşağıdaki uyarılar mevcuttur:

* Kitaplık yalnızca etiket yardımcıları içerdiğinden, ASP.NET Core çalıştığı belirli platformları hedeflemek daha basittir: .NET Core ve .NET Framework. Etiket Yardımcıları, Unity, UWP ve Xamarin gibi diğer .NET Standard 2,0 uyumlu hedef çerçeveler tarafından kullanılamaz.
* .NET Framework .NET Standard 2,0 ' in kullanılması .NET Framework 4.7.2 giderilen bazı sorunları içerir. .NET Framework 4.6.1 'yi hedefleyerek .NET Framework 4.6.1 aracılığıyla 4.7.1 kullanarak tüketicilerle ilgili deneyimi geliştirebilirsiniz.

Kitaplığınızın platforma özgü API 'Leri çağırması gerekiyorsa, .NET Standard yerine belirli .NET uygulamalarını hedefleyin. Daha fazla bilgi için bkz. [Çoklu hedefleme](/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting).

## <a name="use-an-api-that-hasnt-changed"></a>Değiştirilmemiş bir API kullanma

.NET Core 2,2 ' den 3,0 ' e bir ara yazılım kitaplığı yükselttiğiniz bir senaryo düşünün. Kitaplıkta kullanılan ASP.NET Core ara yazılım API 'Leri, ASP.NET Core 2,2 ve 3,0 arasında değişmemiştir. .NET Core 3,0 ' de ara yazılım kitaplığını desteklemeye devam etmek için aşağıdaki adımları uygulayın:

* [Standart kitaplık kılavuzunu](/dotnet/standard/library-guidance/)izleyin.
* Karşılık gelen derleme paylaşılan çerçevede yoksa her bir API 'nin NuGet paketi için bir paket başvurusu ekleyin.

## <a name="use-an-api-that-changed"></a>Değiştirilen bir API kullanma

.NET Core 2,2 ' den .NET Core 3,0 ' ye bir kitaplığı yükseltmekte olduğunuz bir senaryoyu düşünün. Kitaplıkta kullanılan bir ASP.NET Core API 'SI, ASP.NET Core 3,0 ' de bir [son değişikliğe](/dotnet/core/compatibility/breaking-changes) sahiptir. Kitaplığın, tüm sürümlerde bozulmuş API 'yi kullanmadan yeniden yazılabilir olup olmayacağını göz önünde bulundurun.

Kitaplığı yeniden yazabilmeniz için bunu yapın ve paket başvurularıyla daha önceki bir hedef çerçevesini (örneğin, .NET Standard 2,0 veya .NET Framework 4.6.1) hedefleyin.

Kitaplığı yeniden yazamıyoruz aşağıdaki adımları uygulayın:

* .NET Core 3,0 için bir hedef ekleyin.
* Paylaşılan çerçeve `<FrameworkReference>` için bir öğe ekleyin.
* Kodu koşullu olarak derlemek için uygun hedef çerçeve simgesiyle [#if Önişlemci yönergesini](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) kullanın.

Örneğin, HTTP istek ve yanıt akışlarındaki zaman uyumlu okuma ve yazma işlemleri, ASP.NET Core 3,0 itibariyle varsayılan olarak devre dışıdır. ASP.NET Core 2,2, varsayılan olarak zaman uyumlu davranışı destekler. Zaman uyumlu okuma ve yazma işlemlerinin, g/ç 'nin gerçekleştiği durumlarda etkinleştirilmesi gereken bir ara yazılım kitaplığı düşünün. Kitaplık, uygun Önişlemci yönergesinde zaman uyumlu özellikleri etkinleştirmek için kodu içermelidir. Örneğin:

[!code-csharp[](target-aspnetcore/samples/middleware.cs?highlight=9-24)]

## <a name="use-an-api-introduced-in-30"></a>3,0 ' de tanıtılan bir API kullanma

ASP.NET Core 3,0 ' de tanıtılan ASP.NET Core bir API kullanmak istediğinizi düşünün. Aşağıdaki soruları göz önünde bulundurun:

1. Kitaplık işlevi yeni API 'YI gerektiriyor mu?
1. Kitaplık bu özelliği farklı bir şekilde uygulayabilir miyim?

Kitaplık işlevi için API gerekiyorsa ve bu, alt düzey uygulamanın bir yolu yoktur:

* Yalnızca .NET Core 3. x 'i hedefleyin.
* Paylaşılan çerçeve `<FrameworkReference>` için bir öğe ekleyin.

Kitaplık, özelliği farklı bir şekilde uygulayabilirler:

* .NET Core 3. x ' i hedef çerçeve olarak ekleyin.
* Paylaşılan çerçeve `<FrameworkReference>` için bir öğe ekleyin.
* Kodu koşullu olarak derlemek için uygun hedef çerçeve simgesiyle [#if Önişlemci yönergesini](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if) kullanın.

Örneğin, aşağıdaki etiket Yardımcısı ASP.NET Core 3,0 ' de <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> tanıtılan arabirimi kullanır. .NET Core 3,0 ' i hedefleyen tüketiciler, `NETCOREAPP3_0` hedef çerçeve sembolü tarafından tanımlanan kod yolunu yürütür. Etiket Yardımcısı 'nın Oluşturucu parametre türü, .NET Core <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment> 2,1 ve .NET Framework 4.6.1 tüketicileri için olarak değişir. Bu değişiklik gereklidir çünkü ASP.NET Core 3,0 eski olarak `IHostingEnvironment` işaretlendi ve değiştirme olarak `IWebHostEnvironment` önerilmişti.

```csharp
[HtmlTargetElement("script", Attributes = "asp-inline")]
public class ScriptInliningTagHelper : TagHelper
{
    private readonly IFileProvider _wwwroot;

#if NETCOREAPP3_0
    public ScriptInliningTagHelper(IWebHostEnvironment env)
#else
    public ScriptInliningTagHelper(IHostingEnvironment env)
#endif
    {
        _wwwroot = env.WebRootFileProvider;
    }

    // code omitted for brevity
}
```

Aşağıdaki çok hedefli proje dosyası bu etiket yardımcı senaryosunu destekler:

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

## <a name="use-an-api-removed-from-the-shared-framework"></a>Paylaşılan çerçeveden kaldırılmış bir API kullanma

Paylaşılan çerçeveden kaldırılmış bir ASP.NET Core derlemesi kullanmak için, uygun paket başvurusunu ekleyin. ASP.NET Core 3,0 ' de paylaşılan çerçeveden kaldırılan paketlerin listesi için bkz. [artık kullanılmayan paket başvurularını kaldır](xref:migration/22-to-30#remove-obsolete-package-references).

Örneğin, Web API istemcisini eklemek için:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNet.WebApi.Client" Version="5.2.7" />
  </ItemGroup>

</Project>
```

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:razor-pages/ui-class>
* <xref:blazor/class-libraries>
* [.NET uygulama desteği](/dotnet/standard/net-standard#net-implementation-support)
* [.NET destek ilkeleri](https://dotnet.microsoft.com/platform/support/policy)
