Proof Key for Code Exchange Nedir?
Single Page Application(SPA) ve Mobil uygulamalara client secret gibi kritik olan değerleri göndermek oldukça tehlike arz edebilmektedir. Misal, SPA’lar özünde JavaScript oldukları için, kendilerine gönderilen çoğu veriyi tarayıcıda tutmakta ve işlemlerini tarayıcılar üzerinde gerçekleştirmektedirler. Biliyorsunuz ki, tarayıcı üzerinde tutulan verilere kullanıcılar az çabayla erişebilmektedirler. Bu durumda client secret gibi önemli bir değeri tarayıcı üzerinde çalışan SPA gibi uygulamalara göndermek, kötü niyetli kişiler tarafından elde edilmesini ve türlü saldırılara yahut sızıntılara mahal verilmesini sağlayabilir. Aynı şekilde bu durum mobil uygulamalar içinde geçerlidir. Mobil uygulamalar, tersine mühendislikle çok rahat deşifre edilebilmekte ve kendilerine gönderilen kritik verileri kötü niyetli kullanıcılara ister istemez kaptırabilmektedirler.

İşte, OAuth 2.0 ve Open Id Connect protokollerini kullanırken yahut bu protokolleri benimsemiş olan IdentityServer4 gibi framework’ler de çalışırken client’a gönderilmesi gereken client secret gibi kritik değerleri mevcudiyette olan böyle bir riske karşı göndermek yerine güvenlik açısından farklı çözümler düşünülmüş ve Proff Key for Code Exchange yöntemi tasarlanmıştır.

PKCE, kullanıcı kimliğinin doğrulanmasını talep eden client’ın, access token elde edebilmesi için tutması gereken secret gibi kritik bilgiler yerine farklı değerler üreterek doğrulama işleminin güvenli bir şekilde yapılmasını sağlamaktadır. Server, kendisine gelen talep içerisinde kod değişimi yapılabilecek bir doğrulayıcı ile yapacağı doğrulama neticesinde güvenli bir kullanıcı doğrulaması gerçekleştirecek ve böylece kötü niyetli client’lardan korunmuş olacaktır.

Şimdi burada işlevsellik gösterecek iki aktörümüzü net tanıyalım, tanımlayalım;

* code_verifier

Client tarafından yapılan access token talebi neticesinde doğrulama işlemi için kullanılan random üretilmiş bir koddur.

* code_challenge

code_verifier’ı doğrulamak için client’a gönderilmiş olan random üretilmiş bir koddur.

Client, authorization code talebinde bulunduğunda öncelikle client’a ait code_challenge ve code_verifier üretilir. code_verifier authorization code ile birlikte client’a gönderilirken, code_challenge Auth Server’da bu client’a özel kaydedilir. Ardından authorization code’u alan client, access token isteğinde bulunabilmek için code_verifier‘ı da istekte gönderir. Auth Server, gelen istekteki code_verifier‘ı alır ve önceden oluşturulup ilgili client’a dair tutulan code_challenge değeri ile karşılaştırır. Eğer bu karşılaştırma neticesinde doğrulama gerçekleştirilirse client access token’ı elde eder. Böylece ilgili client, secret değeri yerine sadece code_verifier ve code_challenge değerlerini kullanmış olacaktır ve olası risk ortadan kalkacaktı

https://www.gencayyildiz.com/blog/identityserver4-yazi-serisi-20-pkceproof-key-for-code-exchange

https://learn.microsoft.com/tr-tr/azure/active-directory/develop/v2-oauth2-auth-code-flow



# Auth0 Pkce

![alt text](https://images.ctfassets.net/cdy7uua7fh8z/3pstjSYx3YNSiJQnwKZvm5/33c941faf2e0c434a9ab1f0f3a06e13a/auth-sequence-auth-code-pkce.png)

# Auth0 QuickStart projesi


### 1. Auth0 Bilgileri
## https://auth0.com/ sitesine kayıt olduktan sonra bir kullanıcı panelinden bir proje oluşturup gerekli bilgileri uygulamaya eki
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuth0WebAppAuthentication(options => {
        options.Domain = Configuration["Auth0:Domain"]; 
        options.ClientId = Configuration["Auth0:ClientId"];
    });
}
```

### 2. middleware ekleme

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...
    app.UseAuthentication();
    app.UseAuthorization();
    ...
}
```
### 3. Login

```csharp
public async Task Login(string returnUrl = "/")
{
    var authenticationProperties = new LoginAuthenticationPropertiesBuilder()
        .WithRedirectUri(returnUrl)
        .Build();

    await HttpContext.ChallengeAsync(Auth0Constants.AuthenticationScheme, authenticationProperties);
}

```

### 4. Kullanıcı sayfası

```csharp
[Authorize]
public IActionResult Profile()
{
    return View(new UserProfileViewModel()
    {
        Name = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.Name)?.Value,
        EmailAddress = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.Email)?.Value,
        ProfileImage = User.Claims.FirstOrDefault(c => c.Type == "picture")?.Value
    });
}
```

### 5. Logout

```csharp
[Authorize]
public async Task Logout()
{
    var authenticationProperties = new LogoutAuthenticationPropertiesBuilder()
        // Indicate here where Auth0 should redirect the user after a logout.
        // Note that the resulting absolute Uri must be whitelisted in the
        // **Allowed Logout URLs** settings for the client.
        .WithRedirectUri(Url.Action("Index", "Home"))
        .Build();
        
    await HttpContext.SignOutAsync(Auth0Constants.AuthenticationScheme, authenticationProperties);
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
}
```
