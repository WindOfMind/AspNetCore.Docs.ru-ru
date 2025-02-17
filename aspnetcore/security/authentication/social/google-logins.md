---
title: Настройка внешней учетной записи Google в ASP.NET Core
author: rick-anderson
description: В этом учебнике показано интеграцию Google учетной записи пользователя и проверки подлинности в существующее приложение ASP.NET Core.
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 06/19/2019
uid: security/authentication/google-logins
ms.openlocfilehash: b0edac411e73cd2eec7c4e212b99971577f59cfb
ms.sourcegitcommit: 06a455d63ff7d6b571ca832e8117f4ac9d646baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/21/2019
ms.locfileid: "67316453"
---
# <a name="google-external-login-setup-in-aspnet-core"></a>Настройка внешней учетной записи Google в ASP.NET Core

Авторы: [Валерий Новицкий](https://github.com/01binary) (Valeriy Novytskyy) и [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

[Начиная с 7 марта 2019 г. была завершена традиционные Google + API](https://developers.google.com/+/api-shutdown). Google + вход и разработчикам необходимо переместить в новый входа Google в системе. Пакеты ASP.NET Core 2.1 и 2.2 для проверки подлинности Google обновили в соответствии с изменениями. Дополнительные сведения и временные способы устранения рисков для ASP.NET Core, см. в разделе [проблема GitHub](https://github.com/aspnet/AspNetCore/issues/6486). Этот учебник был обновлен с помощью нового процесса установки.

Этом руководстве показано, как разрешить пользователям выполнить вход с использованием своей учетной записью Google, с помощью ASP.NET Core 2.2 проект, созданный на [предыдущую страницу](xref:security/authentication/social/index).

## <a name="create-a-google-api-console-project-and-client-id"></a>Создать идентификатор проекта и клиент консоли Google API

* Перейдите к [интеграция Google Sign-In в веб-приложения](https://developers.google.com/identity/sign-in/web/devconsole-project) и выберите **НАСТРОИТЬ ПРОЕКТ**.
* В **настроить клиент OAuth** диалоговом окне выберите **веб-сервере**.
* В **авторизованные URI перенаправления** текстовом поле, значение URI перенаправления. Например: `https://localhost:5001/signin-google`
* Сохранить **идентификатор клиента** и **секрет клиента**.
* При развертывании на сайте, зарегистрировать новый общедоступный URL-адрес из **консоли Google**.

## <a name="store-google-clientid-and-clientsecret"></a>Store Google ClientID и ClientSecret

Конфиденциальные параметры, такие как Google Store `Client ID` и `Client Secret` с [Secret Manager](xref:security/app-secrets). В целях этого учебника назовите токены `Authentication:Google:ClientId` и `Authentication:Google:ClientSecret`:

```console
dotnet user-secrets set "Authentication:Google:ClientId" "X.apps.googleusercontent.com"
dotnet user-secrets set "Authentication:Google:ClientSecret" "<client secret>"
```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

Вы можете управлять ваши учетные данные API и использования в [консоль API](https://console.developers.google.com/apis/dashboard).

## <a name="configure-google-authentication"></a>Настройка проверки подлинности Google

Добавить службу Google `Startup.ConfigureServices`:

[!code-csharp[](~/security/authentication/social/social-code/StartupGoogle.cs?name=snippet_ConfigureServices&highlight=10-18)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a>Войдите с помощью Google

* Запустите приложение и нажмите кнопку **вход**. Появится возможность войти с помощью Google.
* Нажмите кнопку **Google** кнопку, которая перенаправляет Google для проверки подлинности.
* После ввода учетных данных Google, вы будете перенаправлены к веб-сайта.

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

См. в разделе <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> Справочник по API, Дополнительные сведения о параметрах конфигурации, поддерживается проверка подлинности Google. Это может использоваться для запроса различные сведения о пользователе.

## <a name="change-the-default-callback-uri"></a>Изменить URI обратного вызова по умолчанию

Сегмент URI `/signin-google` задан в качестве обратного вызова по умолчанию поставщик проверки подлинности Google. URI обратного вызова по умолчанию можно изменить при настройке по промежуточного слоя проверки подлинности Google с помощью наследуемого [RemoteAuthenticationOptions.CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath) свойство [GoogleOptions](/dotnet/api/microsoft.aspnetcore.authentication.google.googleoptions) класса.

## <a name="troubleshooting"></a>Устранение неполадок

* Если вход не работает, а не получают все ошибки, переключитесь в режим разработки для упрощения процесса отладки проблемы.
* Если удостоверение не настроена, вызвав `services.AddIdentity` в `ConfigureServices`, попытка проверки подлинности приводит к *ArgumentException: Необходимо указать параметр «SignInScheme»* . Шаблон проекта, используемый в этом руководстве гарантирует, что это будет сделано.
* Если база данных сайта не был создан путем применения первоначальной миграции, вы получаете *сбой операции из базы данных при обработке запроса* ошибки. Выберите **применить миграции** для создания базы данных и обновите страницу, чтобы продолжить выполнение после ошибки.

## <a name="next-steps"></a>Следующие шаги

* В этой статье объясняется, как можно выполнить проверку подлинности с помощью Google. Можно выполнить аналогичный подход для проверки подлинности с помощью других поставщиков, перечисленных на [предыдущую страницу](xref:security/authentication/social/index).
* После того, как опубликовать приложение в Azure, сбросить `ClientSecret` в консоли Google API.
* Задайте `Authentication:Google:ClientId` и `Authentication:Google:ClientSecret` как параметры приложения на портале Azure. Система конфигурации предназначена для чтения разделов из переменных среды.
