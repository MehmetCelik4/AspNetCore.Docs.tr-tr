<span data-ttu-id="6781e-101">Dizin sayfası (*Wwwroot/index.html*) sayfası JavaScript içinde `AuthenticationService` tanımlayan bir komut dosyası içerir.</span><span class="sxs-lookup"><span data-stu-id="6781e-101">The Index page (*wwwroot/index.html*) page includes a script that defines the `AuthenticationService` in JavaScript.</span></span> <span data-ttu-id="6781e-102">`AuthenticationService` OıDC protokolünün alt düzey ayrıntılarını işler.</span><span class="sxs-lookup"><span data-stu-id="6781e-102">`AuthenticationService` handles the low-level details of the the OIDC protocol.</span></span> <span data-ttu-id="6781e-103">Uygulama, kimlik doğrulama işlemlerini gerçekleştirmek için komut dosyasında tanımlanan yöntemleri dahili olarak çağırır.</span><span class="sxs-lookup"><span data-stu-id="6781e-103">The app internally calls methods defined in the script to perform the authentication operations.</span></span>

```html
<script src="_content/Microsoft.Authentication.WebAssembly.Msal/
    AuthenticationService.js"></script>
```