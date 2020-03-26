---
title: ASP.NET Core 'de çok faktörlü kimlik doğrulaması
author: damienbod
description: ASP.NET Core uygulamasında Multi-Factor Authentication (MFA) ayarlamayı öğrenin.
monikerRange: '>= aspnetcore-3.1'
ms.author: rick-anderson
ms.custom: mvc
ms.date: 03/17/2020
no-loc:
- Identity
uid: security/authentication/mfa
ms.openlocfilehash: 6220688d53f0718ca5be5f63dd5d9539d37e2391
ms.sourcegitcommit: d64ef143c64ee4fdade8f9ea0b753b16752c5998
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 03/18/2020
ms.locfileid: "79520199"
---
# <a name="multi-factor-authentication-in-aspnet-core"></a><span data-ttu-id="88e6c-103">ASP.NET Core 'de çok faktörlü kimlik doğrulaması</span><span class="sxs-lookup"><span data-stu-id="88e6c-103">Multi-factor authentication in ASP.NET Core</span></span>

<span data-ttu-id="88e6c-104">, [Davmien Bowden](https://github.com/damienbod) tarafından</span><span class="sxs-lookup"><span data-stu-id="88e6c-104">By [Damien Bowden](https://github.com/damienbod)</span></span>

<span data-ttu-id="88e6c-105">Multi-Factor Authentication (MFA), bir kullanıcının ek tanımlama biçimleri için oturum açma olayında istediği bir işlemdir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-105">Multi-factor authentication (MFA) is a process in which a user is requested during a sign-in event for additional forms of identification.</span></span> <span data-ttu-id="88e6c-106">Bu istem, bir Cellphone 'dan bir kod girmek, FIDO2 anahtarı kullanmak veya parmak izi taraması sağlamak olabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-106">This prompt could be to enter a code from a cellphone, use a FIDO2 key, or to provide a fingerprint scan.</span></span> <span data-ttu-id="88e6c-107">İkinci bir kimlik doğrulama formu gerektirdiğinde güvenlik geliştirilmiştir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-107">When you require a second form of authentication, security is enhanced.</span></span> <span data-ttu-id="88e6c-108">Ek faktör bir saldırgan tarafından kolayca alınıp çoğaltılamaz.</span><span class="sxs-lookup"><span data-stu-id="88e6c-108">The additional factor isn't easily obtained or duplicated by an attacker.</span></span>

<span data-ttu-id="88e6c-109">Bu makalede aşağıdaki alanlarda yer verilmiştir:</span><span class="sxs-lookup"><span data-stu-id="88e6c-109">This article covers the following areas:</span></span>

* <span data-ttu-id="88e6c-110">MFA nedir ve MFA akışlarının kullanılması önerilir</span><span class="sxs-lookup"><span data-stu-id="88e6c-110">What is MFA and what MFA flows are recommended</span></span>
* <span data-ttu-id="88e6c-111">ASP.NET Core Identity kullanarak yönetim sayfaları için MFA 'yı yapılandırma</span><span class="sxs-lookup"><span data-stu-id="88e6c-111">Configure MFA for administration pages using ASP.NET Core Identity</span></span>
* <span data-ttu-id="88e6c-112">OpenID Connect sunucusuna MFA oturum açma gereksinimi gönderin</span><span class="sxs-lookup"><span data-stu-id="88e6c-112">Send MFA sign-in requirement to OpenID Connect server</span></span>
* <span data-ttu-id="88e6c-113">ASP.NET Core OpenID Connect istemcisini MFA gerektir olarak zorla</span><span class="sxs-lookup"><span data-stu-id="88e6c-113">Force ASP.NET Core OpenID Connect client to require MFA</span></span>

## <a name="mfa-2fa"></a><span data-ttu-id="88e6c-114">MFA, 2FA</span><span class="sxs-lookup"><span data-stu-id="88e6c-114">MFA, 2FA</span></span>

<span data-ttu-id="88e6c-115">MFA, bildiğiniz bir şey, sahip olduğunuz bir şey veya kullanıcının kimlik doğrulaması için Biyometri doğrulaması gibi bir kimlik için en az iki veya daha fazla kanıt türü gerektirir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-115">MFA requires at least two or more types of proof for an identity like something you know, something you possess, or biometric validation for the user to authenticate.</span></span>

<span data-ttu-id="88e6c-116">İki öğeli kimlik doğrulaması (2FA) MFA 'nın bir alt kümesi gibidir, ancak MFA 'nın kimliği kanıtlamak için iki veya daha fazla faktör gerektirmesine yönelik fark.</span><span class="sxs-lookup"><span data-stu-id="88e6c-116">Two-factor authentication (2FA) is like a subset of MFA, but the difference being that MFA can require two or more factors to prove the identity.</span></span>

### <a name="mfa-totp-time-based-one-time-password-algorithm"></a><span data-ttu-id="88e6c-117">MFA TOTP (zamana bağlı bir kerelik parola algoritması)</span><span class="sxs-lookup"><span data-stu-id="88e6c-117">MFA TOTP (Time-based One-time Password Algorithm)</span></span>

<span data-ttu-id="88e6c-118">TOTP kullanarak MFA, ASP.NET Core Identitykullanarak desteklenen bir uygulama.</span><span class="sxs-lookup"><span data-stu-id="88e6c-118">MFA using TOTP is a supported implementation using ASP.NET Core Identity.</span></span> <span data-ttu-id="88e6c-119">Bu, aşağıdakiler dahil olmak üzere tüm uyumlu Doğrulayıcı uygulamaları ile birlikte kullanılabilir:</span><span class="sxs-lookup"><span data-stu-id="88e6c-119">This can be used together with any compliant authenticator app, including:</span></span>

* <span data-ttu-id="88e6c-120">Microsoft Authenticator uygulaması</span><span class="sxs-lookup"><span data-stu-id="88e6c-120">Microsoft Authenticator App</span></span>
* <span data-ttu-id="88e6c-121">Google Authenticator uygulaması</span><span class="sxs-lookup"><span data-stu-id="88e6c-121">Google Authenticator App</span></span>

<span data-ttu-id="88e6c-122">Uygulama ayrıntıları için aşağıdaki bağlantıya bakın:</span><span class="sxs-lookup"><span data-stu-id="88e6c-122">See the following link for implementation details:</span></span>

[<span data-ttu-id="88e6c-123">ASP.NET Core 'daki TOTP Authenticator uygulamaları için QR kodu oluşturmayı etkinleştirme</span><span class="sxs-lookup"><span data-stu-id="88e6c-123">Enable QR Code generation for TOTP authenticator apps in ASP.NET Core</span></span>](xref:security/authentication/identity-enable-qrcodes)

### <a name="mfa-fido2-or-passwordless"></a><span data-ttu-id="88e6c-124">MFA FIDO2 veya passwordless</span><span class="sxs-lookup"><span data-stu-id="88e6c-124">MFA FIDO2 or passwordless</span></span>

<span data-ttu-id="88e6c-125">FIDO2 Şu anda:</span><span class="sxs-lookup"><span data-stu-id="88e6c-125">FIDO2 is currently:</span></span>

* <span data-ttu-id="88e6c-126">MFA elde etmenin en güvenli yolu.</span><span class="sxs-lookup"><span data-stu-id="88e6c-126">The most secure way of achieving MFA.</span></span>
* <span data-ttu-id="88e6c-127">Sızdırma saldırılarına karşı koruyan tek MFA akışı.</span><span class="sxs-lookup"><span data-stu-id="88e6c-127">The only MFA flow that protects against phishing attacks.</span></span>

<span data-ttu-id="88e6c-128">Mevcut olduğunda ASP.NET Core doğrudan FIDO2 desteklemez.</span><span class="sxs-lookup"><span data-stu-id="88e6c-128">At present, ASP.NET Core doesn't support FIDO2 directly.</span></span> <span data-ttu-id="88e6c-129">FIDO2, MFA veya parolasız akışlar için kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-129">FIDO2 can be used for MFA or passwordless flows.</span></span>

<span data-ttu-id="88e6c-130">Azure Active Directory, FIDO2 ve parolasız akışlar için destek sağlar.</span><span class="sxs-lookup"><span data-stu-id="88e6c-130">Azure Active Directory provides support for FIDO2 and passwordless flows.</span></span> <span data-ttu-id="88e6c-131">Daha fazla bilgi için bkz. [Azure Active Directory Için passwordless kimlik doğrulama seçenekleri](/azure/active-directory/authentication/concept-authentication-passwordless).</span><span class="sxs-lookup"><span data-stu-id="88e6c-131">For more information, see [Passwordless authentication options for Azure Active Directory](/azure/active-directory/authentication/concept-authentication-passwordless).</span></span>

### <a name="mfa-sms"></a><span data-ttu-id="88e6c-132">MFA SMS</span><span class="sxs-lookup"><span data-stu-id="88e6c-132">MFA SMS</span></span>

<span data-ttu-id="88e6c-133">SMS ile MFA, parola kimlik doğrulaması (tek etken) ile karşılaştırıldığında güvenlik çarpanını büyük ölçüde artırır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-133">MFA with SMS increases security massively compared with password authentication (single factor).</span></span> <span data-ttu-id="88e6c-134">Ancak, SMS 'nin ikinci bir faktör olarak kullanılması artık önerilmez.</span><span class="sxs-lookup"><span data-stu-id="88e6c-134">However, using SMS as a second factor is no longer recommended.</span></span> <span data-ttu-id="88e6c-135">Bu tür bir uygulama için çok sayıda bilinen saldırı vektörü var.</span><span class="sxs-lookup"><span data-stu-id="88e6c-135">Too many known attack vectors exist for this type of implementation.</span></span>

[<span data-ttu-id="88e6c-136">NıST yönergeleri</span><span class="sxs-lookup"><span data-stu-id="88e6c-136">NIST guidelines</span></span>](https://pages.nist.gov/800-63-3/sp800-63b.html)

## <a name="configure-mfa-for-administration-pages-using-aspnet-core-opno-locidentity"></a><span data-ttu-id="88e6c-137">ASP.NET Core Identity kullanarak yönetim sayfaları için MFA 'yı yapılandırma</span><span class="sxs-lookup"><span data-stu-id="88e6c-137">Configure MFA for administration pages using ASP.NET Core Identity</span></span>

<span data-ttu-id="88e6c-138">MFA, kullanıcıların bir ASP.NET Core Identity uygulaması içindeki hassas sayfalara erişmesini zorunlu olarak verebilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-138">MFA could be forced on users to access sensitive pages within an ASP.NET Core Identity app.</span></span> <span data-ttu-id="88e6c-139">Bu, farklı kimlikler için farklı erişim düzeylerinin bulunduğu uygulamalar için yararlı olabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-139">This could be useful for apps where different levels of access exist for the different identities.</span></span> <span data-ttu-id="88e6c-140">Örneğin, kullanıcılar bir parola oturum açma kullanarak profil verilerini görüntüleyebilir, ancak yönetim sayfalarına erişmek için bir yöneticinin MFA 'yı kullanması gerekir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-140">For example, users might be able to view the profile data using a password login, but an administrator would be required to use MFA to access the administrative pages.</span></span>

### <a name="extend-the-login-with-an-mfa-claim"></a><span data-ttu-id="88e6c-141">Bir MFA talebi ile oturum açmayı genişletme</span><span class="sxs-lookup"><span data-stu-id="88e6c-141">Extend the login with an MFA claim</span></span>

<span data-ttu-id="88e6c-142">Tanıtım kodu, Identity ve Razor Pages ile ASP.NET Core kullanarak kurulum işlemi yapılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-142">The demo code is setup using ASP.NET Core with Identity and Razor Pages.</span></span> <span data-ttu-id="88e6c-143">`AddIdentity` yöntemi bir `AddDefaultIdentity` yerine kullanılır. bu nedenle, başarılı bir oturum açma işleminden sonra, kimliğe talep eklemek için `IUserClaimsPrincipalFactory` bir uygulama kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-143">The `AddIdentity` method is used instead of `AddDefaultIdentity` one, so an `IUserClaimsPrincipalFactory` implementation can be used to add claims to the identity after a successful login.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));
    
    services.AddIdentity<IdentityUser, IdentityRole>(
            options => options.SignIn.RequireConfirmedAccount = false)
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddSingleton<IEmailSender, EmailSender>();
    services.AddScoped<IUserClaimsPrincipalFactory<IdentityUser>, 
        AdditionalUserClaimsPrincipalFactory>();

    services.AddAuthorization(options =>
        options.AddPolicy("TwoFactorEnabled",
            x => x.RequireClaim("amr", "mfa")));

    services.AddRazorPages();
}
```

<span data-ttu-id="88e6c-144">`AdditionalUserClaimsPrincipalFactory` sınıfı, `amr` talebini yalnızca başarılı bir oturum açma işleminden sonra Kullanıcı taleplerine ekler.</span><span class="sxs-lookup"><span data-stu-id="88e6c-144">The `AdditionalUserClaimsPrincipalFactory` class adds the `amr` claim to the user claims only after a successful login.</span></span> <span data-ttu-id="88e6c-145">Talebin değeri veritabanından okundu.</span><span class="sxs-lookup"><span data-stu-id="88e6c-145">The claim's value is read from the database.</span></span> <span data-ttu-id="88e6c-146">Talep buraya eklenir, çünkü kimlik MFA ile oturum açtıysa Kullanıcı yalnızca daha yüksek korumalı görünüme erişir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-146">The claim is added here because the user should only access the higher protected view if the identity has logged in with MFA.</span></span> <span data-ttu-id="88e6c-147">Veritabanı görünümü talebi kullanmak yerine doğrudan veritabanından okunduktan sonra MFA 'yı etkinleştirdikten sonra doğrudan MFA olmadan görünüme erişebilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="88e6c-147">If the database view is read from the database directly instead of using the claim, it's possible to access the view without MFA directly after activating the MFA.</span></span>

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.Options;
using System.Collections.Generic;
using System.Security.Claims;
using System.Threading.Tasks;

namespace IdentityStandaloneMfa
{
    public class AdditionalUserClaimsPrincipalFactory : 
        UserClaimsPrincipalFactory<IdentityUser, IdentityRole>
    {
        public AdditionalUserClaimsPrincipalFactory( 
            UserManager<IdentityUser> userManager,
            RoleManager<IdentityRole> roleManager, 
            IOptions<IdentityOptions> optionsAccessor) 
            : base(userManager, roleManager, optionsAccessor)
        {
        }

        public async override Task<ClaimsPrincipal> CreateAsync(IdentityUser user)
        {
            var principal = await base.CreateAsync(user);
            var identity = (ClaimsIdentity)principal.Identity;

            var claims = new List<Claim>();

            if (user.TwoFactorEnabled)
            {
                claims.Add(new Claim("amr", "mfa"));
            }
            else
            {
                claims.Add(new Claim("amr", "pwd"));
            }

            identity.AddClaims(claims);
            return principal;
        }
    }
}
```

<span data-ttu-id="88e6c-148">`Startup` sınıfında Identity hizmeti kurulumu değiştiğinden, Identity düzenlerinin güncellenmesi gerekir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-148">Because the Identity service setup changed in the `Startup` class, the layouts of the Identity need to be updated.</span></span> <span data-ttu-id="88e6c-149">Identity sayfalarını uygulamaya dönüştürün.</span><span class="sxs-lookup"><span data-stu-id="88e6c-149">Scaffold the Identity pages into the app.</span></span> <span data-ttu-id="88e6c-150">*Identity/Account/Manage/_Layout. cshtml* dosyasında düzeni tanımlayın.</span><span class="sxs-lookup"><span data-stu-id="88e6c-150">Define the layout in the *Identity/Account/Manage/_Layout.cshtml* file.</span></span>

```cshtml
@{
    Layout = "/Pages/Shared/_Layout.cshtml";
}
```

<span data-ttu-id="88e6c-151">Ayrıca, Identity sayfalarındaki tüm yönetim sayfalarını için Düzen ata:</span><span class="sxs-lookup"><span data-stu-id="88e6c-151">Also assign the layout for all the manage pages from the Identity pages:</span></span>

```cshtml
@{
    Layout = "_Layout.cshtml";
}
```

### <a name="validate-the-mfa-requirement-in-the-administration-page"></a><span data-ttu-id="88e6c-152">Yönetim sayfasında MFA gereksinimini doğrulama</span><span class="sxs-lookup"><span data-stu-id="88e6c-152">Validate the MFA requirement in the administration page</span></span>

<span data-ttu-id="88e6c-153">Yönetim Razor sayfası, kullanıcının MFA kullanarak oturum açtığını doğrular.</span><span class="sxs-lookup"><span data-stu-id="88e6c-153">The administration Razor Page validates that the user has logged in using MFA.</span></span> <span data-ttu-id="88e6c-154">`OnGet` yönteminde, kimlik Kullanıcı taleplerine erişmek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-154">In the `OnGet` method, the identity is used to access the user claims.</span></span> <span data-ttu-id="88e6c-155">`amr` talebi `mfa`değer için denetlenir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-155">The `amr` claim is checked for the value `mfa`.</span></span> <span data-ttu-id="88e6c-156">Kimliğin bu talebi eksikse veya `false`, sayfa MFA 'yı etkinleştir sayfasına yeniden yönlendirir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-156">If the identity is missing this claim or is `false`, the page redirects to the Enable MFA page.</span></span> <span data-ttu-id="88e6c-157">Bu, Kullanıcı zaten MFA olmadan oturum açtığından, bu mümkündür.</span><span class="sxs-lookup"><span data-stu-id="88e6c-157">This is possible because the user has logged in already, but without MFA.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace IdentityStandaloneMfa
{
    public class AdminModel : PageModel
    {
        public IActionResult OnGet()
        {
            var claimTwoFactorEnabled = 
                User.Claims.FirstOrDefault(t => t.Type == "amr");

            if (claimTwoFactorEnabled != null && 
                "mfa".Equals(claimTwoFactorEnabled.Value))
            {
                // You logged in with MFA, do the administrative stuff
            }
            else
            {
                return Redirect(
                    "/Identity/Account/Manage/TwoFactorAuthentication");
            }

            return Page();
        }
    }
}
```

### <a name="ui-logic-to-toggle-user-login-information"></a><span data-ttu-id="88e6c-158">Kullanıcı oturum açma bilgilerini değiştirmek için UI mantığı</span><span class="sxs-lookup"><span data-stu-id="88e6c-158">UI logic to toggle user login information</span></span>

<span data-ttu-id="88e6c-159">Başlatma sırasında bir yetkilendirme ilkesi eklenmiştir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-159">An authorization policy was added at startup.</span></span> <span data-ttu-id="88e6c-160">İlke, `mfa`değer `amr` talep gerektiriyor.</span><span class="sxs-lookup"><span data-stu-id="88e6c-160">The policy requires the `amr` claim with the value `mfa`.</span></span>

```csharp
services.AddAuthorization(options =>
    options.AddPolicy("TwoFactorEnabled",
        x => x.RequireClaim("amr", "mfa")));
```

<span data-ttu-id="88e6c-161">Bu ilke daha sonra, **yönetici** menüsünü uyarı ile göstermek veya gizlemek için `_Layout` görünümünde kullanılabilir:</span><span class="sxs-lookup"><span data-stu-id="88e6c-161">This policy can then be used in the `_Layout` view to show or hide the **Admin** menu with the warning:</span></span>

```cshtml
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Identity
@inject SignInManager<IdentityUser> SignInManager
@inject UserManager<IdentityUser> UserManager
@inject IAuthorizationService AuthorizationService
```

<span data-ttu-id="88e6c-162">Kimlik MFA kullanılarak oturum açtıysa, **yönetici** menüsü araç ipucu uyarısı olmadan görüntülenir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-162">If the identity has logged in using MFA, the **Admin** menu is displayed without the tooltip warning.</span></span> <span data-ttu-id="88e6c-163">Kullanıcı MFA olmadan oturum açtığında, **yönetici (etkin değil)** menüsü, kullanıcıya bildiren araç ipucuyla birlikte görüntülenir (uyarı hakkında).</span><span class="sxs-lookup"><span data-stu-id="88e6c-163">When the user has logged in without MFA, the **Admin (Not Enabled)** menu is displayed along with the tooltip that informs the user (explaining the warning).</span></span>

```cshtml
@if (SignInManager.IsSignedIn(User))
{
    @if ((AuthorizationService.AuthorizeAsync(User, "TwoFactorEnabled")).Result.Succeeded)
    {
        <li class="nav-item">
            <a class="nav-link text-dark" asp-area="" asp-page="/Admin">Admin</a>
        </li>
    }
    else
    {
        <li class="nav-item">
            <a class="nav-link text-dark" asp-area="" asp-page="/Admin" 
               id="tooltip-demo"  
               data-toggle="tooltip" 
               data-placement="bottom" 
               title="MFA is NOT enabled. This is required for the Admin Page. If you have activated MFA, then logout, login again.">
                Admin (Not Enabled)
            </a>
        </li>
    }
}
```

<span data-ttu-id="88e6c-164">Kullanıcı MFA olmadan oturum açarsa, uyarı görüntülenir:</span><span class="sxs-lookup"><span data-stu-id="88e6c-164">If the user logs in without MFA, the warning is displayed:</span></span>

![Yönetici MFA kimlik doğrulaması](mfa/_static/identitystandalonemfa_01.png)

<span data-ttu-id="88e6c-166">**Yönetici** bağlantısına tıklarken Kullanıcı MFA etkinleştirme görünümüne yönlendirilir:</span><span class="sxs-lookup"><span data-stu-id="88e6c-166">The user is redirected to the MFA enable view when clicking the **Admin** link:</span></span>

![Yönetici MFA kimlik doğrulamasını etkinleştirir](mfa/_static/identitystandalonemfa_02.png)

## <a name="send-mfa-sign-in-requirement-to-openid-connect-server"></a><span data-ttu-id="88e6c-168">OpenID Connect sunucusuna MFA oturum açma gereksinimi gönderin</span><span class="sxs-lookup"><span data-stu-id="88e6c-168">Send MFA sign-in requirement to OpenID Connect server</span></span> 

<span data-ttu-id="88e6c-169">`acr_values` parametresi, `mfa` gereken değeri istemciden bir kimlik doğrulaması isteğindeki sunucuya geçirmek için kullanılabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-169">The `acr_values` parameter can be used to pass the `mfa` required value from the client to the server in an authentication request.</span></span>

> [!NOTE]
> <span data-ttu-id="88e6c-170">Bunun çalışması için `acr_values` parametresinin açık KIMLIK Connect sunucusunda işlenmesi gerekir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-170">The `acr_values` parameter needs to be handled on the Open ID Connect server for this to work.</span></span>

### <a name="openid-connect-aspnet-core-client"></a><span data-ttu-id="88e6c-171">OpenID Connect ASP.NET Core istemcisi</span><span class="sxs-lookup"><span data-stu-id="88e6c-171">OpenID Connect ASP.NET Core client</span></span>

<span data-ttu-id="88e6c-172">ASP.NET Core Razor Pages açık KIMLIĞI Connect istemci uygulaması, açık KIMLIK Connect sunucusunda oturum açmak için `AddOpenIdConnect` yöntemini kullanır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-172">The ASP.NET Core Razor Pages Open ID Connect client app uses the `AddOpenIdConnect` method to login to the Open ID Connect server.</span></span> <span data-ttu-id="88e6c-173">`acr_values` parametresi `mfa` değeri ile ayarlanır ve kimlik doğrulama isteğiyle birlikte gönderilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-173">The `acr_values` parameter is set with the `mfa` value and sent with the authentication request.</span></span> <span data-ttu-id="88e6c-174">`OpenIdConnectEvents` bunu eklemek için kullanılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-174">The `OpenIdConnectEvents` is used to add this.</span></span>

<span data-ttu-id="88e6c-175">Önerilen `acr_values` parametre değerleri için bkz. [kimlik doğrulama yöntemi başvuru değerleri](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08).</span><span class="sxs-lookup"><span data-stu-id="88e6c-175">For recommended `acr_values` parameter values, see [Authentication Method Reference Values](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08).</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme =
            OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.SignInScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.Authority = "<OpenID Connect server URL>";
        options.RequireHttpsMetadata = true;
        options.ClientId = "<OpenID Connect client ID>";
        options.ClientSecret = "<>";
        // Code with PKCE can also be used here
        options.ResponseType = "code id_token";
        options.Scope.Add("profile");
        options.Scope.Add("offline_access");
        options.SaveTokens = true;
        options.Events = new OpenIdConnectEvents
        {
            OnRedirectToIdentityProvider = context =>
            {
                context.ProtocolMessage.SetParameter("acr_values", "mfa");
                return Task.FromResult(0);
            }
        };
    });
```

### <a name="example-openid-connect-identityserver-4-server-with-aspnet-core-opno-locidentity"></a><span data-ttu-id="88e6c-176">Örnek OpenID Connect IdentityServer 4 sunucusu ASP.NET Core Identity</span><span class="sxs-lookup"><span data-stu-id="88e6c-176">Example OpenID Connect IdentityServer 4 server with ASP.NET Core Identity</span></span>

<span data-ttu-id="88e6c-177">MVC görünümleriyle ASP.NET Core Identity kullanılarak uygulanan OpenID Connect sunucusunda, *ErrorEnable2FA. cshtml* adlı yeni bir görünüm oluşturulur.</span><span class="sxs-lookup"><span data-stu-id="88e6c-177">On the OpenID Connect server, which is implemented using ASP.NET Core Identity with MVC views, a new view named *ErrorEnable2FA.cshtml* is created.</span></span> <span data-ttu-id="88e6c-178">Görünüm:</span><span class="sxs-lookup"><span data-stu-id="88e6c-178">The view:</span></span>

* <span data-ttu-id="88e6c-179">Identity MFA gerektiren bir uygulamadan geliyorsa, ancak kullanıcı bunu Identityetkinleştirmediğinde görüntüler.</span><span class="sxs-lookup"><span data-stu-id="88e6c-179">Displays if the Identity comes from an app that requires MFA but the user hasn't activated this in Identity.</span></span>
* <span data-ttu-id="88e6c-180">Kullanıcıya bildirir ve bunu etkinleştirmek için bir bağlantı ekler.</span><span class="sxs-lookup"><span data-stu-id="88e6c-180">Informs the user and adds a link to activate this.</span></span>

```cshtml
@{
    ViewData["Title"] = "ErrorEnable2FA";
}

<h1>The client application requires you to have MFA enabled. Enable this, try login again.</h1>

<br />

You can enable MFA to login here:

<br />

<a asp-controller="Manage" asp-action="TwoFactorAuthentication">Enable MFA</a>
```

<span data-ttu-id="88e6c-181">`Login` yönteminde, açık KIMLIK Connect istek parametrelerine erişmek için `IIdentityServerInteractionService` arabirimi uygulama `_interaction` kullanılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-181">In the `Login` method, the `IIdentityServerInteractionService` interface implementation `_interaction` is used to access the Open ID Connect request parameters.</span></span> <span data-ttu-id="88e6c-182">`acr_values` parametresine `AcrValues` özelliği kullanılarak erişilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-182">The `acr_values` parameter is accessed using the `AcrValues` property.</span></span> <span data-ttu-id="88e6c-183">İstemci bu `mfa` kümesi ile gönderildiğinden, bu daha sonra denetlenebilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-183">As the client sent this with `mfa` set, this can then be checked.</span></span>

<span data-ttu-id="88e6c-184">MFA gerekliyse ve ASP.NET Core Identity içindeki Kullanıcı MFA 'yı etkinleştirmişse, oturum açma işlemi devam eder.</span><span class="sxs-lookup"><span data-stu-id="88e6c-184">If MFA is required, and the user in ASP.NET Core Identity has MFA enabled, then the login continues.</span></span> <span data-ttu-id="88e6c-185">Kullanıcının MFA özelliği etkin olmadığında, Kullanıcı *ErrorEnable2FA. cshtml*özel görünümüne yönlendirilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-185">When the user has no MFA enabled, the user is redirected to the custom view *ErrorEnable2FA.cshtml*.</span></span> <span data-ttu-id="88e6c-186">Ardından Kullanıcı ASP.NET Core Identity oturum açar.</span><span class="sxs-lookup"><span data-stu-id="88e6c-186">Then ASP.NET Core Identity signs the user in.</span></span>

```csharp
//
// POST: /Account/Login
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginInputModel model)
{
    var returnUrl = model.ReturnUrl;
    var context = 
        await _interaction.GetAuthorizationContextAsync(returnUrl);
    var requires2Fa = 
        context?.AcrValues.Count(t => t.Contains("mfa")) >= 1;

    var user = await _userManager.FindByNameAsync(model.Email);
    if (user != null && !user.TwoFactorEnabled && requires2Fa)
    {
        return RedirectToAction(nameof(ErrorEnable2FA));
    }

    // code omitted for brevity
```

<span data-ttu-id="88e6c-187">`ExternalLoginCallback` yöntemi yerel Identity oturum açma gibi çalışmaktadır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-187">The `ExternalLoginCallback` method works like the local Identity login.</span></span> <span data-ttu-id="88e6c-188">`AcrValues` özelliği `mfa` değeri için denetlenir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-188">The `AcrValues` property is checked for the `mfa` value.</span></span> <span data-ttu-id="88e6c-189">`mfa` değeri varsa, oturum açma tamamlanmadan önce MFA zorlanır (örneğin, `ErrorEnable2FA` görünümüne yeniden yönlendirilir).</span><span class="sxs-lookup"><span data-stu-id="88e6c-189">If the `mfa` value is present, MFA is forced before the login completes (for example, redirected to the `ErrorEnable2FA` view).</span></span>

```csharp
//
// GET: /Account/ExternalLoginCallback
[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> ExternalLoginCallback(
    string returnUrl = null,
    string remoteError = null)
{
    var context =
        await _interaction.GetAuthorizationContextAsync(returnUrl);
    var requires2Fa =
        context?.AcrValues.Count(t => t.Contains("mfa")) >= 1;

    if (remoteError != null)
    {
        ModelState.AddModelError(
            string.Empty,
            _sharedLocalizer["EXTERNAL_PROVIDER_ERROR", 
            remoteError]);
        return View(nameof(Login));
    }
    var info = await _signInManager.GetExternalLoginInfoAsync();

    if (info == null)
    {
        return RedirectToAction(nameof(Login));
    }

    var email = info.Principal.FindFirstValue(ClaimTypes.Email);

    if (!string.IsNullOrEmpty(email))
    {
        var user = await _userManager.FindByNameAsync(email);
        if (user != null && !user.TwoFactorEnabled && requires2Fa)
        {
            return RedirectToAction(nameof(ErrorEnable2FA));
        }
    }

    // Sign in the user with this external login provider if the user already has a login.
    var result = await _signInManager
        .ExternalLoginSignInAsync(
            info.LoginProvider, 
            info.ProviderKey, 
            isPersistent: 
            false);

    // code omitted for brevity
```

<span data-ttu-id="88e6c-190">Kullanıcı zaten oturum açmışsa istemci uygulaması:</span><span class="sxs-lookup"><span data-stu-id="88e6c-190">If the user is already logged in, the client app:</span></span>

* <span data-ttu-id="88e6c-191">`amr` talebini hala doğrular.</span><span class="sxs-lookup"><span data-stu-id="88e6c-191">Still validates the `amr` claim.</span></span>
* <span data-ttu-id="88e6c-192">MFA 'yı ASP.NET Core Identity görünümünün bir bağlantısıyla ayarlayabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="88e6c-192">Can set up the MFA with a link to the ASP.NET Core Identity view.</span></span>

![acr_values-1](mfa/_static/acr_values-1.png)

## <a name="force-aspnet-core-openid-connect-client-to-require-mfa"></a><span data-ttu-id="88e6c-194">ASP.NET Core OpenID Connect istemcisini MFA gerektir olarak zorla</span><span class="sxs-lookup"><span data-stu-id="88e6c-194">Force ASP.NET Core OpenID Connect client to require MFA</span></span>

<span data-ttu-id="88e6c-195">Bu örnek, OpenID Connect kullanan bir ASP.NET Core Razor sayfası uygulamasının, kullanıcıların MFA kullanarak kimlik doğrulamasından sahip olduğunu gerektirdiğini gösterir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-195">This example shows how an ASP.NET Core Razor Page app, which uses OpenID Connect to sign in, can require that users have authenticated using MFA.</span></span>

<span data-ttu-id="88e6c-196">MFA gereksinimini doğrulamak için bir `IAuthorizationRequirement` gereksinimi oluşturulur.</span><span class="sxs-lookup"><span data-stu-id="88e6c-196">To validate the MFA requirement, an `IAuthorizationRequirement` requirement is created.</span></span> <span data-ttu-id="88e6c-197">Bu, MFA gerektiren bir ilke kullanılarak sayfalara eklenecektir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-197">This will be added to the pages using a policy that requires MFA.</span></span>

```csharp
using Microsoft.AspNetCore.Authorization;
 
namespace AspNetCoreRequireMfaOidc
{
    public class RequireMfa : IAuthorizationRequirement{}
}
```

<span data-ttu-id="88e6c-198">`amr` talebi kullanacak ve `mfa`değeri kontrol edecek bir `AuthorizationHandler` uygulanır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-198">An `AuthorizationHandler` is implemented that will use the `amr` claim and check for the value `mfa`.</span></span> <span data-ttu-id="88e6c-199">`amr`, başarılı bir kimlik doğrulamasının `id_token` döndürülür ve [kimlik doğrulama yöntemi başvuru değerleri](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08) belirtiminde tanımlanan birçok farklı değere sahip olabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-199">The `amr` is returned in the `id_token` of a successful authentication and can have many different values as defined in the [Authentication Method Reference Values](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08) specification.</span></span>

<span data-ttu-id="88e6c-200">Döndürülen değer, kimliğin kimlik doğrulamasının nasıl yapıldığını ve açık KIMLIK sunucu uygulamasında bir bağlantı olduğunu gösterir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-200">The returned value depends on how the identity authenticated and on the Open ID Connect server implementation.</span></span>

<span data-ttu-id="88e6c-201">`AuthorizationHandler`, `RequireMfa` gereksinimini kullanır ve `amr` talebini doğrular.</span><span class="sxs-lookup"><span data-stu-id="88e6c-201">The `AuthorizationHandler` uses the `RequireMfa` requirement and validates the `amr` claim.</span></span> <span data-ttu-id="88e6c-202">OpenID Connect sunucusu, ASP.NET Core Identityile birlikte ıdentityserver4 kullanılarak uygulanabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-202">The OpenID Connect server can be implemented using IdentityServer4 with ASP.NET Core Identity.</span></span> <span data-ttu-id="88e6c-203">Bir Kullanıcı TOTP kullanarak oturum açtığında, `amr` talep bir MFA değeri ile döndürülür.</span><span class="sxs-lookup"><span data-stu-id="88e6c-203">When a user logs in using TOTP, the `amr` claim is returned with an MFA value.</span></span> <span data-ttu-id="88e6c-204">Farklı bir OpenID Connect sunucu uygulamasının veya farklı bir MFA türünün kullanılması durumunda, `amr` talebi farklı bir değere sahip olur veya olabilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-204">If using a different OpenID Connect server implementation or a different MFA type, the `amr` claim will, or can, have a different value.</span></span> <span data-ttu-id="88e6c-205">Bu kod de kabul etmek için genişletilmelidir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-205">The code must be extended to accept this as well.</span></span>

```csharp
using Microsoft.AspNetCore.Authorization;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace AspNetCoreRequireMfaOidc
{
    public class RequireMfaHandler : AuthorizationHandler<RequireMfa>
    {
        protected override Task HandleRequirementAsync(
            AuthorizationHandlerContext context, 
            RequireMfa requirement)
        {
            if (context == null)
                throw new ArgumentNullException(nameof(context));
            if (requirement == null)
                throw new ArgumentNullException(nameof(requirement));

            var amrClaim =
                context.User.Claims.FirstOrDefault(t => t.Type == "amr");

            if (amrClaim != null && amrClaim.Value == Amr.Mfa)
            {
                context.Succeed(requirement);
            }

            return Task.CompletedTask;
        }
    }
}
```

<span data-ttu-id="88e6c-206">`Startup.ConfigureServices` yönteminde, `AddOpenIdConnect` yöntemi varsayılan zorluk düzeni olarak kullanılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-206">In the `Startup.ConfigureServices` method, the `AddOpenIdConnect` method is used as the default challenge scheme.</span></span> <span data-ttu-id="88e6c-207">`amr` talebini denetlemek için kullanılan yetkilendirme işleyicisi Denetim kapsayıcısının INVERSION öğesine eklenir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-207">The authorization handler, which is used to check the `amr` claim, is added to the Inversion of Control container.</span></span> <span data-ttu-id="88e6c-208">Daha sonra `RequireMfa` gereksinimini ekleyen bir ilke oluşturulur.</span><span class="sxs-lookup"><span data-stu-id="88e6c-208">A policy is then created which adds the `RequireMfa` requirement.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.ConfigureApplicationCookie(options =>
        options.Cookie.SecurePolicy =
            CookieSecurePolicy.Always);

    services.AddSingleton<IAuthorizationHandler, RequireMfaHandler>();

    services.AddAuthentication(options =>
    {
        options.DefaultScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme =
            OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.SignInScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.Authority = "https://localhost:44352";
        options.RequireHttpsMetadata = true;
        options.ClientId = "AspNetCoreRequireMfaOidc";
        options.ClientSecret = "AspNetCoreRequireMfaOidcSecret";
        options.ResponseType = "code id_token";
        options.Scope.Add("profile");
        options.Scope.Add("offline_access");
        options.SaveTokens = true;
    });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireMfa", policyIsAdminRequirement =>
        {
            policyIsAdminRequirement.Requirements.Add(new RequireMfa());
        });
    });

    services.AddRazorPages();
}
```

<span data-ttu-id="88e6c-209">Bu ilke daha sonra Razor sayfasında gerektiği şekilde kullanılır.</span><span class="sxs-lookup"><span data-stu-id="88e6c-209">This policy is then used in the Razor page as required.</span></span> <span data-ttu-id="88e6c-210">İlke, tüm uygulama için genel olarak eklenebilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-210">The policy could be added globally for the entire app as well.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;

namespace AspNetCoreRequireMfaOidc.Pages
{
    [Authorize(Policy= "RequireMfa")]
    public class IndexModel : PageModel
    {
        private readonly ILogger<IndexModel> _logger;

        public IndexModel(ILogger<IndexModel> logger)
        {
            _logger = logger;
        }

        public void OnGet()
        {
        }
    }
}
```

<span data-ttu-id="88e6c-211">Kullanıcı MFA olmadan kimlik doğrulaması gerçekleştiriyorsa, `amr` talebi muhtemelen bir `pwd` değerine sahip olur.</span><span class="sxs-lookup"><span data-stu-id="88e6c-211">If the user authenticates without MFA, the `amr` claim will probably have a `pwd` value.</span></span> <span data-ttu-id="88e6c-212">İsteğin sayfaya erişim yetkisi yok.</span><span class="sxs-lookup"><span data-stu-id="88e6c-212">The request won't be authorized to access the page.</span></span> <span data-ttu-id="88e6c-213">Varsayılan değerleri kullanarak, Kullanıcı *Hesap/Accessreddedildi* sayfasına yönlendirilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-213">Using the default values, the user will be redirected to the *Account/AccessDenied* page.</span></span> <span data-ttu-id="88e6c-214">Bu davranış değiştirilebilir veya kendi özel mantığınızı buradan uygulayabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="88e6c-214">This behavior can be changed or you can implement your own custom logic here.</span></span> <span data-ttu-id="88e6c-215">Bu örnekte, geçerli kullanıcının, hesabı için MFA ayarlayabilmesi için bir bağlantı eklenir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-215">In this example, a link is added so that the valid user can set up MFA for their account.</span></span>

```cshtml
@page
@model AspNetCoreRequireMfaOidc.AccessDeniedModel
@{
    ViewData["Title"] = "AccessDenied";
    Layout = "~/Pages/Shared/_Layout.cshtml";
}

<h1>AccessDenied</h1>

You require MFA to login here

<a href="https://localhost:44352/Manage/TwoFactorAuthentication">Enable MFA</a>
```

<span data-ttu-id="88e6c-216">Artık yalnızca MFA ile kimlik doğrulaması yapan kullanıcılar sayfaya veya Web sitesine erişebilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-216">Now only users that authenticate with MFA can access the page or website.</span></span> <span data-ttu-id="88e6c-217">Farklı MFA türleri kullanılıyorsa veya 2FA tamam ise, `amr` talebi farklı değerlere sahip olur ve doğru şekilde işlenmek üzere gereklidir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-217">If different MFA types are used or if 2FA is okay, the `amr` claim will have different values and needs to be processed correctly.</span></span> <span data-ttu-id="88e6c-218">Farklı açık KIMLIK Connect sunucuları Ayrıca bu talep için farklı değerler döndürür ve [kimlik doğrulama yöntemi başvuru değerleri](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08) belirtimini izlemeyebilir.</span><span class="sxs-lookup"><span data-stu-id="88e6c-218">Different Open ID Connect servers also return different values for this claim and might not follow the [Authentication Method Reference Values](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08) specification.</span></span>

<span data-ttu-id="88e6c-219">MFA olmadan oturum açarken (örneğin, yalnızca bir parola kullanarak):</span><span class="sxs-lookup"><span data-stu-id="88e6c-219">When logging in without MFA (for example, using just a password):</span></span>

* <span data-ttu-id="88e6c-220">`amr` `pwd` değeri vardır:</span><span class="sxs-lookup"><span data-stu-id="88e6c-220">The `amr` has the `pwd` value:</span></span>

    ![require_mfa_oidc_02. png](mfa/_static/require_mfa_oidc_02.png)

* <span data-ttu-id="88e6c-222">Erişim reddedildi:</span><span class="sxs-lookup"><span data-stu-id="88e6c-222">Access is denied:</span></span>

    ![require_mfa_oidc_03. png](mfa/_static/require_mfa_oidc_03.png)

<span data-ttu-id="88e6c-224">Alternatif olarak, Identityile OTP kullanarak oturum açılıyor:</span><span class="sxs-lookup"><span data-stu-id="88e6c-224">Alternatively, logging in using OTP with Identity:</span></span>

![require_mfa_oidc_01. png](mfa/_static/require_mfa_oidc_01.png)

## <a name="additional-resources"></a><span data-ttu-id="88e6c-226">Ek kaynaklar</span><span class="sxs-lookup"><span data-stu-id="88e6c-226">Additional resources</span></span>

* [<span data-ttu-id="88e6c-227">ASP.NET Core 'daki TOTP Authenticator uygulamaları için QR kodu oluşturmayı etkinleştirme</span><span class="sxs-lookup"><span data-stu-id="88e6c-227">Enable QR Code generation for TOTP authenticator apps in ASP.NET Core</span></span>](xref:security/authentication/identity-enable-qrcodes)
* [<span data-ttu-id="88e6c-228">Azure Active Directory için passwordless kimlik doğrulama seçenekleri</span><span class="sxs-lookup"><span data-stu-id="88e6c-228">Passwordless authentication options for Azure Active Directory</span></span>](/azure/active-directory/authentication/concept-authentication-passwordless)
* [<span data-ttu-id="88e6c-229">.NET kullanarak FIDO2/WebAuthn kanıtlama ve onaylama için FIDO2 .NET kitaplığı</span><span class="sxs-lookup"><span data-stu-id="88e6c-229">FIDO2 .NET library for FIDO2 / WebAuthn Attestation and Assertion using .NET</span></span>](https://github.com/abergs/fido2-net-lib)
* [<span data-ttu-id="88e6c-230">WebAuthn başar</span><span class="sxs-lookup"><span data-stu-id="88e6c-230">WebAuthn Awesome</span></span>](https://github.com/herrjemand/awesome-webauthn)