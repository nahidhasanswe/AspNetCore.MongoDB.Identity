# AspNetCore.MongoDB.Identity
Microsoft AspNetCore Identity in MongoDB with Mongo CRUD operation

# Nuget Package
https://www.nuget.org/packages/AspNetCore.MongoDB.Identity

# How to use
You have to add the following lines to your Startup.cs in the ASP.NET Core Project.

```C#
services.AddIdentity<ApplicationUser, MongoIdentityRole>(options =>
{
	options.Password.RequireDigit = false;
	options.Password.RequireUppercase = false;
	options.Password.RequireLowercase = false;
	options.Password.RequireNonAlphanumeric = false;
	options.Password.RequiredLength = 6;
}).AddDefaultTokenProviders();

services
	.Configure<MongoDBOption>(Configuration.GetSection("MongoDBOption"))
	.AddMongoDatabase()
	.AddMongoDBIdentity<ApplicationUser, MongoIdentityRole>();
```

```ApplicationUser``` class is must inherited by ```MongoIdentityUser``` as below
```C#
 public class ApplicationUser : MongoIdentityUser
    {

    }
```
```[N.B]: If you want to add extra property in your identity then add property in ApplicationUser class```


In appsettings.json you can configure the correct Path with a new section to your MongoDB instance.

```json
"ConnectionStrings": {
    "DefaultConnection": "Server=xxx"
  },
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MongoDBOption": {
    "ConnectionString": "mongodb://localhost:27017",
    "Database": "AspNetCoreMongoIdentity",
    "User": {
      "CollectionName": "Users",
      "ManageIndicies": true
    },
    "Role": {
      "CollectionName": "Roles",
      "ManageIndicies": true
    }
  }
```

Now you have to write your own entity for MongoDB Operation. Your Entity must inherited from `IMongoEntity`. IMongoEntity contain Bson id, CreatedBy, CreatedDate, UpdatedDate, UpdatedBy, isRemoved. So you will not includ these property to your enity. Your Entity will be looked like as below

```c#
public class SampleModel : IMongoEntity
    {
        public string Name { get; set; }
        public string Address { get; set; }
    }
```
For Mongo CRUD operation you will follow my ```AspNetCore.MongoDB``` library git repository [Here](https://github.com/nahidhasanswe/AspNetCore.MongoDB)

# How to use MongoDB Identity

Now you have to Inject ```UserManager<ApplicationUser>``` and ```SignInManager<ApplicationUser>``` into your Controller and Use Identity as below :

```C#
public class AccountController : Controller
{
	private readonly UserManager<ApplicationUser> _userManager;
	private readonly SignInManager<ApplicationUser> _signInManager;
	private readonly IEmailSender _emailSender;
	private readonly ILogger _logger;

	public AccountController(
		UserManager<ApplicationUser> userManager,
		SignInManager<ApplicationUser> signInManager,
		IEmailSender emailSender,
		ILogger<AccountController> logger)
	{
		_userManager = userManager;
		_signInManager = signInManager;
		_emailSender = emailSender;
		_logger = logger;
	}

	[HttpGet]
	[AllowAnonymous]
	public async Task<IActionResult> Login(string returnUrl = null)
	{
		// Clear the existing external cookie to ensure a clean login process
		await HttpContext.SignOutAsync(IdentityConstants.ExternalScheme);

		ViewData["ReturnUrl"] = returnUrl;
		return View();
	}

	[HttpPost]
	[AllowAnonymous]
	[ValidateAntiForgeryToken]
	public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
	{
		ViewData["ReturnUrl"] = returnUrl;
		if (ModelState.IsValid)
		{
			// This doesn't count login failures towards account lockout
			// To enable password failures to trigger account lockout, set lockoutOnFailure: true
			var result = await _signInManager.PasswordSignInAsync(model.Email, model.Password, model.RememberMe, lockoutOnFailure: false);
			if (result.Succeeded)
			{
				_logger.LogInformation("User logged in.");
				return RedirectToLocal(returnUrl);
			}
			if (result.RequiresTwoFactor)
			{
				return RedirectToAction(nameof(LoginWith2fa), new { returnUrl, model.RememberMe });
			}
			if (result.IsLockedOut)
			{
				_logger.LogWarning("User account locked out.");
				return RedirectToAction(nameof(Lockout));
			}
			else
			{
				ModelState.AddModelError(string.Empty, "Invalid login attempt.");
				return View(model);
			}
		}

		// If we got this far, something failed, redisplay form
		return View(model);
	}
}
```
Now you will successfully integrate Identity with MongoDB.

Identity process follow the ```AspNetCore Identity``` like [Here](https://github.com/aspnet/Identity)

## Contact Me
If you fetch any problem to implementing. Please ask me anytime through mail `nahidh527@gmail.com` and create a issue. Thanks for using this.