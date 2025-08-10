# Sample Scenario: Complete JWT + Identity Implementation

## User Request
"Add JWT authentication with refresh tokens to my Blazor Server app with role-based authorization"

## Phase 1: Orchestrator Initialization

### Input to dotnet-orchestrator:
```json
{
  "task": "implement_authentication",
  "goal": "Add JWT auth with refresh tokens and roles",
  "context": {
    "project_type": "blazor_server",
    "existing_structure": "Clean Architecture with EF Core",
    "constraints": ["maintain existing user data", "support multiple roles"]
  },
  "requirements": {
    "functional": [
      "User registration and login",
      "JWT token generation",
      "Refresh token mechanism",
      "Role-based authorization"
    ],
    "non_functional": [
      "15-minute access tokens",
      "7-day refresh tokens",
      "Secure token storage"
    ]
  }
}
```

### Orchestrator Response:
```json
{
  "execution_plan": {
    "phases": 5,
    "estimated_time": "30 minutes",
    "agents_sequence": [
      "dotnet-backend-architect",
      "ef-core-entity-modeler",
      "aspnet-core-api-developer",
      "blazor-component-architect",
      "identity-auth-specialist"
    ]
  }
}
```

## Phase 2: Architecture Design

### Agent: dotnet-backend-architect

**Input:**
```json
{
  "task": "architecture_design",
  "goal": "Design JWT authentication architecture",
  "requirements": {
    "patterns": ["Clean Architecture", "Repository"],
    "services": ["IAuthService", "ITokenService", "IUserService"],
    "integration": ["Existing User entity", "EF Core DbContext"]
  }
}
```

**Output Files:**

`/src/MyApp.Application/Common/Interfaces/IAuthenticationService.cs`
```csharp
namespace MyApp.Application.Common.Interfaces;

public interface IAuthenticationService
{
    Task<AuthenticationResult> LoginAsync(string email, string password);
    Task<AuthenticationResult> RefreshTokenAsync(string token, string refreshToken);
    Task<RegistrationResult> RegisterAsync(RegisterRequest request);
    Task RevokeTokenAsync(string refreshToken);
}

public interface ITokenService
{
    string GenerateAccessToken(ApplicationUser user, IList<string> roles);
    string GenerateRefreshToken();
    ClaimsPrincipal ValidateToken(string token);
    Task<bool> ValidateRefreshTokenAsync(string refreshToken, string userId);
}

public interface ICurrentUserService
{
    string? UserId { get; }
    string? Email { get; }
    bool IsAuthenticated { get; }
    IEnumerable<string> Roles { get; }
}
```

`/src/MyApp.Application/Authentication/Models/AuthenticationModels.cs`
```csharp
namespace MyApp.Application.Authentication.Models;

public record AuthenticationResult
{
    public bool Succeeded { get; init; }
    public string? AccessToken { get; init; }
    public string? RefreshToken { get; init; }
    public DateTime? ExpiresAt { get; init; }
    public UserDto? User { get; init; }
    public IEnumerable<string> Errors { get; init; } = Array.Empty<string>();
}

public record RegisterRequest(
    string Email,
    string Password,
    string FirstName,
    string LastName,
    string? PhoneNumber = null
);

public record LoginRequest(
    string Email,
    string Password,
    bool RememberMe = false
);

public record RefreshTokenRequest(
    string AccessToken,
    string RefreshToken
);
```

## Phase 3: Database Models

### Agent: ef-core-entity-modeler

**Input:**
```json
{
  "task": "entity_modeling",
  "goal": "Create Identity and refresh token entities",
  "requirements": {
    "entities": ["ApplicationUser", "ApplicationRole", "RefreshToken"],
    "features": ["soft-delete", "auditing"],
    "relationships": ["User has many RefreshTokens"]
  }
}
```

**Output Files:**

`/src/MyApp.Domain/Entities/ApplicationUser.cs`
```csharp
namespace MyApp.Domain.Entities;

public class ApplicationUser : IdentityUser<Guid>, IAuditable, ISoftDelete
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string FullName => $"{FirstName} {LastName}";
    
    // Audit fields
    public DateTime Created { get; set; }
    public string? CreatedBy { get; set; }
    public DateTime? LastModified { get; set; }
    public string? LastModifiedBy { get; set; }
    
    // Soft delete
    public bool IsDeleted { get; set; }
    public DateTime? DeletedAt { get; set; }
    
    // Navigation properties
    public virtual ICollection<RefreshToken> RefreshTokens { get; set; } = new List<RefreshToken>();
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; } = new List<ApplicationUserRole>();
}

public class RefreshToken : BaseEntity<Guid>
{
    public string Token { get; set; } = string.Empty;
    public string JwtId { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
    public DateTime ExpiresAt { get; set; }
    public bool IsUsed { get; set; }
    public bool IsRevoked { get; set; }
    public string? ReplacedByToken { get; set; }
    
    // Navigation
    public Guid UserId { get; set; }
    public virtual ApplicationUser User { get; set; } = null!;
}
```

`/src/MyApp.Infrastructure/Persistence/Configurations/IdentityConfiguration.cs`
```csharp
namespace MyApp.Infrastructure.Persistence.Configurations;

public class ApplicationUserConfiguration : IEntityTypeConfiguration<ApplicationUser>
{
    public void Configure(EntityTypeBuilder<ApplicationUser> builder)
    {
        builder.Property(u => u.FirstName).HasMaxLength(100).IsRequired();
        builder.Property(u => u.LastName).HasMaxLength(100).IsRequired();
        
        builder.HasQueryFilter(u => !u.IsDeleted);
        
        builder.HasMany(u => u.RefreshTokens)
            .WithOne(rt => rt.User)
            .HasForeignKey(rt => rt.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}

public class RefreshTokenConfiguration : IEntityTypeConfiguration<RefreshToken>
{
    public void Configure(EntityTypeBuilder<RefreshToken> builder)
    {
        builder.HasKey(rt => rt.Id);
        
        builder.Property(rt => rt.Token)
            .HasMaxLength(500)
            .IsRequired();
        
        builder.Property(rt => rt.JwtId)
            .HasMaxLength(100)
            .IsRequired();
        
        builder.HasIndex(rt => rt.Token).IsUnique();
        builder.HasIndex(rt => new { rt.UserId, rt.IsUsed, rt.IsRevoked });
    }
}
```

`Migration: 20240131_AddIdentityAndRefreshTokens.cs`
```csharp
public partial class AddIdentityAndRefreshTokens : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Add Identity tables
        migrationBuilder.CreateTable(
            name: "AspNetRoles",
            columns: table => new
            {
                Id = table.Column<Guid>(type: "uniqueidentifier", nullable: false),
                Name = table.Column<string>(type: "nvarchar(256)", maxLength: 256, nullable: true),
                NormalizedName = table.Column<string>(type: "nvarchar(256)", maxLength: 256, nullable: true),
                ConcurrencyStamp = table.Column<string>(type: "nvarchar(max)", nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_AspNetRoles", x => x.Id);
            });
        
        // Add RefreshTokens table
        migrationBuilder.CreateTable(
            name: "RefreshTokens",
            columns: table => new
            {
                Id = table.Column<Guid>(type: "uniqueidentifier", nullable: false),
                Token = table.Column<string>(type: "nvarchar(500)", maxLength: 500, nullable: false),
                JwtId = table.Column<string>(type: "nvarchar(100)", maxLength: 100, nullable: false),
                CreatedAt = table.Column<DateTime>(type: "datetime2", nullable: false),
                ExpiresAt = table.Column<DateTime>(type: "datetime2", nullable: false),
                IsUsed = table.Column<bool>(type: "bit", nullable: false),
                IsRevoked = table.Column<bool>(type: "bit", nullable: false),
                ReplacedByToken = table.Column<string>(type: "nvarchar(500)", nullable: true),
                UserId = table.Column<Guid>(type: "uniqueidentifier", nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_RefreshTokens", x => x.Id);
                table.ForeignKey(
                    name: "FK_RefreshTokens_AspNetUsers_UserId",
                    column: x => x.UserId,
                    principalTable: "AspNetUsers",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
        
        migrationBuilder.CreateIndex(
            name: "IX_RefreshTokens_Token",
            table: "RefreshTokens",
            column: "Token",
            unique: true);
    }
}
```

## Phase 4: API Implementation

### Agent: aspnet-core-api-developer

**Input:**
```json
{
  "task": "api_development",
  "goal": "Create authentication API endpoints",
  "endpoints": [
    "POST /api/auth/register",
    "POST /api/auth/login",
    "POST /api/auth/refresh",
    "POST /api/auth/revoke"
  ]
}
```

**Output Files:**

`/src/MyApp.WebApi/Controllers/AuthController.cs`
```csharp
namespace MyApp.WebApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthenticationService _authService;
    private readonly ILogger<AuthController> _logger;
    
    public AuthController(
        IAuthenticationService authService,
        ILogger<AuthController> logger)
    {
        _authService = authService;
        _logger = logger;
    }
    
    [HttpPost("register")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Register([FromBody] RegisterRequest request)
    {
        var result = await _authService.RegisterAsync(request);
        
        if (!result.Succeeded)
        {
            return BadRequest(new ValidationProblemDetails
            {
                Errors = result.Errors.ToDictionary(e => e.Code, e => new[] { e.Description })
            });
        }
        
        return Ok(new
        {
            accessToken = result.AccessToken,
            refreshToken = result.RefreshToken,
            expiresAt = result.ExpiresAt,
            user = result.User
        });
    }
    
    [HttpPost("login")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var result = await _authService.LoginAsync(request.Email, request.Password);
        
        if (!result.Succeeded)
        {
            return Unauthorized(new { message = "Invalid email or password" });
        }
        
        // Set refresh token in HTTP-only cookie
        Response.Cookies.Append("refreshToken", result.RefreshToken!,
            new CookieOptions
            {
                HttpOnly = true,
                Secure = true,
                SameSite = SameSiteMode.Strict,
                Expires = DateTimeOffset.UtcNow.AddDays(7)
            });
        
        return Ok(new
        {
            accessToken = result.AccessToken,
            expiresAt = result.ExpiresAt,
            user = result.User
        });
    }
    
    [HttpPost("refresh")]
    [ProducesResponseType(typeof(AuthenticationResult), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<IActionResult> Refresh([FromBody] RefreshTokenRequest request)
    {
        var refreshToken = Request.Cookies["refreshToken"] ?? request.RefreshToken;
        
        var result = await _authService.RefreshTokenAsync(request.AccessToken, refreshToken);
        
        if (!result.Succeeded)
        {
            return Unauthorized(new { message = "Invalid token" });
        }
        
        // Update refresh token cookie
        Response.Cookies.Append("refreshToken", result.RefreshToken!,
            new CookieOptions
            {
                HttpOnly = true,
                Secure = true,
                SameSite = SameSiteMode.Strict,
                Expires = DateTimeOffset.UtcNow.AddDays(7)
            });
        
        return Ok(new
        {
            accessToken = result.AccessToken,
            expiresAt = result.ExpiresAt
        });
    }
    
    [HttpPost("revoke")]
    [Authorize]
    public async Task<IActionResult> Revoke()
    {
        var refreshToken = Request.Cookies["refreshToken"];
        
        if (!string.IsNullOrEmpty(refreshToken))
        {
            await _authService.RevokeTokenAsync(refreshToken);
        }
        
        Response.Cookies.Delete("refreshToken");
        
        return NoContent();
    }
}
```

`/src/MyApp.Infrastructure/Services/JwtService.cs`
```csharp
namespace MyApp.Infrastructure.Services;

public class JwtService : ITokenService
{
    private readonly JwtSettings _jwtSettings;
    private readonly IDateTime _dateTime;
    
    public JwtService(IOptions<JwtSettings> jwtSettings, IDateTime dateTime)
    {
        _jwtSettings = jwtSettings.Value;
        _dateTime = dateTime;
    }
    
    public string GenerateAccessToken(ApplicationUser user, IList<string> roles)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_jwtSettings.Secret);
        
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new(JwtRegisteredClaimNames.Email, user.Email!),
            new("fullName", user.FullName),
            new(JwtRegisteredClaimNames.Iat, 
                new DateTimeOffset(_dateTime.Now).ToUnixTimeSeconds().ToString(), 
                ClaimValueTypes.Integer64)
        };
        
        // Add role claims
        claims.AddRange(roles.Select(role => new Claim(ClaimTypes.Role, role)));
        
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = _dateTime.Now.AddMinutes(_jwtSettings.AccessTokenExpirationMinutes),
            Issuer = _jwtSettings.Issuer,
            Audience = _jwtSettings.Audience,
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key), 
                SecurityAlgorithms.HmacSha256Signature)
        };
        
        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
    
    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomNumber);
        return Convert.ToBase64String(randomNumber);
    }
    
    public ClaimsPrincipal ValidateToken(string token)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_jwtSettings.Secret);
        
        var validationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ValidateIssuer = true,
            ValidIssuer = _jwtSettings.Issuer,
            ValidateAudience = true,
            ValidAudience = _jwtSettings.Audience,
            ValidateLifetime = false, // Don't validate expiry for refresh
            ClockSkew = TimeSpan.Zero
        };
        
        var principal = tokenHandler.ValidateToken(token, validationParameters, out _);
        return principal;
    }
}
```

## Phase 5: Blazor Components

### Agent: blazor-component-architect

**Input:**
```json
{
  "task": "component_design",
  "goal": "Create authentication UI components",
  "components": [
    "LoginForm",
    "RegisterForm",
    "AuthProvider",
    "AuthorizedView"
  ]
}
```

**Output Files:**

`/src/MyApp.BlazorUI/Components/Auth/LoginForm.razor`
```razor
@inject IAuthenticationClient AuthClient
@inject NavigationManager Navigation
@inject IToastService Toast

<EditForm Model="@_model" OnValidSubmit="HandleLogin" class="login-form">
    <DataAnnotationsValidator />
    
    <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <InputText @bind-Value="_model.Email" 
                   class="form-control" 
                   id="email" 
                   placeholder="email@example.com" />
        <ValidationMessage For="() => _model.Email" />
    </div>
    
    <div class="mb-3">
        <label for="password" class="form-label">Password</label>
        <InputText @bind-Value="_model.Password" 
                   type="password" 
                   class="form-control" 
                   id="password" />
        <ValidationMessage For="() => _model.Password" />
    </div>
    
    <div class="mb-3 form-check">
        <InputCheckbox @bind-Value="_model.RememberMe" 
                       class="form-check-input" 
                       id="rememberMe" />
        <label class="form-check-label" for="rememberMe">
            Remember me
        </label>
    </div>
    
    @if (_isLoading)
    {
        <button class="btn btn-primary" type="button" disabled>
            <span class="spinner-border spinner-border-sm me-2" role="status"></span>
            Signing in...
        </button>
    }
    else
    {
        <button type="submit" class="btn btn-primary">Sign In</button>
    }
    
    @if (!string.IsNullOrEmpty(_errorMessage))
    {
        <div class="alert alert-danger mt-3" role="alert">
            @_errorMessage
        </div>
    }
</EditForm>

@code {
    private LoginModel _model = new();
    private bool _isLoading;
    private string? _errorMessage;
    
    [Parameter] public EventCallback<AuthenticationResult> OnLoginSuccess { get; set; }
    [Parameter] public string? ReturnUrl { get; set; }
    
    private async Task HandleLogin()
    {
        _isLoading = true;
        _errorMessage = null;
        
        try
        {
            var result = await AuthClient.LoginAsync(_model);
            
            if (result.Succeeded)
            {
                await OnLoginSuccess.InvokeAsync(result);
                
                Toast.ShowSuccess("Login successful!");
                
                var returnUrl = ReturnUrl ?? "/";
                Navigation.NavigateTo(returnUrl);
            }
            else
            {
                _errorMessage = string.Join(", ", result.Errors);
            }
        }
        catch (Exception ex)
        {
            _errorMessage = "An error occurred during login. Please try again.";
            Toast.ShowError("Login failed: " + ex.Message);
        }
        finally
        {
            _isLoading = false;
        }
    }
    
    public class LoginModel
    {
        [Required]
        [EmailAddress]
        public string Email { get; set; } = string.Empty;
        
        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; } = string.Empty;
        
        public bool RememberMe { get; set; }
    }
}
```

`/src/MyApp.BlazorUI/Services/CustomAuthStateProvider.cs`
```csharp
namespace MyApp.BlazorUI.Services;

public class CustomAuthStateProvider : AuthenticationStateProvider
{
    private readonly ILocalStorageService _localStorage;
    private readonly IAuthenticationClient _authClient;
    private readonly NavigationManager _navigation;
    
    private readonly AuthenticationState _anonymous = 
        new(new ClaimsPrincipal(new ClaimsIdentity()));
    
    public CustomAuthStateProvider(
        ILocalStorageService localStorage,
        IAuthenticationClient authClient,
        NavigationManager navigation)
    {
        _localStorage = localStorage;
        _authClient = authClient;
        _navigation = navigation;
    }
    
    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        try
        {
            var token = await _localStorage.GetItemAsync<string>("accessToken");
            
            if (string.IsNullOrEmpty(token))
            {
                return _anonymous;
            }
            
            var claims = ParseClaimsFromJwt(token);
            var expiry = claims.FirstOrDefault(c => c.Type == "exp")?.Value;
            
            if (expiry != null)
            {
                var expiryDateTime = DateTimeOffset.FromUnixTimeSeconds(long.Parse(expiry));
                
                if (expiryDateTime < DateTimeOffset.UtcNow)
                {
                    // Token expired, try to refresh
                    var refreshed = await RefreshTokenAsync();
                    
                    if (!refreshed)
                    {
                        await ClearAuthenticationAsync();
                        return _anonymous;
                    }
                    
                    // Re-parse claims from new token
                    token = await _localStorage.GetItemAsync<string>("accessToken");
                    claims = ParseClaimsFromJwt(token!);
                }
            }
            
            var identity = new ClaimsIdentity(claims, "jwt");
            var user = new ClaimsPrincipal(identity);
            
            return new AuthenticationState(user);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error getting auth state: {ex.Message}");
            return _anonymous;
        }
    }
    
    public async Task MarkUserAsAuthenticated(AuthenticationResult result)
    {
        await _localStorage.SetItemAsync("accessToken", result.AccessToken);
        
        var claims = ParseClaimsFromJwt(result.AccessToken!);
        var identity = new ClaimsIdentity(claims, "jwt");
        var user = new ClaimsPrincipal(identity);
        
        NotifyAuthenticationStateChanged(Task.FromResult(new AuthenticationState(user)));
    }
    
    public async Task MarkUserAsLoggedOut()
    {
        await ClearAuthenticationAsync();
        NotifyAuthenticationStateChanged(Task.FromResult(_anonymous));
        _navigation.NavigateTo("/login");
    }
    
    private async Task<bool> RefreshTokenAsync()
    {
        try
        {
            var result = await _authClient.RefreshTokenAsync();
            
            if (result.Succeeded)
            {
                await _localStorage.SetItemAsync("accessToken", result.AccessToken);
                return true;
            }
        }
        catch
        {
            // Refresh failed
        }
        
        return false;
    }
    
    private async Task ClearAuthenticationAsync()
    {
        await _localStorage.RemoveItemAsync("accessToken");
        await _authClient.RevokeTokenAsync();
    }
    
    private IEnumerable<Claim> ParseClaimsFromJwt(string jwt)
    {
        var payload = jwt.Split('.')[1];
        var jsonBytes = ParseBase64WithoutPadding(payload);
        var keyValuePairs = JsonSerializer.Deserialize<Dictionary<string, object>>(jsonBytes)!;
        
        return keyValuePairs.Select(kvp => new Claim(kvp.Key, kvp.Value.ToString()!));
    }
    
    private byte[] ParseBase64WithoutPadding(string base64)
    {
        switch (base64.Length % 4)
        {
            case 2: base64 += "=="; break;
            case 3: base64 += "="; break;
        }
        return Convert.FromBase64String(base64);
    }
}
```

## Phase 6: Configuration

### Agent: identity-auth-specialist

**Input:**
```json
{
  "task": "configure_authentication",
  "goal": "Configure Identity and JWT in Program.cs",
  "requirements": {
    "identity_options": ["password policy", "lockout settings"],
    "jwt_settings": ["secret key", "expiration", "issuer/audience"],
    "authorization_policies": ["Admin", "Manager", "User"]
  }
}
```

**Output Files:**

`/src/MyApp.WebApi/Program.cs`
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add configuration
builder.Services.Configure<JwtSettings>(
    builder.Configuration.GetSection(nameof(JwtSettings)));

// Add database context
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add Identity
builder.Services.AddIdentity<ApplicationUser, ApplicationRole>(options =>
    {
        // Password settings
        options.Password.RequireDigit = true;
        options.Password.RequiredLength = 8;
        options.Password.RequireNonAlphanumeric = true;
        options.Password.RequireUppercase = true;
        options.Password.RequireLowercase = true;
        
        // Lockout settings
        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.Lockout.AllowedForNewUsers = true;
        
        // User settings
        options.User.RequireUniqueEmail = true;
    })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

// Add JWT Authentication
var jwtSettings = builder.Configuration.GetSection(nameof(JwtSettings)).Get<JwtSettings>();
var key = Encoding.ASCII.GetBytes(jwtSettings!.Secret);

builder.Services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.RequireHttpsMetadata = !builder.Environment.IsDevelopment();
        options.SaveToken = true;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ValidateIssuer = true,
            ValidIssuer = jwtSettings.Issuer,
            ValidateAudience = true,
            ValidAudience = jwtSettings.Audience,
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };
        
        // Support token from cookie for Blazor
        options.Events = new JwtBearerEvents
        {
            OnMessageReceived = context =>
            {
                var accessToken = context.Request.Query["access_token"];
                var path = context.HttpContext.Request.Path;
                
                // If token not in header, check cookie
                if (string.IsNullOrEmpty(accessToken))
                {
                    accessToken = context.Request.Cookies["accessToken"];
                }
                
                if (!string.IsNullOrEmpty(accessToken))
                {
                    context.Token = accessToken;
                }
                
                return Task.CompletedTask;
            }
        };
    });

// Add Authorization
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => 
        policy.RequireRole("Admin"));
    
    options.AddPolicy("ManagerOrAbove", policy => 
        policy.RequireRole("Admin", "Manager"));
    
    options.AddPolicy("AuthenticatedUser", policy => 
        policy.RequireAuthenticatedUser());
    
    options.AddPolicy("EmailVerified", policy =>
        policy.RequireClaim("email_verified", "true"));
});

// Register services
builder.Services.AddScoped<ITokenService, JwtService>();
builder.Services.AddScoped<IAuthenticationService, AuthenticationService>();
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();

// Add controllers
builder.Services.AddControllers();

// Add API documentation
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using Bearer scheme",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });
    
    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});

var app = builder.Build();

// Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

// Seed roles and admin user
using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<ApplicationRole>>();
    var userManager = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
    
    await SeedRolesAndAdminAsync(roleManager, userManager);
}

app.Run();

// Seeding method
async Task SeedRolesAndAdminAsync(
    RoleManager<ApplicationRole> roleManager,
    UserManager<ApplicationUser> userManager)
{
    // Create roles
    string[] roles = { "Admin", "Manager", "User" };
    
    foreach (var role in roles)
    {
        if (!await roleManager.RoleExistsAsync(role))
        {
            await roleManager.CreateAsync(new ApplicationRole { Name = role });
        }
    }
    
    // Create admin user
    var adminEmail = "admin@example.com";
    var adminUser = await userManager.FindByEmailAsync(adminEmail);
    
    if (adminUser == null)
    {
        adminUser = new ApplicationUser
        {
            UserName = adminEmail,
            Email = adminEmail,
            FirstName = "System",
            LastName = "Administrator",
            EmailConfirmed = true
        };
        
        await userManager.CreateAsync(adminUser, "Admin@123456");
        await userManager.AddToRoleAsync(adminUser, "Admin");
    }
}
```

`/src/MyApp.WebApi/appsettings.json`
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyAppDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "JwtSettings": {
    "Secret": "YOUR-256-BIT-SECRET-KEY-KEEP-THIS-SECURE-AND-NEVER-COMMIT-TO-SOURCE-CONTROL",
    "Issuer": "MyApp.WebApi",
    "Audience": "MyApp.BlazorUI",
    "AccessTokenExpirationMinutes": 15,
    "RefreshTokenExpirationDays": 7
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  }
}
```

## Final Orchestration Summary

### Complete File Structure Created:
```
/src
├── MyApp.Domain/
│   └── Entities/
│       ├── ApplicationUser.cs
│       ├── ApplicationRole.cs
│       └── RefreshToken.cs
├── MyApp.Application/
│   ├── Common/
│   │   └── Interfaces/
│   │       ├── IAuthenticationService.cs
│   │       ├── ITokenService.cs
│   │       └── ICurrentUserService.cs
│   └── Authentication/
│       └── Models/
│           └── AuthenticationModels.cs
├── MyApp.Infrastructure/
│   ├── Persistence/
│   │   └── Configurations/
│   │       ├── ApplicationUserConfiguration.cs
│   │       └── RefreshTokenConfiguration.cs
│   └── Services/
│       ├── AuthenticationService.cs
│       ├── JwtService.cs
│       └── CurrentUserService.cs
├── MyApp.WebApi/
│   ├── Controllers/
│   │   └── AuthController.cs
│   ├── Program.cs
│   └── appsettings.json
└── MyApp.BlazorUI/
    ├── Components/
    │   └── Auth/
    │       ├── LoginForm.razor
    │       ├── RegisterForm.razor
    │       └── AuthorizedView.razor
    ├── Services/
    │   ├── CustomAuthStateProvider.cs
    │   └── AuthenticationClient.cs
    └── Program.cs
```

### Orchestration Metrics:
- **Agents Used**: 5
- **Files Created**: 18
- **Lines of Code**: ~1,500
- **Time Estimate**: 30 minutes
- **Phases Completed**: 6

### Final Integration Commands:
```bash
# Apply migrations
dotnet ef database update

# Run the application
dotnet run --project src/MyApp.WebApi

# Access Swagger UI
open https://localhost:5001/swagger

# Test login endpoint
curl -X POST https://localhost:5001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"Admin@123456"}'
```

---

This complete scenario demonstrates how the Agenteers orchestration system coordinates multiple specialized agents to implement a complex feature from architecture to deployment, with each agent contributing their expertise to create a production-ready JWT authentication system.
