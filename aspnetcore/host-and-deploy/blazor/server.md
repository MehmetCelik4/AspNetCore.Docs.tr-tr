---
title: ASP.NET Core sunucusu barındırma ve dağıtma Blazor
author: guardrex
description: ASP.NET Core kullanarak sunucu uygulamasını nasıl barındırılacağını ve dağıtacağınızı öğrenin Blazor .
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/04/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/blazor/server
ms.openlocfilehash: 8c06d3a4d0d75a3e2fd9f699af38a23833fa8bce
ms.sourcegitcommit: cd73744bd75fdefb31d25ab906df237f07ee7a0a
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/05/2020
ms.locfileid: "84419950"
---
# <a name="host-and-deploy-blazor-server"></a>Sunucu barındırma ve dağıtma Blazor

, [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com)ve [Daniel Roth](https://github.com/danroth27) tarafından

## <a name="host-configuration-values"></a>Ana bilgisayar yapılandırma değerleri

[ Blazor Sunucu uygulamaları](xref:blazor/hosting-models#blazor-server) [genel ana bilgisayar yapılandırma değerlerini](xref:fundamentals/host/generic-host#host-configuration)kabul edebilir.

## <a name="deployment"></a>Dağıtım

[ Blazor Sunucu barındırma modelinin](xref:blazor/hosting-models#blazor-server)kullanımı, Blazor sunucuda bir ASP.NET Core uygulamasının içinden yürütülür. Kullanıcı Arabirimi güncelleştirmeleri, olay işleme ve JavaScript çağrıları bir bağlantı üzerinden işlenir [SignalR](xref:signalr/introduction) .

ASP.NET Core uygulaması barındırabilen bir Web sunucusu gerekiyor. Visual Studio, ** Blazor sunucu uygulaması** proje şablonunu ( `blazorserverside` [DotNet New](/dotnet/core/tools/dotnet-new) komutu kullanılırken şablon) içerir.

## <a name="scalability"></a>Ölçeklenebilirlik

Bir sunucu uygulaması için kullanılabilir altyapıyı en iyi şekilde kullanmasını sağlamak üzere bir dağıtım planlayın Blazor . Sunucu uygulaması ölçeklenebilirliğini karşılamak için aşağıdaki kaynaklara bakın Blazor :

* [BlazorSunucu uygulamalarının temelleri](xref:blazor/hosting-models#blazor-server)
* <xref:security/blazor/server/threat-mitigation>

### <a name="deployment-server"></a>Dağıtım sunucusu

Tek bir sunucunun ölçeklenebilirliğini değerlendirirken (ölçeği büyütme), bir uygulama için kullanılabilir olan bellek büyük olasılıkla uygulamanın kullanıcı talebi arttıkça arttırabileceği ilk kaynaktır. Sunucudaki kullanılabilir bellek şunları etkiler:

* Bir sunucunun destekleyebileceği etkin devre sayısı.
* İstemcide UI gecikme süresi.

Güvenli ve ölçeklenebilir sunucu uygulamaları oluşturma hakkında yönergeler için Blazor bkz <xref:security/blazor/server/threat-mitigation> ..

Her bağlantı hattı en düşük *Merhaba Dünya*stilinde bir uygulama için YAKLAŞıK 250 KB bellek kullanır. Bir devrenin boyutu, uygulamanın koduna ve her bileşenle ilişkili durum bakım gereksinimlerine bağlıdır. Uygulama ve altyapınız için geliştirme sırasında kaynak taleplerini ölçmenizi öneririz, ancak aşağıdaki taban çizgisi, dağıtım hedefini planlamada bir başlangıç noktası olabilir: uygulamanızın 5.000 eşzamanlı kullanıcı desteklemesini istiyorsanız, en az 1,3 GB sunucu belleğini uygulamaya (veya Kullanıcı başına ~ 273 KB) göre bütçeleme yapmayı düşünün.

### <a name="signalr-configuration"></a>SignalRyapılandırmada

BlazorSunucu uygulamaları SignalR , tarayıcıyla iletişim kurmak için ASP.NET Core kullanır. [ SignalR uygulamasının barındırma ve ölçeklendirme koşulları](xref:signalr/publish-to-azure-web-app) için Blazor geçerlidir Sunucu uygulamaları.

BlazorSignalRdaha düşük gecikme süresi, güvenilirlik ve [güvenlik](xref:signalr/security)nedeniyle taşıma olarak WebSockets kullanırken en iyi şekilde işe yarar. Uzun yoklama, SignalR WebSockets kullanılamadığında veya uygulama açıkça uzun yoklamayı kullanacak şekilde yapılandırıldığında kullanılır. Azure App Service dağıtım sırasında, uygulamayı hizmetin Azure portal ayarları içinde kullanmak üzere yapılandırın. Azure App Service için uygulamayı yapılandırma hakkında ayrıntılı bilgi için bkz. [ SignalR yayımlama yönergeleri](xref:signalr/publish-to-azure-web-app).

#### <a name="azure-signalr-service"></a>Azure SignalR hizmeti

Sunucu uygulamaları için [Azure SignalR hizmetini](/azure/azure-signalr) kullanmanızı öneririz Blazor . Hizmet, bir Blazor sunucu uygulamasının ölçeğini çok sayıda eşzamanlı bağlantıya ölçeklendirmeye olanak tanır SignalR . Ayrıca, SignalR hizmetin küresel erişim ve yüksek performanslı veri merkezleri Coğrafya nedeniyle gecikme süresini azaltmaya önemli ölçüde yardımcı olur. Azure hizmetini bir uygulamayı yapılandırmak (ve isteğe bağlı olarak sağlamak) için SignalR :

1. [Prerendering sırasında istemciler aynı sunucuya geri yönlendirildiği](xref:blazor/hosting-models#connection-to-the-server) *yapışkan oturumları*desteklemek için hizmeti etkinleştirin. `ServerStickyMode`Seçeneğini veya yapılandırma değerini olarak ayarlayın `Required` . Genellikle, bir uygulama aşağıdaki yaklaşımlardan **birini** kullanarak yapılandırmayı oluşturur:

   * `Startup.ConfigureServices`:
  
     ```csharp
     services.AddSignalR().AddAzureSignalR(options =>
     {
         options.ServerStickyMode = 
             Microsoft.Azure.SignalR.ServerStickyMode.Required;
     });
     ```

   * Yapılandırma (aşağıdaki yaklaşımlardan **birini** kullanın):
  
     * *appSettings. JSON*:

       ```json
       "Azure:SignalR:ServerStickyMode": "Required"
       ```

     * App Service 'in **Configuration**  >  Azure Portal (**ad**:, değer:) yapılandırma**uygulaması ayarları** `Azure:SignalR:ServerStickyMode` **Value** `Required` .

1. Sunucu uygulaması için Visual Studio 'da bir Azure Apps yayımlama profili oluşturun Blazor .
1. **Azure SignalR hizmet** bağımlılığını profile ekleyin. Azure aboneliğinin uygulamaya atanacak önceden mevcut bir Azure SignalR hizmeti örneği yoksa, yeni bir hizmet örneği sağlamak için **Yeni bir Azure SignalR hizmet örneği oluştur** ' u seçin.
1. Uygulamayı Azure’da yayımlama.

#### <a name="iis"></a>IIS

IIS kullanırken şunları etkinleştirin:

* [IIS üzerinde WebSockets](xref:fundamentals/websockets#enabling-websockets-on-iis).
* [Uygulama Isteği yönlendirme Ile yapışkan oturumlar](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing).

#### <a name="kubernetes"></a>Kubernetes

[Yapışkan oturumlar için aşağıdaki Kubernetes ek açıklamalarını](https://kubernetes.github.io/ingress-nginx/examples/affinity/cookie/)içeren bir giriş tanımı oluşturun:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: <ingress-name>
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "affinity"
    nginx.ingress.kubernetes.io/session-cookie-expires: "14400"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "14400"
```

#### <a name="linux-with-nginx"></a>Nginx ile Linux

SignalRWebSockets 'in düzgün çalışması için, proxy 'nin `Upgrade` ve `Connection` üst bilgilerinin aşağıdaki değerlere ayarlandığını ve şu değerlere eşlenmiş olduğunu doğrulayın `$connection_upgrade` :

* Varsayılan olarak yükseltme üst bilgisi değeri.
* `close`Yükseltme üst bilgisi eksik veya boş.

```
http {
    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen      80;
        server_name example.com *.example.com
        location / {
            proxy_pass         http://localhost:5000;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection $connection_upgrade;
            proxy_set_header   Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
}
```

Daha fazla bilgi için aşağıdaki makaleleri inceleyin:

* [WebSocket proxy 'Si olarak NGıNX](https://www.nginx.com/blog/websocket-nginx/)
* [WebSocket proxy](http://nginx.org/docs/http/websocket.html)
* <xref:host-and-deploy/linux-nginx>

## <a name="linux-with-apache"></a>Apache ile Linux

BlazorLinux 'Ta Apache 'nin arkasındaki bir uygulamayı barındırmak için, `ProxyPass` http ve WebSockets trafiği için yapılandırın.

Aşağıdaki örnekte:

* Kestrel sunucusu konak makinede çalışıyor.
* Uygulama, 5000 numaralı bağlantı noktasında trafiği dinler.

```
ProxyRequests       On
ProxyPreserveHost   On
ProxyPassMatch      ^/_blazor/(.*) http://localhost:5000/_blazor/$1
ProxyPass           /_blazor ws://localhost:5000/_blazor
ProxyPass           / http://localhost:5000/
ProxyPassReverse    / http://localhost:5000/
```

Aşağıdaki modülleri etkinleştirin:

```
a2enmod   proxy
a2enmod   proxy_wstunnel
```

Tarayıcı konsolunda WebSockets hataları olup olmadığını denetleyin. Örnek hatalar:

* Firefox, ws://the-domain-name.tld/_blazor?id=XXX adresinde sunucuyla bağlantı kuramıyor.
* Hata: ' WebSockets ' taşıması başlatılamadı: hata: aktarımda bir hata oluştu.
* Hata: ' LongPolling ' taşıması başlatılamadı: TypeError: this. Transport tanımsız
* Hata: kullanılabilir aktarımlarla sunucuya bağlanılamıyor. WebSockets başarısız oldu
* Hata: bağlantı ' bağlı ' durumunda değilse veri gönderilemez.

Daha fazla bilgi için [Apache belgelerine](https://httpd.apache.org/docs/current/mod/mod_proxy.html)bakın.

### <a name="measure-network-latency"></a>Ölçü ağı gecikmesi

Aşağıdaki örnekte gösterildiği gibi, [js birlikte çalışması](xref:blazor/call-javascript-from-dotnet) ağ gecikmesini ölçmek için kullanılabilir:

```razor
@inject IJSRuntime JS

@if (latency is null)
{
    <span>Calculating...</span>
}
else
{
    <span>@(latency.Value.TotalMilliseconds)ms</span>
}

@code
{
    private DateTime startTime;
    private TimeSpan? latency;

    protected override async Task OnInitializedAsync()
    {
        startTime = DateTime.UtcNow;
        var _ = await JS.InvokeAsync<string>("toString");
        latency = DateTime.UtcNow - startTime;
    }
}
```

Makul bir kullanıcı arabirimi deneyimi için, 250ms veya daha az sayıda sürekli Kullanıcı arabirimi gecikme süresi önerilir.
