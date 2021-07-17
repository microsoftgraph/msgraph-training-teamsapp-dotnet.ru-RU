<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается о создании приложения Microsoft Teams с помощью ASP.NET Core и API microsoft Graph для получения сведений о календаре для пользователя.

> [!TIP]
> Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [GitHub репозиторий](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet). Инструкции по настройке  приложения с помощью ID-приложения и секрета см. в файле README в демо-папке.

## <a name="prerequisites"></a>Предварительные требования

Перед началом этого учебного пособия на компьютере разработки должно быть установлено следующее.

- [.NET SDK](https://dotnet.microsoft.com/download).
- [ngrok](https://ngrok.com/)

Вы также должны иметь учетную запись Microsoft work или school в клиенте Microsoft 365, которая включила настраиваемую Teams [загрузку приложения.](/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant#enable-custom-teams-apps-and-turn-on-custom-app-uploading) Если у вас нет учетной записи Microsoft work или school, или ваша организация не [](https://developer.microsoft.com/office/dev-program) включила настраиваемую Teams загрузку приложений, вы можете зарегистрироваться в программе разработчика Microsoft 365, чтобы получить бесплатную подписку Office 365 разработчика.

> [!NOTE]
> Этот учебник был написан с помощью версии .NET SDK версии 5.0.302. Действия в этом руководстве могут работать с другими версиями, но они не были проверены.

## <a name="feedback"></a>Отзывы

В репозитории [](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet)GitHub.
