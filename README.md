---
services: active-directory
platforms: dotnet
author: dstrockis
---


# 使用OpenID Connect将Azure AD整合到Web应用程序中

这个示例演示如何创建一个使用OpenID Connect来验证来自单个Azure AD租户用户的.NET MVC的Web应用程序，使用到ASP.Net OpenID Connect OWIN中间件。

有关这个协议在这个场景及其他场景下如何工作的更多详细信息，请阅读：[Azure AD 的身份验证方案](https://www.azure.cn/documentation/articles/active-directory-authentication-scenarios/)。

> 查阅GitHub页面的[releases](../../releases)标签以寻找这个代码示例之前的版本。

## 如何运行这个示例

入门很简单！为了运行这个示例你需要：

- Visual Studio 2013
- 连接互联网
- 一个Azure的订阅（如果您还没有Azure订阅，请点击[此处](https://www.azure.cn/)申请试用的订阅账号）

每一个Azure订阅都有一个相关联的Azure Active Directory租户。如果你还没有Azure订阅，你可以在[https://www.azure.cn/](https://www.azure.cn/)注册一个免费的订阅。这个示例使用到的Azure AD的所有功能都是免费的。

### 第一步： 克隆或者下载这个资源库

从你的shell或者命令行：

	git clone https://github.com/wacn-samples/active-directory-dotnet-webapp-openidconnect.git

### 第二步： 在你的Azure Active Directory租户中创建一个用户账号

如果你在Azure Active Directory租户中已经有一个用户账号，你可以跳过此步。如果之前没有在目录中创建用户账户，你现在就需要做这个事情。如果你创建了一个账号并且想使用它登录Azure门户网站，不要忘记将这个用户账号添加成共同管理员。

### 第三步： 在Azure Active Directory租户中注册这个示例

1. 登录[门户网站](https://manage.windowsazure.cn)。
2. 点击左侧导航栏中的Active Directory 。
3. 点击你想注册示例程序的目录租户。
4. 点击程序标签。
5. 点击下方的“添加”。
6. 选择“Web 应用程序和/或 Web API”。
7. 为您的程序输入友好的名字，例如示例“WebApp-OpenIDConnect-DotNet”，然后点击继续。
8. 在登录URL中输入示例的基地址，默认是`https://localhost:44320/`。
9. 在应用程序 ID URI中输入`https://<your_tenant_name>/WebApp-OpenIDConnect-DotNet`，使用你的Azure AD租户的名字替代`<your_tenant_name>`

都完成了！在进入下一个步骤前，你需要找到程序的客户端ID。

1. 如果然在Azure门户，点击你程序的配置标签。
2. 找到客户端ID的值，然后复制到粘贴板。

### 第四步： 配置示例来使用Azure Active Directory租户

1. 在Visual Studio 2013中打开解决方案。
2. 打开`web.config`文件。
3. 找到app key `ida:Tenant`，然后使用你的AAD租户的名字替换掉它的值。
4. 找到app key `ida:ClientId`，然后使用Azure门户中客户端ID的值替换掉它的值
5. 如果你改变了示例的基地址，找到app key `ida:PostLogoutRedirectUri` ，然后使用示例中新的基地址的值替换掉它的值。

### 第五步： 运行示例

清除解决方案，重建解决方案，然后运行它。

点击首页的sign-in的链接来进行登陆。在Azure AD登录界面输入你Azure AD租户里的用户名和密码。

## 如何部署这个示例到Azure

敬请期待。

## 关于代码

这个示例演示如何创建一个使用OpenID Connect来验证来自单个Azure AD租户用户的.NET MVC的Web应用程序。在`Startup.Auth.cs`文件中初始化中间件，通过传入程序的客户端ID和程序注册的Azure AD租户的URL。中间件然后负责：

- 下载Azure AD元数据，找到签名密钥，然后找到租户颁发者名字。
- 通过传入的JWT验证签名和颁发者来处理OpenID Connect登录连接响应，提取用户的要求，然后把他们放在
 ClaimsPrincipal.Current。
- ASP.Net OWIN中间件整合session cookie来建立用户会话。

你可以在类或者方法上打上`[Authorize]`特征来触发中间件发送OpenID Connect登录请求，或者发起质询，

	C#
	HttpContext.GetOwinContext().Authentication.Challenge(
	new AuthenticationProperties { RedirectUri = "/" },
	OpenIdConnectAuthenticationDefaults.AuthenticationType);

同样你可以发送登出请求，

	C#
	HttpContext.GetOwinContext().Authentication.SignOut(
	OpenIdConnectAuthenticationDefaults.AuthenticationType,
	CookieAuthenticationDefaults.AuthenticationType);

当用户登出时，他们会被重定向到OpenID Connect中间件初始化时指定的`Post_Logout_Redirect_Uri`

这个项目中用到所有的OWIN中间件都是开源代码[Katana project](http://katanaproject.codeplex.com)的一部分，你可以[在此](http://owin.org)阅读更多关于OWIN的内容。

## 如何重新创建这个示例

1. 在Visual Studio 2013中创建一个新的MVC Web程序，认证部分选择"No Authentication"。
2. 设置SSL Enabled为true。
3. 在项目属性，Web属性，设置项目的Url为SSL URL。
4. 通过NuGets添加下列 ASP.Net OWIN中间件：
	- Microsoft.IdentityModel.Protocol.Extensions, System.IdentityModel.Tokens.Jwt 
	- Microsoft.Owin.Security.OpenIdConnect, Microsoft.Owin.Security.Cookies 
	- Microsoft.Owin.Host.SystemWeb.
5. 在`App_Start`文件夹中创建`Startup.Auth.cs`类。你需要移除`.App_Start` 命名空间。使用示例中相同文件的代码来代替`Startup` 类中的代码。一定要把整个类定义！定义从`public class Startup` 改变成 `public partial class Startup`。
6. 在 `Startup.Auth.cs`通过`using` 语法添加缺少的引用`Owin`, `Microsoft.Owin.Security`, `Microsoft.Owin.Security.Cookies`, `Microsoft.Owin.Security.OpenIdConnect`, `System.Configuration`, 和 `System.Globalization`。
7. 右击项目，选择添加，选择"OWIN Startup class"，使用"Startup"来命名这个类。如果在菜单中没有找到"OWIN Startup Class"，选择“Class”然后再搜索框中输入"OWIN"来替代，"OWIN Startup class"将显示为选择；选择它，然后使用`Startup.cs`来命名这个类。
8. 使用示例中`Startup.cs`文件中的代码来替代新创建的`Startup.cs`。注意类的定义需要从`public class Startup` 变为 `public partial class Startup`。
9. 在 `Views` --> `Shared`文件夹中创建一个局部视图`_LoginPartial.cshtml`，然后使用示例中的代码来替代其中的内容。
10. 使用示例中 `_Layout.cshtml`的内容来替换`Views` --> `Shared`文件夹中 `_Layout.cshtml`的内容。更有效的，可以添加一行来实现同样的效果，`@Html.Partial("_LoginPartial")`，我们在之前添加的`_LoginPartial`视图已经实现了。
11. 创建一个新的空controller命名为`AccountController`，使用示例中相同文件中的内容来实现它。
12. 如果你希望用户在看到程序中的任何页面时去登录，那么在`HomeController`这个类上放置 `[Authorize]`特征。如果你没有做这项工作，用户就不需要登陆也能看到程序的主页，然后能通过页面中sign-in的链接来登录。
13. 差不多完成了！参考上面“运行示例”的步骤在AAD租户注册程序。
14. 在`web.config`的 `<appSettings>`节点下创建`ida:ClientId`, `ida:AADInstance`, `ida:Tenant`, 和 `ida:PostLogoutRedirectUri`这些键，并赋予对应的值。在中国版Azure AD，`ida:AADInstance`的值是`https://login.chinacloudapi.cn/{0}`。



