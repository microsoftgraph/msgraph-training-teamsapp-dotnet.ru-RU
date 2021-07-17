<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9542e-101">В этом упражнении вы расширит приложение от предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9542e-101">In this exercise you will extend the application from the previous exercise to support single sign-on authentication with Azure AD.</span></span> <span data-ttu-id="9542e-102">Это необходимо для получения маркера доступа OAuth, который нужен для вызова API Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9542e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="9542e-103">На этом шаге будет настроена [библиотека Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="9542e-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="9542e-104">Чтобы не хранить код приложения и секрет в источнике, для хранения этих значений используется [диспетчер секрета .NET.](/aspnet/core/security/app-secrets)</span><span class="sxs-lookup"><span data-stu-id="9542e-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="9542e-105">Секретный менеджер предназначен только для целей разработки, для хранения секретов в производственных приложениях должен использовать доверенный секретный менеджер.</span><span class="sxs-lookup"><span data-stu-id="9542e-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="9542e-106">Откройте **./appsettings.jsи** замените содержимое следующим.</span><span class="sxs-lookup"><span data-stu-id="9542e-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. <span data-ttu-id="9542e-107">Откройте свой CLI в каталоге, в котором расположен **GraphTutorial.csproj,** и запустите следующие команды, заменив свой ID приложения на портале Azure и секрет `YOUR_APP_ID` `YOUR_APP_SECRET` приложения.</span><span class="sxs-lookup"><span data-stu-id="9542e-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="9542e-108">Реализация входа в систему</span><span class="sxs-lookup"><span data-stu-id="9542e-108">Implement sign-in</span></span>

<span data-ttu-id="9542e-109">Во-первых, в коде JavaScript приложения реализована одна входная подпись.</span><span class="sxs-lookup"><span data-stu-id="9542e-109">First, implement single sign-on in the app's JavaScript code.</span></span> <span data-ttu-id="9542e-110">Вы будете использовать Microsoft Teams [JavaScript SDK](/javascript/api/overview/msteams-client) для получения маркера доступа, который позволяет коду JavaScript, запущенным в клиенте Teams, выполнять вызовы AJAX в веб-API, которые будут реализованы позже.</span><span class="sxs-lookup"><span data-stu-id="9542e-110">You will use the [Microsoft Teams JavaScript SDK](/javascript/api/overview/msteams-client) to get an access token which allows the JavaScript code running in the Teams client to make AJAX calls to Web API you will implement later.</span></span>

1. <span data-ttu-id="9542e-111">Откройте **./Pages/Index.cshtml** и добавьте в тег следующий `<script>` код.</span><span class="sxs-lookup"><span data-stu-id="9542e-111">Open **./Pages/Index.cshtml** and add the following code inside the `<script>` tag.</span></span>

    ```javascript
    (function () {
      if (microsoftTeams) {
        microsoftTeams.initialize();

        microsoftTeams.authentication.getAuthToken({
          successCallback: (token) => {
            // TEMPORARY: Display the access token for debugging
            $('#tab-container').empty();

            $('<code/>', {
              text: token,
              style: 'word-break: break-all;'
            }).appendTo('#tab-container');
          },
          failureCallback: (error) => {
            renderError(error);
          }
        });
      }
    })();

    function renderError(error) {
      $('#tab-container').empty();

      $('<h1/>', {
        text: 'Error'
      }).appendTo('#tab-container');

      $('<code/>', {
        text: JSON.stringify(error, Object.getOwnPropertyNames(error)),
        style: 'word-break: break-all;'
      }).appendTo('#tab-container');
    }
    ```

    <span data-ttu-id="9542e-112">Это вызывает бесшумную проверку подлинности в качестве пользователя, который подписан на `microsoftTeams.authentication.getAuthToken` Teams.</span><span class="sxs-lookup"><span data-stu-id="9542e-112">This calls the `microsoftTeams.authentication.getAuthToken` to silently authenticate as the user that is signed in to Teams.</span></span> <span data-ttu-id="9542e-113">Обычно не требуется никаких подсказок пользовательского интерфейса, если только пользователю не требуется согласие.</span><span class="sxs-lookup"><span data-stu-id="9542e-113">There is typically not any UI prompts involved, unless the user has to consent.</span></span> <span data-ttu-id="9542e-114">Затем код отображает маркер на вкладке.</span><span class="sxs-lookup"><span data-stu-id="9542e-114">The code then displays the token in the tab.</span></span>

1. <span data-ttu-id="9542e-115">Сохраните изменения и запустите приложение, выполив следующую команду в CLI.</span><span class="sxs-lookup"><span data-stu-id="9542e-115">Save your changes and start your application by running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="9542e-116">Если вы перезапустили ngrok и url-адрес ngrok изменился, перед тестированием обязательно обновите значение ngrok в следующем месте. </span><span class="sxs-lookup"><span data-stu-id="9542e-116">If you have restarted ngrok and your ngrok URL has changed, be sure to update the ngrok value in the following place **before** you test.</span></span>
    >
    > - <span data-ttu-id="9542e-117">URI перенаправления в регистрации приложения</span><span class="sxs-lookup"><span data-stu-id="9542e-117">The redirect URI in your app registration</span></span>
    > - <span data-ttu-id="9542e-118">ID-URI приложения в регистрации приложения</span><span class="sxs-lookup"><span data-stu-id="9542e-118">The application ID URI in your app registration</span></span>
    > - <span data-ttu-id="9542e-119">`contentUrl` в manifest.jsна</span><span class="sxs-lookup"><span data-stu-id="9542e-119">`contentUrl` in manifest.json</span></span>
    > - <span data-ttu-id="9542e-120">`validDomains` в manifest.jsна</span><span class="sxs-lookup"><span data-stu-id="9542e-120">`validDomains` in manifest.json</span></span>
    > - <span data-ttu-id="9542e-121">`resource` в manifest.jsна</span><span class="sxs-lookup"><span data-stu-id="9542e-121">`resource` in manifest.json</span></span>

1. <span data-ttu-id="9542e-122">Создайте файл ZIP **сmanifest.js,** **color.png** и **outline.png.**</span><span class="sxs-lookup"><span data-stu-id="9542e-122">Create a ZIP file with **manifest.json**, **color.png**, and **outline.png**.</span></span>

1. <span data-ttu-id="9542e-123">В Microsoft Teams выберите **Приложения** в левой панели, выберите Upload настраиваемого **приложения,** а затем выберите Upload для меня или **моей команды**.</span><span class="sxs-lookup"><span data-stu-id="9542e-123">In Microsoft Teams, select **Apps** in the left-hand bar, select **Upload a custom app**, then select **Upload for me or my teams**.</span></span>

    ![Снимок экрана Upload настраиваемой ссылки приложения в Microsoft Teams](images/upload-custom-app.png)

1. <span data-ttu-id="9542e-125">Просмотрите созданный ранее файл ZIP и выберите **Open.**</span><span class="sxs-lookup"><span data-stu-id="9542e-125">Browse to the ZIP file you created previously and select **Open**.</span></span>

1. <span data-ttu-id="9542e-126">Просмотрите сведения о приложении и выберите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="9542e-126">Review the application information and select **Add**.</span></span>

1. <span data-ttu-id="9542e-127">Приложение открывается в Teams и отображает маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="9542e-127">The application opens in Teams and displays an access token.</span></span>

<span data-ttu-id="9542e-128">Если вы скопируете маркер, его можно вклеить [в jwt.ms.](https://jwt.ms)</span><span class="sxs-lookup"><span data-stu-id="9542e-128">If you copy the token, you can paste it into [jwt.ms](https://jwt.ms).</span></span> <span data-ttu-id="9542e-129">Убедитесь, что аудитория (утверждение) — это ваш ID приложения, и единственной областью (утверждением) является созданная область `aud` `scp` `access_as_user` API.</span><span class="sxs-lookup"><span data-stu-id="9542e-129">Verify that the audience (the `aud` claim) is your application ID, and the only scope (the `scp` claim) is the `access_as_user` API scope you created.</span></span> <span data-ttu-id="9542e-130">Это означает, что этот маркер не предоставляет прямой доступ к Microsoft Graph!</span><span class="sxs-lookup"><span data-stu-id="9542e-130">That means that this token does not grant direct access to Microsoft Graph!</span></span> <span data-ttu-id="9542e-131">Вместо этого [веб-API,](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) который будет реализован в ближайшее время, должен будет обменять этот маркер с помощью потока от имени, чтобы получить маркер, который будет работать с вызовами microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9542e-131">Instead, the Web API you will implement soon will need to exchange this token using the [on-behalf-of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to get a token that will work with Microsoft Graph calls.</span></span>

## <a name="configure-authentication-in-the-aspnet-core-app"></a><span data-ttu-id="9542e-132">Настройка проверки подлинности в ASP.NET Core приложении</span><span class="sxs-lookup"><span data-stu-id="9542e-132">Configure authentication in the ASP.NET Core app</span></span>

<span data-ttu-id="9542e-133">Начните с добавления служб платформы microsoft Identity в приложение.</span><span class="sxs-lookup"><span data-stu-id="9542e-133">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="9542e-134">Откройте **файл ./Startup.cs** и добавьте следующее утверждение `using` в верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="9542e-134">Open the **./Startup.cs** file and add the following `using` statement to the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. <span data-ttu-id="9542e-135">Добавьте следующую строку перед `app.UseAuthorization();` строкой в `Configure` функции.</span><span class="sxs-lookup"><span data-stu-id="9542e-135">Add the following line just before the `app.UseAuthorization();` line in the `Configure` function.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="9542e-136">Добавьте следующую строку сразу `endpoints.MapRazorPages();` после строки в `Configure` функции.</span><span class="sxs-lookup"><span data-stu-id="9542e-136">Add the following line just after the `endpoints.MapRazorPages();` line in the `Configure` function.</span></span>

    ```csharp
    endpoints.MapControllers();
    ```

1. <span data-ttu-id="9542e-137">Замените имеющуюся функцию `ConfigureServices` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="9542e-137">Replace the existing `ConfigureServices` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    <span data-ttu-id="9542e-138">Этот код настраивает приложение для проверки подлинности вызовов веб-API на основе маркера носитель JWT в `Authorization` загонах.</span><span class="sxs-lookup"><span data-stu-id="9542e-138">This code configures the application to allow calls to Web APIs to be authenticated based on the JWT bearer token in the `Authorization` header.</span></span> <span data-ttu-id="9542e-139">Он также добавляет службы приобретения маркеров, которые могут обмениваться этим маркером через поток от имени.</span><span class="sxs-lookup"><span data-stu-id="9542e-139">It also adds the token acquisition services that can exchange that token via the on-behalf-of flow.</span></span>

## <a name="create-the-web-api-controller"></a><span data-ttu-id="9542e-140">Создание контроллера веб-API</span><span class="sxs-lookup"><span data-stu-id="9542e-140">Create the Web API controller</span></span>

1. <span data-ttu-id="9542e-141">Создание нового каталога в корне проекта с именем **Контроллеры**.</span><span class="sxs-lookup"><span data-stu-id="9542e-141">Create a new directory in the root of the project named **Controllers**.</span></span>

1. <span data-ttu-id="9542e-142">Создайте новый файл в **каталоге ./Controllers** с именем **CalendarController.cs** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="9542e-142">Create a new file in the **./Controllers** directory named **CalendarController.cs** and add the following code.</span></span>

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Net;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.Resource;
    using Microsoft.Graph;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        [Authorize]
        public class CalendarController : ControllerBase
        {
            private static readonly string[] apiScopes = new[] { "access_as_user" };

            private readonly GraphServiceClient _graphClient;
            private readonly ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<CalendarController> _logger;

            public CalendarController(ITokenAcquisition tokenAcquisition, GraphServiceClient graphClient, ILogger<CalendarController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _graphClient = graphClient;
                _logger = logger;
            }

            [HttpGet]
            public async Task<ActionResult<string>> Get()
            {
                // This verifies that the access_as_user scope is
                // present in the bearer token, throws if not
                HttpContext.VerifyUserHasAnyAcceptedScope(apiScopes);

                // To verify that the identity libraries have authenticated
                // based on the token, log the user's name
                _logger.LogInformation($"Authenticated user: {User.GetDisplayName()}");

                try
                {
                    // TEMPORARY
                    // Get a Graph token via OBO flow
                    var token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(new[]{
                            "User.Read",
                            "MailboxSettings.Read",
                            "Calendars.ReadWrite" });

                    // Log the token
                    _logger.LogInformation($"Access token for Graph: {token}");
                    return Ok("{ \"status\": \"OK\" }");
                }
                catch (MicrosoftIdentityWebChallengeUserException ex)
                {
                    _logger.LogError(ex, "Consent required");
                    // This exception indicates consent is required.
                    // Return a 403 with "consent_required" in the body
                    // to signal to the tab it needs to prompt for consent
                    return new ContentResult {
                        StatusCode = (int)HttpStatusCode.Forbidden,
                        ContentType = "text/plain",
                        Content = "consent_required"
                    };
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error occurred");
                    throw;
                }
            }
        }
    }
    ```

    <span data-ttu-id="9542e-143">При этом реализуется веб-API (), который можно назвать `GET /calendar` с Teams вкладки. Пока он просто пытается обменять маркер носитель на маркер Graph маркера.</span><span class="sxs-lookup"><span data-stu-id="9542e-143">This implements a Web API (`GET /calendar`) that can be called from the Teams tab. For now it simply tries to exchange the bearer token for a Graph token.</span></span> <span data-ttu-id="9542e-144">При первом загрузке вкладки пользователь не сможет разрешить доступ приложения к microsoft Graph от своего имени.</span><span class="sxs-lookup"><span data-stu-id="9542e-144">The first time a user loads the tab, this will fail because they have not yet consented to allow the app access to Microsoft Graph on their behalf.</span></span>

1. <span data-ttu-id="9542e-145">Откройте **./Pages/Index.cshtml** и `successCallback` замените функцию на следующую.</span><span class="sxs-lookup"><span data-stu-id="9542e-145">Open **./Pages/Index.cshtml** and replace the `successCallback` function with the following.</span></span>

    ```javascript
    successCallback: (token) => {
      // TEMPORARY: Call the Web API
      fetch('/calendar', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      }).then(response => {
        response.text()
          .then(body => {
            $('#tab-container').empty();
            $('<code/>', {
              text: body
            }).appendTo('#tab-container');
          });
      }).catch(error => {
        console.error(error);
        renderError(error);
      });
    }
    ```

    <span data-ttu-id="9542e-146">При этом будет вызываться веб-API и отображаться ответ.</span><span class="sxs-lookup"><span data-stu-id="9542e-146">This will call the Web API and display the response.</span></span>

1. <span data-ttu-id="9542e-147">Сохраните изменения и перезапустите приложение.</span><span class="sxs-lookup"><span data-stu-id="9542e-147">Save your changes and restart the app.</span></span> <span data-ttu-id="9542e-148">Обновите вкладку в Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="9542e-148">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="9542e-149">Страница должна `consent_required` отображаться .</span><span class="sxs-lookup"><span data-stu-id="9542e-149">The page should display `consent_required`.</span></span>

1. <span data-ttu-id="9542e-150">Просмотрите выход журнала в CLI.</span><span class="sxs-lookup"><span data-stu-id="9542e-150">Review the log output in your CLI.</span></span> <span data-ttu-id="9542e-151">Обратите внимание на две вещи.</span><span class="sxs-lookup"><span data-stu-id="9542e-151">Notice two things.</span></span>

    - <span data-ttu-id="9542e-152">Запись типа `Authenticated user: MeganB@contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="9542e-152">An entry like `Authenticated user: MeganB@contoso.com`.</span></span> <span data-ttu-id="9542e-153">Веб-API сдал проверку подлинности пользователя на основе маркера, отправленного с запросом API.</span><span class="sxs-lookup"><span data-stu-id="9542e-153">The Web API has authenticated the user based on the token sent with the API request.</span></span>
    - <span data-ttu-id="9542e-154">Запись типа `AADSTS65001: The user or administrator has not consented to use the application with ID...` .</span><span class="sxs-lookup"><span data-stu-id="9542e-154">An entry like `AADSTS65001: The user or administrator has not consented to use the application with ID...`.</span></span> <span data-ttu-id="9542e-155">Это ожидается, так как пользователю еще не было предложено дать согласие на запрашиваемую область Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="9542e-155">This is expected, since the user has not yet been prompted to consent for the requested Microsoft Graph permission scopes.</span></span>

## <a name="implement-consent-prompt"></a><span data-ttu-id="9542e-156">Реализация запроса на согласие</span><span class="sxs-lookup"><span data-stu-id="9542e-156">Implement consent prompt</span></span>

<span data-ttu-id="9542e-157">Поскольку веб-API не может подсказить пользователю, Teams вкладке потребуется реализовать подсказку.</span><span class="sxs-lookup"><span data-stu-id="9542e-157">Because the Web API cannot prompt the user, the Teams tab will need to implement a prompt.</span></span> <span data-ttu-id="9542e-158">Это необходимо сделать только один раз для каждого пользователя.</span><span class="sxs-lookup"><span data-stu-id="9542e-158">This will only need to be done once for each user.</span></span> <span data-ttu-id="9542e-159">После согласия пользователя повторное согласие не требуется, если он явно не отзовет доступ к вашему приложению.</span><span class="sxs-lookup"><span data-stu-id="9542e-159">Once a user consents, they do not need to reconsent unless they explicitly revoke access to your application.</span></span>

1. <span data-ttu-id="9542e-160">Создайте новый файл в **каталоге ./Pages** с именем **Authenticate.cshtml.cs** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="9542e-160">Create a new file in the **./Pages** directory named **Authenticate.cshtml.cs** and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. <span data-ttu-id="9542e-161">Создайте новый файл в **каталоге ./Pages** с именем **Authenticate.cshtml** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="9542e-161">Create a new file in the **./Pages** directory named **Authenticate.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. <span data-ttu-id="9542e-162">Создайте новый файл в **каталоге ./Pages** с именем **AuthComplete.cshtml** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="9542e-162">Create a new file in the **./Pages** directory named **AuthComplete.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. <span data-ttu-id="9542e-163">Откройте **./Pages/Index.cshtml** и добавьте в тег следующие `<script>` функции.</span><span class="sxs-lookup"><span data-stu-id="9542e-163">Open **./Pages/Index.cshtml** and add the following functions inside the `<script>` tag.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. <span data-ttu-id="9542e-164">Добавьте следующую функцию в `<script>` тег, чтобы отобразить успешный результат веб-API.</span><span class="sxs-lookup"><span data-stu-id="9542e-164">Add the following function inside the `<script>` tag to display a successful result from the Web API.</span></span>

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. <span data-ttu-id="9542e-165">Замените `successCallback` существующий следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="9542e-165">Replace the existing `successCallback` with the following code.</span></span>

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. <span data-ttu-id="9542e-166">Сохраните изменения и перезапустите приложение.</span><span class="sxs-lookup"><span data-stu-id="9542e-166">Save your changes and restart the app.</span></span> <span data-ttu-id="9542e-167">Обновите вкладку в Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="9542e-167">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="9542e-168">Вы должны получить всплывающее окно с просьбой о согласии на Graph microsoft.</span><span class="sxs-lookup"><span data-stu-id="9542e-168">You should get a pop-up window asking for consent to the Microsoft Graph permissions scopes.</span></span> <span data-ttu-id="9542e-169">После принятие вкладка должна `{ "status": "OK" }` отображаться .</span><span class="sxs-lookup"><span data-stu-id="9542e-169">After accepting, the tab should display `{ "status": "OK" }`.</span></span>

    > [!NOTE]
    > <span data-ttu-id="9542e-170">Если вкладка отображает, отключите блокаторы всплывающее окна в браузере и `"FailedToOpenWindow"` перезагрузите страницу.</span><span class="sxs-lookup"><span data-stu-id="9542e-170">If the tab displays `"FailedToOpenWindow"`, please disable pop-up blockers in your browser and reload the page.</span></span>

1. <span data-ttu-id="9542e-171">Просмотрите выход журнала.</span><span class="sxs-lookup"><span data-stu-id="9542e-171">Review the log output.</span></span> <span data-ttu-id="9542e-172">Вы должны увидеть `Access token for Graph` запись.</span><span class="sxs-lookup"><span data-stu-id="9542e-172">You should see the `Access token for Graph` entry.</span></span> <span data-ttu-id="9542e-173">Если вы разберите этот маркер, вы заметите, что он содержит области Microsoft Graph, настроенные в **appsettings.js.**</span><span class="sxs-lookup"><span data-stu-id="9542e-173">If you parse that token, you'll notice that it contains the Microsoft Graph scopes configured in **appsettings.json**.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="9542e-174">Хранение и обновление маркеров</span><span class="sxs-lookup"><span data-stu-id="9542e-174">Storing and refreshing tokens</span></span>

<span data-ttu-id="9542e-175">На этом этапе у приложения есть маркер доступа, который отправляется в `Authorization` заголовке вызовов API.</span><span class="sxs-lookup"><span data-stu-id="9542e-175">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="9542e-176">Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="9542e-176">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="9542e-177">Однако этот маркер недолговечен.</span><span class="sxs-lookup"><span data-stu-id="9542e-177">However, this token is short-lived.</span></span> <span data-ttu-id="9542e-178">Срок действия маркера истекает через час после его выпуска.</span><span class="sxs-lookup"><span data-stu-id="9542e-178">The token expires an hour after it is issued.</span></span> <span data-ttu-id="9542e-179">Вот здесь и пригодится маркер обновления.</span><span class="sxs-lookup"><span data-stu-id="9542e-179">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="9542e-180">Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входа в систему.</span><span class="sxs-lookup"><span data-stu-id="9542e-180">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="9542e-181">Поскольку приложение использует библиотеку Microsoft.Identity.Web, не нужно внедрять логику хранения маркеров или обновления.</span><span class="sxs-lookup"><span data-stu-id="9542e-181">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="9542e-182">Приложение использует кэш маркеров в памяти, который является достаточным для приложений, которым не нужно сохранять маркеры при перезапуске приложения.</span><span class="sxs-lookup"><span data-stu-id="9542e-182">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="9542e-183">Производственные приложения могут вместо этого использовать [параметры распределенного кэша](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) в библиотеке Microsoft.Identity.Web.</span><span class="sxs-lookup"><span data-stu-id="9542e-183">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="9542e-184">Метод `GetAccessTokenForUserAsync` обрабатывает срок действия маркера и обновляется для вас.</span><span class="sxs-lookup"><span data-stu-id="9542e-184">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="9542e-185">Сначала он проверяет кэш-маркер, и если срок его действия не истек, он возвращает его.</span><span class="sxs-lookup"><span data-stu-id="9542e-185">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="9542e-186">Если срок действия истек, он использует кэшный маркер обновления для получения нового.</span><span class="sxs-lookup"><span data-stu-id="9542e-186">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="9542e-187">**GraphServiceClient,** который контроллеры получают с помощью инъекции зависимостей, предварительно настроен с поставщиком проверки подлинности, который `GetAccessTokenForUserAsync` использует для вас.</span><span class="sxs-lookup"><span data-stu-id="9542e-187">The **GraphServiceClient** that controllers get via dependency injection is pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
