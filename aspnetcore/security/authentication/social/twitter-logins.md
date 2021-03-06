---
title: ASP.NET Core ile Twitter dışarıdan oturum açma kurulumu
author: rick-anderson
description: Bu öğreticide, Twitter hesabı kullanıcı kimlik doğrulamasının mevcut bir ASP.NET Core uygulamasına tümleştirilmesi gösterilmektedir.
ms.author: riande
ms.custom: mvc
ms.date: 03/19/2020
monikerRange: '>= aspnetcore-3.0'
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authentication/twitter-logins
ms.openlocfilehash: c6498704214de5e805c9bf57033529d4acc5fd3e
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775797"
---
# <a name="twitter-external-sign-in-setup-with-aspnet-core"></a>ASP.NET Core ile Twitter dışarıdan oturum açma kurulumu

Tarafından [Valeriy Novyıtskyy](https://github.com/01binary) ve [Rick Anderson](https://twitter.com/RickAndMSFT)

Bu örnek, [önceki sayfada](xref:security/authentication/social/index)oluşturulmuş bir örnek ASP.NET Core 3,0 projesi kullanarak kullanıcıların [Twitter hesabıyla oturum açmasını](https://dev.twitter.com/web/sign-in/desktop-browser) nasıl olanaklı hale kullanabileceğinizi gösterir.

## <a name="create-the-app-in-twitter"></a>Uygulamayı Twitter 'da oluşturma

* Projeye [Microsoft. AspNetCore. Authentication. Twitter](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Twitter/3.0.0) NuGet paketini ekleyin.

* Gidin [https://apps.twitter.com/](https://apps.twitter.com/) ve oturum açın. Zaten bir Twitter hesabınız yoksa, oluşturmak için **[Şimdi kaydolun](https://twitter.com/signup)** bağlantısını kullanın.

* **Uygulama oluştur**' u seçin. **Uygulama adı**, **uygulama açıklaması** ve genel **Web sitesi** URI 'sini doldurun (etki alanı adı kaydoluncaya kadar geçici olabilir):

* **Twitter Ile oturum açmayı etkinleştir** seçeneğinin yanındaki kutuyu işaretleyin

* Microsoft. AspNetCore.Identity Kullanıcıların varsayılan olarak bir e-posta adresi olmasını gerektirir. **İzinler** sekmesine gidin, **Düzenle** düğmesine tıklayın ve **kullanıcılardan e-posta adresi iste**' nin yanındaki kutuyu işaretleyin.

* **Geri arama URL 'leri** `https://webapp128.azurewebsites.net/signin-twitter`alanına `/signin-twitter` eklenen geliştirme URI 'nizi girin (örneğin:). Bu örnekte daha sonra yapılandırılan Twitter kimlik doğrulama şeması, OAuth akışını uygulamak için `/signin-twitter` rotadaki istekleri otomatik olarak işler.

  > [!NOTE]
  > URI segmenti `/signin-twitter` Twitter kimlik doğrulama sağlayıcısının varsayılan geri çağırması olarak ayarlanır. Twitter kimlik doğrulama ara [yazılımını, '](/dotnet/api/microsoft.aspnetcore.authentication.twitter.twitteroptions) ın devralınan [remoteauthenticationoptions. callbackpath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath) özelliğini kullanarak YAPıLANDıRıRKEN varsayılan geri çağırma URI 'sini değiştirebilirsiniz.

* Formun geri kalanını doldurun ve **Oluştur**' u seçin. Yeni uygulama ayrıntıları görüntülenir:

## <a name="store-the-twitter-consumer-api-key-and-secret"></a>Twitter tüketici API anahtarını ve gizli dizisini depolayın

Twitter tüketicisi API anahtarı ve gizli dizi [Yöneticisi](xref:security/app-secrets)ile gizli dizi gibi hassas ayarları depolayın. Bu örnek için aşağıdaki adımları kullanın:

1. [Gizli depolamayı etkinleştirme](xref:security/app-secrets#enable-secret-storage)konusundaki yönergeler temelinde projeyi gizli depolama için başlatın.
1. Gizli ayarları gizli anahtar deposunda gizli anahtarları `Authentication:Twitter:ConsumerKey` ve `Authentication:Twitter:ConsumerSecret`bunlarla depolayın:

    ```dotnetcli
    dotnet user-secrets set "Authentication:Twitter:ConsumerAPIKey" "<consumer-api-key>"
    dotnet user-secrets set "Authentication:Twitter:ConsumerSecret" "<consumer-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

Bu belirteçler, yeni bir Twitter uygulaması oluşturduktan sonra **anahtarlar ve erişim belirteçleri** sekmesinde bulunabilir:

## <a name="configure-twitter-authentication"></a>Twitter kimlik doğrulamasını yapılandırma

Twitter hizmetini `ConfigureServices` *Startup.cs* dosyasındaki yöntemine ekleyin:

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupTwitter3x.cs?name=snippet&highlight=10-15)]

[!INCLUDE [default settings configuration](includes/default-settings.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

Twitter kimlik doğrulaması tarafından desteklenen yapılandırma seçenekleri hakkında daha fazla bilgi için, bkz. [dallı ve seçenekleri](/dotnet/api/microsoft.aspnetcore.builder.twitteroptions) API başvurusu. Bu, Kullanıcı hakkında farklı bilgiler istemek için kullanılabilir.

## <a name="sign-in-with-twitter"></a>Twitter ile oturum açma

Uygulamayı çalıştırın ve **oturum aç '** ı seçin. Twitter ile oturum açma seçeneği görünür:

**Twitter** 'ya tıkladığınızda, kimlik doğrulaması için Twitter 'a yeniden yönlendirme yapılır:

Twitter kimlik bilgilerinizi girdikten sonra, e-postanızı ayarlayabileceğiniz Web sitesine geri yönlendirilirsiniz.

Twitter kimlik bilgilerinizi kullanarak oturumunuz açıldı:

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

<!-- 
### React to cancel Authorize External sign-in
Twitter doesn't support AccessDeniedPath
Rather in the twitter setup, you can provide an External sign-in homepage. The external sign-in homepage doesn't support localhost. Tested with https://cors3.azurewebsites.net/ and that works.
-->

## <a name="troubleshooting"></a>Sorun giderme

* **Yalnızca 2. x ASP.NET Core:** Identity ' De `services.AddIdentity` `ConfigureServices`çağırarak yapılandırılmamışsa, kimlik doğrulamaya çalışmak ArgumentException Ile sonuçlanır *: ' signınscheme ' seçeneği sağlanmalıdır*. Bu örnekte kullanılan proje şablonu bunun yapılmasını sağlar.
* Site veritabanı ilk geçiş uygulanarak oluşturulmadıysa, *istek hatasını Işlerken bir veritabanı işlemi başarısız* olur. Veritabanını oluşturmak için **geçişleri Uygula** ' ya dokunun ve hatanın ötesinde devam etmek için yenileyin.

## <a name="next-steps"></a>Sonraki adımlar

* Bu makalede Twitter ile kimlik doğrulaması yapabilirsiniz. [Önceki sayfada](xref:security/authentication/social/index)listelenen diğer sağlayıcılarla kimlik doğrulaması yapmak için benzer bir yaklaşımı izleyebilirsiniz.

* Web sitenizi Azure Web App 'e yayımladığınızda, `ConsumerSecret` Twitter geliştirici portalındaki ' ı sıfırlamanız gerekir.

* `Authentication:Twitter:ConsumerKey` Ve `Authentication:Twitter:ConsumerSecret` Azure Portal uygulama ayarları olarak ayarlayın. Yapılandırma sistemi, ortam değişkenlerinden anahtarları okumak üzere ayarlanır.
