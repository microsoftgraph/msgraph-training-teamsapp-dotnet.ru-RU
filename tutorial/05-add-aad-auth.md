<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширит приложение от предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD. Это необходимо для получения маркера доступа OAuth, который нужен для вызова API Microsoft Graph. На этом шаге будет настроена [библиотека Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Чтобы не хранить код приложения и секрет в источнике, для хранения этих значений используется [диспетчер секрета .NET.](/aspnet/core/security/app-secrets) Секретный менеджер предназначен только для целей разработки, для хранения секретов в производственных приложениях должен использовать доверенный секретный менеджер.

1. Откройте **./appsettings.jsи** замените содержимое следующим.

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. Откройте свой CLI в каталоге, в котором расположен **GraphTutorial.csproj,** и запустите следующие команды, заменив свой ID приложения на портале Azure и секрет `YOUR_APP_ID` `YOUR_APP_SECRET` приложения.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Реализация входа в систему

Во-первых, в коде JavaScript приложения реализована одна входная подпись. Вы будете использовать Microsoft Teams [JavaScript SDK](/javascript/api/overview/msteams-client) для получения маркера доступа, который позволяет коду JavaScript, запущенным в клиенте Teams, выполнять вызовы AJAX в веб-API, которые будут реализованы позже.

1. Откройте **./Pages/Index.cshtml** и добавьте в тег следующий `<script>` код.

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

    Это вызывает бесшумную проверку подлинности в качестве пользователя, который подписан на `microsoftTeams.authentication.getAuthToken` Teams. Обычно не требуется никаких подсказок пользовательского интерфейса, если только пользователю не требуется согласие. Затем код отображает маркер на вкладке.

1. Сохраните изменения и запустите приложение, выполив следующую команду в CLI.

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > Если вы перезапустили ngrok и url-адрес ngrok изменился, перед тестированием обязательно обновите значение ngrok в следующем месте. 
    >
    > - URI перенаправления в регистрации приложения
    > - ID-URI приложения в регистрации приложения
    > - `contentUrl` в manifest.jsна
    > - `validDomains` в manifest.jsна
    > - `resource` в manifest.jsна

1. Создайте файл ZIP **сmanifest.js,** **color.png** и **outline.png.**

1. В Microsoft Teams выберите **Приложения** в левой панели, выберите Upload настраиваемого **приложения,** а затем выберите Upload для меня или **моей команды**.

    ![Снимок экрана Upload настраиваемой ссылки приложения в Microsoft Teams](images/upload-custom-app.png)

1. Просмотрите созданный ранее файл ZIP и выберите **Open.**

1. Просмотрите сведения о приложении и выберите **Добавить**.

1. Приложение открывается в Teams и отображает маркер доступа.

Если вы скопируете маркер, его можно вклеить [в jwt.ms.](https://jwt.ms) Убедитесь, что аудитория (утверждение) — это ваш ID приложения, и единственной областью (утверждением) является созданная область `aud` `scp` `access_as_user` API. Это означает, что этот маркер не предоставляет прямой доступ к Microsoft Graph! Вместо этого [веб-API,](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) который будет реализован в ближайшее время, должен будет обменять этот маркер с помощью потока от имени, чтобы получить маркер, который будет работать с вызовами microsoft Graph.

## <a name="configure-authentication-in-the-aspnet-core-app"></a>Настройка проверки подлинности в ASP.NET Core приложении

Начните с добавления служб платформы microsoft Identity в приложение.

1. Откройте **файл ./Startup.cs** и добавьте следующее утверждение `using` в верхнюю часть файла.

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. Добавьте следующую строку перед `app.UseAuthorization();` строкой в `Configure` функции.

    ```csharp
    app.UseAuthentication();
    ```

1. Добавьте следующую строку сразу `endpoints.MapRazorPages();` после строки в `Configure` функции.

    ```csharp
    endpoints.MapControllers();
    ```

1. Замените имеющуюся функцию `ConfigureServices` указанным ниже кодом.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    Этот код настраивает приложение для проверки подлинности вызовов веб-API на основе маркера носитель JWT в `Authorization` загонах. Он также добавляет службы приобретения маркеров, которые могут обмениваться этим маркером через поток от имени.

## <a name="create-the-web-api-controller"></a>Создание контроллера веб-API

1. Создание нового каталога в корне проекта с именем **Контроллеры**.

1. Создайте новый файл в **каталоге ./Controllers** с именем **CalendarController.cs** и добавьте следующий код.

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

    При этом реализуется веб-API (), который можно назвать `GET /calendar` с Teams вкладки. Пока он просто пытается обменять маркер носитель на маркер Graph маркера. При первом загрузке вкладки пользователь не сможет разрешить доступ приложения к microsoft Graph от своего имени.

1. Откройте **./Pages/Index.cshtml** и `successCallback` замените функцию на следующую.

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

    При этом будет вызываться веб-API и отображаться ответ.

1. Сохраните изменения и перезапустите приложение. Обновите вкладку в Microsoft Teams. Страница должна `consent_required` отображаться .

1. Просмотрите выход журнала в CLI. Обратите внимание на две вещи.

    - Запись типа `Authenticated user: MeganB@contoso.com` . Веб-API сдал проверку подлинности пользователя на основе маркера, отправленного с запросом API.
    - Запись типа `AADSTS65001: The user or administrator has not consented to use the application with ID...` . Это ожидается, так как пользователю еще не было предложено дать согласие на запрашиваемую область Graph Microsoft.

## <a name="implement-consent-prompt"></a>Реализация запроса на согласие

Поскольку веб-API не может подсказить пользователю, Teams вкладке потребуется реализовать подсказку. Это необходимо сделать только один раз для каждого пользователя. После согласия пользователя повторное согласие не требуется, если он явно не отзовет доступ к вашему приложению.

1. Создайте новый файл в **каталоге ./Pages** с именем **Authenticate.cshtml.cs** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. Создайте новый файл в **каталоге ./Pages** с именем **Authenticate.cshtml** и добавьте следующий код.

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. Создайте новый файл в **каталоге ./Pages** с именем **AuthComplete.cshtml** и добавьте следующий код.

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. Откройте **./Pages/Index.cshtml** и добавьте в тег следующие `<script>` функции.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. Добавьте следующую функцию в `<script>` тег, чтобы отобразить успешный результат веб-API.

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. Замените `successCallback` существующий следующим кодом.

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. Сохраните изменения и перезапустите приложение. Обновите вкладку в Microsoft Teams. Вы должны получить всплывающее окно с просьбой о согласии на Graph microsoft. После принятие вкладка должна `{ "status": "OK" }` отображаться .

    > [!NOTE]
    > Если вкладка отображает, отключите блокаторы всплывающее окна в браузере и `"FailedToOpenWindow"` перезагрузите страницу.

1. Просмотрите выход журнала. Вы должны увидеть `Access token for Graph` запись. Если вы разберите этот маркер, вы заметите, что он содержит области Microsoft Graph, настроенные в **appsettings.js.**

## <a name="storing-and-refreshing-tokens"></a>Хранение и обновление маркеров

На этом этапе у приложения есть маркер доступа, который отправляется в `Authorization` заголовке вызовов API. Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.

Однако этот маркер недолговечен. Срок действия маркера истекает через час после его выпуска. Вот здесь и пригодится маркер обновления. Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входа в систему.

Поскольку приложение использует библиотеку Microsoft.Identity.Web, не нужно внедрять логику хранения маркеров или обновления.

Приложение использует кэш маркеров в памяти, который является достаточным для приложений, которым не нужно сохранять маркеры при перезапуске приложения. Производственные приложения могут вместо этого использовать [параметры распределенного кэша](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) в библиотеке Microsoft.Identity.Web.

Метод `GetAccessTokenForUserAsync` обрабатывает срок действия маркера и обновляется для вас. Сначала он проверяет кэш-маркер, и если срок его действия не истек, он возвращает его. Если срок действия истек, он использует кэшный маркер обновления для получения нового.

**GraphServiceClient,** который контроллеры получают с помощью инъекции зависимостей, предварительно настроен с поставщиком проверки подлинности, который `GetAccessTokenForUserAsync` использует для вас.
