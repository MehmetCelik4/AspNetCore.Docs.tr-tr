---
title: ASP.NET Core kimlik doğrulamasına genel bakış
author: mjrousos
description: ASP.NET Core kimlik doğrulaması hakkında bilgi edinin.
ms.author: riande
ms.custom: mvc
ms.date: 03/03/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authentication/index
ms.openlocfilehash: 4e47bd91ce15836035d3e8f0a8ceed264f308b22
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82768643"
---
# <a name="overview-of-aspnet-core-authentication"></a>ASP.NET Core kimlik doğrulamasına genel bakış

, [Mike Rousos](https://github.com/mjrousos) tarafından

Kimlik doğrulaması, bir kullanıcının kimliğini belirleme işlemidir. [Yetkilendirme](xref:security/authorization/introduction) , bir kullanıcının bir kaynağa erişip erişemeyeceğini belirleme işlemidir. ASP.NET Core, kimlik doğrulaması, kimlik doğrulama `IAuthenticationService` [ara yazılımı](xref:fundamentals/middleware/index)tarafından kullanılan tarafından işlenir. Kimlik doğrulama hizmeti, kimlik doğrulama ile ilgili eylemleri tamamlamaya yönelik kayıtlı kimlik doğrulama işleyicilerini kullanır. Kimlik doğrulaması ile ilgili eylemlere örnek olarak şunlar verilebilir:

* Kullanıcı kimlik doğrulaması.
* Kimliği doğrulanmamış bir Kullanıcı kısıtlı bir kaynağa erişmeyi denediğinde yanıt verir.

Kayıtlı kimlik doğrulama işleyicileri ve yapılandırma seçenekleri "şemalar" olarak adlandırılır.

Kimlik doğrulama düzenleri, kimlik doğrulama hizmetleri ' de `Startup.ConfigureServices`kaydedilerek belirtilir:

* Bir çağrısından sonra şemaya özgü genişletme yöntemini çağırarak `services.AddAuthentication` (örneğin, `AddJwtBearer` veya `AddCookie`gibi). Bu uzantı yöntemleri, uygun ayarlarla şemaları kaydetmek için [Authenticationbuilder. AddScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder.AddScheme*) kullanır.
* Daha seyrek, [Authenticationbuilder. AddScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder.AddScheme*) doğrudan çağırarak.

Örneğin, aşağıdaki kod, tanımlama bilgisi ve JWT taşıyıcı kimlik doğrulama şemaları için kimlik doğrulama hizmetlerini ve işleyicileri kaydeder:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options => Configuration.Bind("JwtSettings", options))
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options => Configuration.Bind("CookieSettings", options));
```

`AddAuthentication` Parametresi `JwtBearerDefaults.AuthenticationScheme` , belirli bir şema istenmediğinde varsayılan olarak kullanılacak düzenin adıdır.

Birden çok düzen kullanılırsa, yetkilendirme ilkeleri (veya yetkilendirme öznitelikleri) kullanıcının kimliğini doğrulamak için bağımlı oldukları [kimlik doğrulama düzenini (veya düzenleri) belirtebilir](xref:security/authorization/limitingidentitybyscheme) . Yukarıdaki örnekte, tanımlama bilgisi kimlik doğrulama düzeni adı belirtilerek kullanılabilir (`CookieAuthenticationDefaults.AuthenticationScheme` varsayılan olarak, çağrılırken `AddCookie`farklı bir ad sağlansa da).

Bazı durumlarda, çağrısı `AddAuthentication` diğer uzantı yöntemleri tarafından otomatik olarak yapılır. Örneğin, [ASP.NET Core Identity ](xref:security/authentication/identity) `AddAuthentication` kullanırken dahili olarak çağrılır.

Kimlik doğrulama ara yazılımı, uygulamasının `Startup.Configure` ' de <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> uzantı yöntemi çağırarak öğesine eklenir `IApplicationBuilder`. Çağıran `UseAuthentication` , daha önce kaydedilen kimlik doğrulama düzenlerini kullanan ara yazılımı kaydeder. Kimliği `UseAuthentication` doğrulanan kullanıcılara bağlı olan herhangi bir ara yazılımın önüne çağrı yapın. Endpoint Routing kullanılırken, çağrısının gitmesi `UseAuthentication` gerekir:

* Ardından `UseRouting`, kimlik doğrulama kararlarında yol bilgileri kullanılabilir.
* Önce `UseEndpoints`, kullanıcıların, uç noktalara erişmeden önce kimlik doğrulaması yapılır.

## <a name="authentication-concepts"></a>Kimlik doğrulama kavramları

### <a name="authentication-scheme"></a>Kimlik doğrulama düzeni

Kimlik doğrulama düzeni, öğesine karşılık gelen bir addır:

* Bir kimlik doğrulama işleyicisi.
* İşleyicinin belirli bir örneğini yapılandırma seçenekleri.

Şemaları, ilişkili işleyicinin kimlik doğrulama, sınama ve fordeklarasyonu davranışlarına başvurma mekanizması olarak faydalıdır. Örneğin, bir yetkilendirme ilkesi, kullanıcının kimliğini doğrulamak için hangi kimlik doğrulama şemasının (veya düzenlerinin) kullanılması gerektiğini belirtmek için şema adlarını kullanabilir. Kimlik doğrulamasını yapılandırırken, varsayılan kimlik doğrulama düzenini belirlemek yaygın bir hale gelir. Varsayılan şema, bir kaynak belirli bir düzeni istemediği takdirde kullanılır. Şunları da mümkündür:

* Kimlik doğrulama, sınama ve fordeklarasyon eylemleri için kullanılacak farklı varsayılan şemaları belirtin.
* Birden çok düzeni [ilke düzenlerini](xref:security/authentication/policyschemes)kullanarak bir tek birleştirme.

### <a name="authentication-handler"></a>Kimlik doğrulama işleyicisi

Kimlik doğrulama işleyicisi:

* , Bir düzenin davranışını uygulayan bir türdür.
* Veya <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>' den <xref:Microsoft.AspNetCore.Authentication.IAuthenticationHandler> türetilir.
* , Kullanıcıların kimliğini doğrulamak için Birincil sorumluluğa sahiptir.

Kimlik doğrulama düzeninin yapılandırmasına ve gelen istek bağlamına göre, kimlik doğrulama işleyicileri:

* Kimlik <xref:Microsoft.AspNetCore.Authentication.AuthenticationTicket> doğrulaması başarılı olursa kullanıcının kimliğini temsil eden nesneleri oluşturun.
* Kimlik doğrulaması başarısız olursa ' No Result ' veya ' Failure ' döndürün.
* Kullanıcıların kaynaklara erişmeyi deneyeceği işlemler ve yasaklamaya yönelik yöntemlere sahiptir:
  * Bunlara erişim yetkisi yoktur (fordeklarasyon).
  * Kimliği doğrulanmamış olduğunda (zorluk).

### <a name="authenticate"></a>Kimlik doğrulaması

Kimlik doğrulama düzeninin kimlik doğrulama eylemi, kullanıcının kimliğini istek bağlamına göre oluşturmaktan sorumludur. Kimlik doğrulamanın başarılı <xref:Microsoft.AspNetCore.Authentication.AuthenticateResult> olup olmadığını ve bu durumda kullanıcının kimlik doğrulama biletinde kimliğini belirten bir döndürür. Bkz. <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync%2A>. Kimlik doğrulama örnekleri şunları içerir:

* Tanımlama bilgisi kimlik doğrulama şeması kullanıcının kimlik bilgilerinden kimliğini oluşturan.
* Bir JWT taşıyıcı şeması, kullanıcının kimliğini oluşturmak için bir JWT taşıyıcı belirtecini seri durumdan çıkarma ve doğrulama.

### <a name="challenge"></a>Sınama

Kimliği doğrulanmamış bir kullanıcı kimlik doğrulaması gerektiren bir uç nokta istediğinde yetkilendirme ile kimlik doğrulama sınaması çağrılır. Örneğin, anonim bir Kullanıcı kısıtlı bir kaynak istediğinde veya bir oturum açma bağlantısına tıkladığı zaman bir kimlik doğrulama sınaması verilir. Yetkilendirme, belirtilen kimlik doğrulama düzenlerini kullanarak bir zorluk çağırır ya da hiçbiri belirtilmemişse varsayılan değer. Bkz. <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync%2A>. Kimlik doğrulama sınaması örnekleri şunları içerir:

* Bir tanımlama bilgisi kimlik doğrulama şeması kullanıcıyı bir oturum açma sayfasına yönlendiriyor.
* `www-authenticate: bearer` Üst bilgiyle 401 sonucu döndüren bir JWT taşıyıcı şeması.

Bir sınama eylemi, kullanıcının istenen kaynağa erişmek için hangi kimlik doğrulama mekanizmasını kullanacağınızı bilmesine izin verir.

### <a name="forbid"></a>Yasaklamaz

Kimliği doğrulanmış bir kullanıcı erişimine izin verilmeyen bir kaynağa erişmeyi denediğinde kimlik doğrulama düzeninin fordeklarasyon eylemi yetkilendirme tarafından çağrılır. Bkz. <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync%2A>. Kimlik doğrulama fordeklarasyon örnekleri şunları içerir:
* Bir tanımlama bilgisi kimlik doğrulama şeması, kullanıcıyı erişim yasak bir sayfaya yönlendirdi.
* 403 sonucunu döndüren bir JWT taşıyıcı şeması.
* Özel bir kimlik doğrulama şeması, kullanıcının kaynağa erişim isteyebileceği bir sayfaya yeniden yönlendiriliyor.

Bir fordeklarasyon eylemi, kullanıcının şunları bilmesini sağlayabilir:

* Bunlar doğrulanır.
* İstenen kaynağa erişmelerine izin verilmez.

Challenge ve fordeklarasyon arasındaki farklılıklar için aşağıdaki bağlantılara bakın:

* [İşletimsel kaynak Işleyicisiyle Challenge ve fordeklarasyonu](xref:security/authorization/resourcebased#challenge-and-forbid-with-an-operational-resource-handler).
* [Çekişme ve fordeklarasyon arasındaki farklar](xref:security/authorization/secure-data#challenge).

## <a name="authentication-providers-per-tenant"></a>Kiracı başına kimlik doğrulama sağlayıcıları

ASP.NET Core Framework 'te çok kiracılı kimlik doğrulaması için yerleşik bir çözüm yoktur.
Müşterilerin, yerleşik özellikleri kullanarak bir tane yazabilmesi kesinlikle mümkün olsa da, müşterilerin bu amaçla daha fazla [çekirdeğe](https://www.orchardcore.net/) bakmalarını öneririz.

Orchard Core:

* ASP.NET Core ile oluşturulan açık kaynaklı modüler ve çok kiracılı bir uygulama çerçevesi.
* Bu uygulama çerçevesinin üzerine inşa edilen bir içerik yönetim sistemi (CMS).

Kiracı başına kimlik doğrulama sağlayıcılarının bir örneği için bkz. [Orchard Core](https://github.com/OrchardCMS/OrchardCore) kaynağı.

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authentication/policyschemes>
* <xref:security/authorization/secure-data>
