---
title: Apache ile Linux üzerinde ASP.NET Core barındırma
author: rick-anderson
description: HTTP trafiğini Kestrel üzerinde çalışan bir ASP.NET Core Web uygulamasına yönlendirmek için CentOS üzerinde bir ters proxy sunucusu olarak Apache 'yi ayarlamayı öğrenin.
monikerRange: '>= aspnetcore-2.1'
ms.author: shboyer
ms.custom: mvc
ms.date: 04/10/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/linux-apache
ms.openlocfilehash: 9f0825f65f316ee4caf67e82fe5812e3a1ae813e
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775914"
---
# <a name="host-aspnet-core-on-linux-with-apache"></a>Apache ile Linux üzerinde ASP.NET Core barındırma

Sağlayan- [Shayne Boyer](https://github.com/spboyer)

Bu kılavuzu kullanarak, HTTP trafiğinin [Kestrel](xref:fundamentals/servers/kestrel) Server üzerinde çalışan bir ASP.NET Core Web uygulamasına yönlendirilmesini sağlamak Için [CentOS 7](https://www.centos.org/) ' de bir ters proxy sunucusu olarak [Apache](https://httpd.apache.org/) 'yi ayarlamayı öğrenin. [Mod_proxy uzantısı](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html) ve ilgili Modüller sunucunun ters proxy 'sini oluşturur.

## <a name="prerequisites"></a>Önkoşullar

* Sudo ayrıcalığına sahip standart bir kullanıcı hesabıyla CentOS 7 çalıştıran sunucu.
* .NET Core çalışma zamanını sunucuya yükler.
   1. [.Net çekirdeğini indir sayfasını](https://dotnet.microsoft.com/download/dotnet-core)ziyaret edin.
   1. En son Önizleme olmayan .NET Core sürümünü seçin.
   1. **Uygulama çalıştırma-çalışma zamanı**altındaki tabloda en son önizleme dışı çalışma zamanını indirin.
   1. Linux **Paket Yöneticisi yönergeleri** bağlantısını seçin ve CentOS talimatlarını izleyin.
* Mevcut bir ASP.NET Core uygulaması.

Paylaşılan Framework 'ü yükselttikten sonra gelecekte herhangi bir noktada, sunucu tarafından barındırılan ASP.NET Core uygulamaları yeniden başlatın.

## <a name="publish-and-copy-over-the-app"></a>Uygulama üzerinde Yayımla ve Kopyala

Uygulamayı [çerçeveye bağımlı bir dağıtım](/dotnet/core/deploying/#framework-dependent-deployments-fdd)için yapılandırın.

Uygulama yerel olarak çalıştırılır ve güvenli bağlantı (HTTPS) yapmak üzere yapılandırılmamışsa aşağıdaki yaklaşımlardan birini benimseyin:

* Uygulamayı güvenli yerel bağlantıları işleyecek şekilde yapılandırın. Daha fazla bilgi için [https yapılandırma](#https-configuration) bölümüne bakın.
* `https://localhost:5001` *Properties/launchsettings. JSON* dosyasındaki `applicationUrl` özelliğinden (varsa) kaldırın.

Bir uygulamayı sunucuda çalışabilecek bir dizine (örneğin, *bin/Release/&lt;target_framework_moniker&gt;/Publish*) paketlemek için geliştirme ortamından [DotNet Publish](/dotnet/core/tools/dotnet-publish) çalıştırın:

```dotnetcli
dotnet publish --configuration Release
```

Uygulama, sunucuda .NET Core çalışma zamanının bakımını yapmayı tercih ediyorsanız, [kendi kendine içerilen bir dağıtım](/dotnet/core/deploying/#self-contained-deployments-scd) olarak da yayımlanabilir.

ASP.NET Core uygulamasını, kuruluşun iş akışını (örneğin, SCP, SFTP) tümleştiren bir aracı kullanarak sunucuya kopyalayın. *Var* dizini altında Web uygulamalarının (örneğin, *var/www/HelloApp*) yerini bulmak yaygındır.

> [!NOTE]
> Bir üretim dağıtım senaryosunda, sürekli tümleştirme iş akışı, uygulamayı yayımlama ve varlıkları sunucuya kopyalama işini yapar.

## <a name="configure-a-proxy-server"></a>Proxy sunucusu yapılandırma

Ters proxy, dinamik Web uygulamaları sunmak için ortak bir kurulumtir. Ters proxy, HTTP isteğini sonlandırır ve ASP.NET uygulamasına iletir.

Proxy sunucusu, istemci isteklerini istekleri yerine başka bir sunucuya ileten bir sunucu olur. Ters proxy, genellikle rastgele istemciler adına sabit bir hedefe iletilir. Bu kılavuzda Apache, Kestrel 'in ASP.NET Core uygulamasına hizmet veren aynı sunucuda çalışan ters proxy olarak yapılandırılmıştır.

İstekler ters proxy tarafından iletileceği için, [Microsoft. AspNetCore. HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/) paketindeki [Iletilen üstbilgiler ara yazılımını](xref:host-and-deploy/proxy-load-balancer) kullanın. Ara yazılım, `X-Forwarded-Proto` üstbilgiyi `Request.Scheme`kullanarak, yeniden yönlendirme URI 'leri ve diğer güvenlik ilkelerini doğru çalışacak şekilde güncelleştirir.

Bir şemaya bağlı kimlik doğrulama, bağlantı oluşturma, yeniden yönlendirme ve coğrafi konum gibi herhangi bir bileşen, Iletilen üstbilgiler ara yazılımı çağrıldıktan sonra yerleştirilmelidir. Genel bir kural olarak, Iletilen üstbilgiler ara yazılımı, tanılama ve hata işleme ara yazılımı dışında diğer ara yazılım ile önce çalışmalıdır. Bu sıralama, iletilen üst bilgi bilgilerine bağlı olan ara yazılımın işleme için üst bilgi değerlerini kullanmasını sağlar.

Diğer ara <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersExtensions.UseForwardedHeaders*> yazılım çağrılmadan `Startup.Configure` önce yönteminin en üstünde yöntemi çağırın. Ara yazılımı, `X-Forwarded-For` ve `X-Forwarded-Proto` üst bilgilerini iletecek şekilde yapılandırın:

```csharp
// using Microsoft.AspNetCore.HttpOverrides;

app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseAuthentication();
```

Hayır <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions> , ara yazılım için belirtilmemişse, iletmek için varsayılan üstbilgiler şunlardır `None`.

Standart localhost adresi (127.0.0.1) dahil olmak üzere geri döngü adreslerinde çalışan proxy 'ler (127.0.0.0/8, [:: 1]), varsayılan olarak güvenilirdir. Kuruluş içindeki diğer güvenilir proxy 'ler veya ağlar, Internet ve Web sunucusu arasında istekleri ele alıyorsa, bunları <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownProxies*> veya <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.KnownNetworks*> ile <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions>listesine ekleyin. Aşağıdaki örnek, içindeki `KnownProxies` `Startup.ConfigureServices`iletilen ÜSTBILGILER ara sunucusuna alana 10.0.0.100 IP adresinde bir güvenilen ara sunucu ekler:

```csharp
// using System.Net;

services.Configure<ForwardedHeadersOptions>(options =>
{
    options.KnownProxies.Add(IPAddress.Parse("10.0.0.100"));
});
```

Daha fazla bilgi için bkz. <xref:host-and-deploy/proxy-load-balancer>.

### <a name="install-apache"></a>Apache 'yi yükler

CentOS paketlerini en son kararlı sürümlerine güncelleştirin:

```bash
sudo yum update -y
```

Tek `yum` bir komutla, CentOS üzerinde Apache Web sunucusunu yükler:

```bash
sudo yum -y install httpd mod_ssl
```

Komutu çalıştırdıktan sonra örnek çıkış:

```bash
Downloading packages:
httpd-2.4.6-40.el7.centos.4.x86_64.rpm               | 2.7 MB  00:00:01
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Installing : httpd-2.4.6-40.el7.centos.4.x86_64      1/1 
Verifying  : httpd-2.4.6-40.el7.centos.4.x86_64      1/1 

Installed:
httpd.x86_64 0:2.4.6-40.el7.centos.4

Complete!
```

> [!NOTE]
> Bu örnekte, CentOS 7 sürümü 64 bit olduğundan çıkış httpd. 86_64 ' i yansıtır. Apache 'nin yüklü olduğu yeri doğrulamak için komut `whereis httpd` isteminden komutunu çalıştırın.

### <a name="configure-apache"></a>Apache yapılandırma

Apache için yapılandırma dosyaları, `/etc/httpd/conf.d/` dizin içinde bulunur. *. Conf* uzantısına sahip herhangi bir dosya, içindeki `/etc/httpd/conf.modules.d/`modül yapılandırma dosyalarının yanı sıra, modülleri yüklemek için gereken yapılandırma dosyalarını içeren alfabetik sırada işlenir.

Uygulama için *HelloApp. conf*adlı bir yapılandırma dosyası oluşturun:

```
<VirtualHost *:*>
    RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
</VirtualHost>

<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
    ServerName www.example.com
    ServerAlias *.example.com
    ErrorLog ${APACHE_LOG_DIR}helloapp-error.log
    CustomLog ${APACHE_LOG_DIR}helloapp-access.log common
</VirtualHost>
```

`VirtualHost` Blok, sunucuda bir veya daha fazla dosyada birden çok kez görünebilir. Yukarıdaki yapılandırma dosyasında Apache, 80 numaralı bağlantı noktasında genel trafiği kabul eder. Etki alanına `www.example.com` sunulmakta ve `*.example.com` diğer ad aynı Web sitesine çözümlenmektedir. Daha fazla bilgi için bkz. [ad tabanlı sanal konak desteği](https://httpd.apache.org/docs/current/vhosts/name-based.html) . İstekler, kökte, sunucunun bağlantı noktası 5000 ' den 127.0.0.1 ' de sunucu üzerinden alınır. İki yönlü iletişim için gereklidir `ProxyPass` ve `ProxyPassReverse` gereklidir. Kestrel 'in IP/bağlantı noktasını değiştirmek için bkz. [Kestrel: Endpoint Configuration](xref:fundamentals/servers/kestrel#endpoint-configuration).

> [!WARNING]
> **VirtualHost** bloğunda uygun bir [ServerName yönergesi](https://httpd.apache.org/docs/current/mod/core.html#servername) belirtmemesi, uygulamanızı güvenlik açıklarına karşı kullanıma sunar. Alt etki alanı joker karakteri bağlama ( `*.example.com`Örneğin,), tüm üst etki alanını (Bu güvenlik açığı olan aksine `*.com`) kontrol ediyorsanız bu güvenlik riskini ortadan yapmaz. Daha fazla bilgi için bkz. [rfc7230 Section-5,4](https://tools.ietf.org/html/rfc7230#section-5.4) .

Günlüğe kaydetme, `ErrorLog` ve `CustomLog` yönergeleri `VirtualHost` başına yapılandırılabilir. `ErrorLog`, sunucunun hataları günlüğe kaydettiği konumdur ve `CustomLog` günlük dosyasının dosya adını ve biçimini ayarlar. Bu durumda istek bilgileri günlüğe kaydedilir. Her istek için bir satır vardır.

Dosyayı kaydedin ve yapılandırmayı test edin. Her şey geçerse yanıtın olması `Syntax [OK]`gerekir.

```bash
sudo service httpd configtest
```

Apache 'i yeniden Başlat:

```bash
sudo systemctl restart httpd
sudo systemctl enable httpd
```

## <a name="monitor-the-app"></a>Uygulamayı izleme

Apache, ' de Kestrel üzerinde çalışan ASP.NET Core uygulamasına `http://localhost:80` yapılan istekleri iletmek için kurulum kuruyor `http://127.0.0.1:5000`. Ancak, Kestrel işlemini yönetmek için Apache ayarlanmamış. Temel Web uygulamasını başlatmak ve izlemek için *systemd* ve hizmet dosyası oluşturma ' yı kullanın. *systemd* , işlem başlatmak, durdurmak ve yönetmek için birçok güçlü özellik sağlayan bir init sistemidir.

### <a name="create-the-service-file"></a>Hizmet dosyasını oluşturma

Hizmet tanımı dosyasını oluşturun:

```bash
sudo nano /etc/systemd/system/kestrel-helloapp.service
```

Uygulama için örnek bir hizmet dosyası:

```
[Unit]
Description=Example .NET Web API App running on CentOS 7

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/local/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=apache
Environment=ASPNETCORE_ENVIRONMENT=Production 

[Install]
WantedBy=multi-user.target
```

Yukarıdaki örnekte, hizmeti yöneten Kullanıcı `User` seçeneğiyle belirtilir. Kullanıcı (`apache`) var olmalıdır ve uygulamanın dosyalarının doğru sahipliğini içermelidir.

Uygulamanın `TimeoutStopSec` ilk kesme sinyali aldıktan sonra kapanması için bekleyeceği süreyi yapılandırmak için kullanın. Uygulama bu dönemde kapanmazsa, uygulamayı sonlandırmak için SIGKıLL çıkarılır. Değeri unitless saniyeler (örneğin, `150`), bir zaman aralığı değeri (örneğin, `2min 30s`) veya `infinity` zaman aşımını devre dışı bırakmak için girin. `TimeoutStopSec`Varsayılan olarak, yönetici yapılandırma `DefaultTimeoutStopSec` dosyasındaki değerini alır (*systemd-System. conf*, *System. conf. d*, *systemd-User. conf*, *User. conf. d*). Çoğu dağıtım için varsayılan zaman aşımı 90 saniyedir.

```
# The default value is 90 seconds for most distributions.
TimeoutStopSec=90
```

Yapılandırma sağlayıcılarının ortam değişkenlerini okuyabilmesi için bazı değerler (örneğin, SQL bağlantı dizeleri) kaçışmalıdır. Yapılandırma dosyasında kullanılmak üzere uygun bir kaçış değeri oluşturmak için aşağıdaki komutu kullanın:

```console
systemd-escape "<value-to-escape>"
```

İki nokta`:`() ayırıcılar ortam değişkeni adlarında desteklenmez. İki nokta üst üste yerine`__`çift alt çizgi () kullanın. Ortam değişkenleri [yapılandırma sağlayıcısı](xref:fundamentals/configuration/index#environment-variables-configuration-provider) , ortam değişkenleri yapılandırmaya okurken çift alt çizgileri iki nokta üst üste dönüştürür. Aşağıdaki örnekte, bağlantı dizesi anahtarı `ConnectionStrings:DefaultConnection` hizmet tanımı dosyasına şu şekilde `ConnectionStrings__DefaultConnection`ayarlanır:

```
Environment=ConnectionStrings__DefaultConnection={Connection String}
```

Dosyayı kaydedin ve hizmeti etkinleştirin:

```bash
sudo systemctl enable kestrel-helloapp.service
```

Hizmeti başlatın ve çalıştığını doğrulayın:

```bash
sudo systemctl start kestrel-helloapp.service
sudo systemctl status kestrel-helloapp.service

● kestrel-helloapp.service - Example .NET Web API App running on CentOS 7
    Loaded: loaded (/etc/systemd/system/kestrel-helloapp.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-helloapp.service
            └─9021 /usr/local/bin/dotnet /var/www/helloapp/helloapp.dll
```

Ters proxy yapılandırılmış ve *systemd*üzerinden yönetilen Kestrel, Web uygulaması tam olarak yapılandırılır ve adresinden `http://localhost`yerel makinedeki bir tarayıcıdan erişilebilir. Yanıt üst bilgilerini inceleyerek **sunucu** üst bilgisi, ASP.NET Core uygulamasının Kestrel tarafından sunulduğunu belirtir:

```
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="view-logs"></a>Günlükleri görüntüleme

Kestrel kullanan Web uygulaması *systemd*kullanılarak yönetildiğinden, olaylar ve süreçler merkezi bir günlüğe kaydedilir. Ancak, bu günlük *systemd*tarafından yönetilen tüm hizmet ve işlemlere ait girişleri içerir. Belirli öğeleri görüntülemek `kestrel-helloapp.service`için aşağıdaki komutu kullanın:

```bash
sudo journalctl -fu kestrel-helloapp.service
```

Zaman filtreleme için komutuyla saat seçeneklerini belirtin. Örneğin, geçerli güne `--since today` filtre uygulamak veya `--until 1 hour ago` önceki saatin girişlerini görmek için kullanın. Daha fazla bilgi için bkz. [journalctl için man sayfası](https://www.unix.com/man-page/centos/1/journalctl/).

```bash
sudo journalctl -fu kestrel-helloapp.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="data-protection"></a>Veri koruma

[ASP.NET Core veri koruma yığını](xref:security/data-protection/introduction) , kimlik doğrulama ara yazılımı (örneğin, tanımlama bilgisi ara yazılımı) ve siteler arası istek sahteciliğini önleme (CSRF) korumaları dahil olmak üzere birkaç ASP.NET Core [middlewares](xref:fundamentals/middleware/index)tarafından kullanılır. Veri koruma API 'Leri Kullanıcı kodu tarafından çağrılmasa bile, veri korumasının kalıcı bir şifreleme [anahtarı deposu](xref:security/data-protection/implementation/key-management)oluşturacak şekilde yapılandırılması gerekir. Veri koruması yapılandırılmamışsa, anahtarlar bellekte tutulur ve uygulama yeniden başlatıldığında atılır.

Uygulama yeniden başlatıldığında anahtar halkası bellekte depolanıyorsa:

* Tüm tanımlama bilgisi tabanlı kimlik doğrulama belirteçleri geçersiz kılınır.
* Kullanıcıların bir sonraki isteğinde yeniden oturum açması gerekir.
* Anahtar halkası ile korunan tüm veriler artık çözülemez. Bu, [CSRF belirteçlerini](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) ve [ASP.NET Core MVC TempData tanımlama bilgilerini](xref:fundamentals/app-state#tempdata)içerebilir.

Veri korumayı, anahtar halkasını sürdürmek ve şifrelemek üzere yapılandırmak için, bkz.:

* <xref:security/data-protection/implementation/key-storage-providers>
* <xref:security/data-protection/implementation/key-encryption-at-rest>

## <a name="secure-the-app"></a>Uygulamanın güvenliğini sağlama

### <a name="configure-firewall"></a>Güvenlik duvarını yapılandırma

*Firewallld* , ağ bölgeleri desteğiyle güvenlik duvarını yönetmek için dinamik bir Daemon. Bağlantı noktaları ve paket filtrelemesi, Iptables tarafından hala yönetilebilir. *Firewalld* , varsayılan olarak yüklenmelidir. `yum`paketi yüklemek veya yüklendiğini doğrulamak için kullanılabilir.

```bash
sudo yum install firewalld -y
```

Yalnızca `firewalld` uygulama için gerekli olan bağlantı noktalarını açmak için kullanın. Bu durumda, 80 ve 443 numaralı bağlantı noktası kullanılır. Aşağıdaki komutlar şunları açmak için 80 ve 443 bağlantı noktalarını kalıcı olarak ayarlar:

```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
```

Güvenlik Duvarı ayarlarını yeniden yükleyin. Varsayılan bölgedeki kullanılabilir hizmetleri ve bağlantı noktalarını kontrol edin. Seçenekler inceleyerek `firewall-cmd -h`kullanılabilir.

```bash
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

```bash
public (default, active)
interfaces: eth0
sources: 
services: dhcpv6-client
ports: 443/tcp 80/tcp
masquerade: no
forward-ports: 
icmp-blocks: 
rich rules: 
```

### <a name="https-configuration"></a>HTTPS yapılandırması

**Uygulamayı güvenli (HTTPS) yerel bağlantılar için yapılandırma**

[DotNet Run](/dotnet/core/tools/dotnet-run) komutu uygulamanın *Özellikler/launchsettings. JSON* dosyasını kullanır, bu da uygulamayı `applicationUrl` özelliği tarafından belirtilen URL 'lerde dinlemek üzere yapılandırır (örneğin, `https://localhost:5001;http://localhost:5000`).

Aşağıdaki yaklaşımlardan birini kullanarak, uygulamayı `dotnet run` komut veya geliştirme ortamı için geliştirme sırasında (F5 veya CTRL + F5 Visual Studio Code) bir sertifikayı kullanacak şekilde yapılandırın:

* [Varsayılan sertifikayı yapılandırmadan Değiştir](xref:fundamentals/servers/kestrel#configuration) (*önerilir*)
* [KestrelServerOptions. ConfigureHttpsDefaults](xref:fundamentals/servers/kestrel#configurehttpsdefaultsactionhttpsconnectionadapteroptions)

**Güvenli (HTTPS) istemci bağlantıları için ters proxy 'yi yapılandırma**

HTTPS için Apache 'yi yapılandırmak için *mod_ssl* modülü kullanılır. *Httpd* modülü yüklendiğinde *mod_ssl* modülü de yüklendi. Yüklenmemişse, yapılandırmaya eklemek için kullanın `yum` .

```bash
sudo yum install mod_ssl
```

HTTPS 'yi zorlamak için, URL `mod_rewrite` yeniden yazmayı etkinleştirmek üzere modülünü yükler:

```bash
sudo yum install mod_rewrite
```

Bağlantı noktası 443 üzerinde URL yeniden yazma ve güvenli iletişim sağlamak için *HelloApp. conf* dosyasını değiştirin:

```
<VirtualHost *:*>
    RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
</VirtualHost>

<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
</VirtualHost>

<VirtualHost *:443>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
    ErrorLog /var/log/httpd/helloapp-error.log
    CustomLog /var/log/httpd/helloapp-access.log common
    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!RC4+RSA:+HIGH:+MEDIUM:!LOW:!RC4
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

> [!NOTE]
> Bu örnek, yerel olarak üretilmiş bir sertifika kullanıyor. **Sslcertificatefile** , etki alanı adı için birincil sertifika dosyası olmalıdır. **Sslcertificatekeyfile** , CSR oluşturulduğunda oluşturulan anahtar dosyası olmalıdır. **Sslcertificatechainfile** , sertifika yetkilisi tarafından sağlanan ara sertifika dosyası (varsa) olmalıdır.

Dosyayı kaydedin ve yapılandırmayı test edin:

```bash
sudo service httpd configtest
```

Apache 'i yeniden Başlat:

```bash
sudo systemctl restart httpd
```

## <a name="additional-apache-suggestions"></a>Ek Apache önerileri

### <a name="restart-apps-with-shared-framework-updates"></a>Paylaşılan çerçeve güncelleştirmeleriyle uygulamaları yeniden başlatma

Sunucuda paylaşılan Framework 'ü yükselttikten sonra, sunucu tarafından barındırılan ASP.NET Core uygulamaları yeniden başlatın.

### <a name="additional-headers"></a>Ek üstbilgiler

Kötü amaçlı saldırılara karşı korumak için, değiştirilmesi veya eklenmesi gereken birkaç üstbilgi vardır. `mod_headers` Modülün yüklü olduğundan emin olun:

```bash
sudo yum install mod_headers
```

#### <a name="secure-apache-from-clickjacking-attacks"></a>Tıklama ve tıklama saldırılarına karşı güvenli Apache

*UI redki saldırısı*olarak da bilinen [tıklama](https://blog.qualys.com/securitylabs/2015/10/20/clickjacking-a-common-implementation-mistake-that-can-put-your-websites-in-danger), bir Web sitesi ziyaretçisinin bir bağlantı veya düğmeye Şu anda ziyaret ettiğinden farklı bir sayfada tıklanması zor olan kötü amaçlı bir saldırıya neden olur. Sitesini `X-FRAME-OPTIONS` güvenli hale getirmek için kullanın.

Tıklama saldırılarını azaltmak için:

1. *Httpd. conf* dosyasını düzenleyin:

   ```bash
   sudo nano /etc/httpd/conf/httpd.conf
   ```

   `Header append X-FRAME-OPTIONS "SAMEORIGIN"` satırını ekleyin.
1. Dosyayı kaydedin.
1. Apache 'i yeniden başlatın.

#### <a name="mime-type-sniffing"></a>MIME türü algılaması

Üst `X-Content-Type-Options` bilgi, Internet Explorer 'ın *MIME algılaması* (dosyanın içeriğinden bir dosya `Content-Type` belirleme) gerçekleştirmesini engeller. Sunucu `Content-Type` üstbilgiyi `text/html` `nosniff` seçenek kümesiyle ayarlarsa, Internet Explorer içeriği dosyanın içeriğinden bağımsız olarak `text/html` işler.

*Httpd. conf* dosyasını düzenleyin:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

`Header set X-Content-Type-Options "nosniff"` satırını ekleyin. Dosyayı kaydedin. Apache 'i yeniden başlatın.

### <a name="load-balancing"></a>Yük Dengeleme

Bu örnek, CentOS 7 ve Kestrel üzerinde Apache 'in aynı örnek makinede nasıl ayarlanacağını ve yapılandırılacağını gösterir. Tek bir hata noktası olmaması için; *mod_proxy_balancer* kullanmak ve **VirtualHost** 'u değiştirmek, Apache proxy sunucusunun arkasındaki Web uygulamalarının birden çok örneğini yönetmeye olanak tanır.

```bash
sudo yum install mod_proxy_balancer
```

Aşağıda gösterilen yapılandırma dosyasında, bağlantı noktası 5001 ' de çalışacak ek `helloapp` bir örneği ayarlanır. *Proxy* bölümü, Yük Dengeleme *istekleri*için iki üyeli bir dengeleyici yapılandırması ile ayarlanır.

```
<VirtualHost *:*>
    RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
</VirtualHost>

<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R,L]
</VirtualHost>

<VirtualHost *:443>
    ProxyPass / balancer://mycluster/ 

    ProxyPassReverse / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5001/

    <Proxy balancer://mycluster>
        BalancerMember http://127.0.0.1:5000
        BalancerMember http://127.0.0.1:5001 
        ProxySet lbmethod=byrequests
    </Proxy>

    <Location />
        SetHandler balancer
    </Location>
    ErrorLog /var/log/httpd/helloapp-error.log
    CustomLog /var/log/httpd/helloapp-access.log common
    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:!RC4+RSA:+HIGH:+MEDIUM:!LOW:!RC4
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

### <a name="rate-limits"></a>Oran limitleri

*Httpd* modülüne eklenen *mod_ratelimit*kullanarak, istemcilerin bant genişliği sınırlandırılabilir:

```bash
sudo nano /etc/httpd/conf.d/ratelimit.conf
```

Örnek dosya, bant genişliğini kök konumu altında 600 KB/sn olarak sınırlandırır:

```
<IfModule mod_ratelimit.c>
    <Location />
        SetOutputFilter RATE_LIMIT
        SetEnv rate-limit 600
    </Location>
</IfModule>
```

### <a name="long-request-header-fields"></a>Uzun istek üst bilgisi alanları

Proxy sunucusu varsayılan ayarları, istek üst bilgisi alanlarını genellikle 8.190 bayt ile sınırlar. Bir uygulama, varsayılan değerden daha uzun bir süre gerektirebilir (örneğin, [Azure Active Directory](https://azure.microsoft.com/services/active-directory/)kullanan uygulamalar). Daha uzun alanlar gerekliyse, proxy sunucusunun [LimitRequestFieldSize](https://httpd.apache.org/docs/2.4/mod/core.html#LimitRequestFieldSize) yönergesi ayarlamayı gerektirir. Uygulanacak değer senaryoya bağlıdır. Daha fazla bilgi için sunucunuzun belgelerine bakın.

> [!WARNING]
> Gerekli `LimitRequestFieldSize` olmadığı takdirde varsayılan değerini artırmaz. Değerin artırılması, kötü amaçlı kullanıcılar tarafından arabellek taşması (taşma) ve hizmet reddi (DoS) saldırıları riskini artırır.

## <a name="additional-resources"></a>Ek kaynaklar

* [Linux üzerinde .NET Core önkoşulları](/dotnet/core/linux-prerequisites)
* <xref:test/troubleshoot>
* <xref:host-and-deploy/proxy-load-balancer>
