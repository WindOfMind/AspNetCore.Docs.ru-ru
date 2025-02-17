---
title: Общие сведения о безопасности ASP.NET Core
author: tdykstra
description: Основные сведения о проверки подлинности, авторизации и безопасности в ASP.NET Core.
ms.author: tdykstra
ms.custom: mvc
ms.date: 10/24/2018
uid: security/index
ms.openlocfilehash: 933501411169d89c4b24edda743c47591aa7a87a
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/27/2019
ms.locfileid: "64881889"
---
# <a name="overview-of-aspnet-core-security"></a>Общие сведения о безопасности ASP.NET Core

ASP.NET Core позволяет разработчикам легко настраивать параметры безопасности для приложений и управлять ими. ASP.NET Core содержит функции для управления проверкой подлинности, авторизацией, защитой данных, применением HTTPS, секретами приложений, защитой от подделки запросов, а также для управления CORS. Эти функции обеспечения безопасности позволяют создавать надежные и защищенные приложения ASP.NET Core.

## <a name="aspnet-core-security-features"></a>Функции безопасности в ASP.NET Core

ASP.NET Core предоставляет множество средств и библиотек для защиты приложений, включая встроенные поставщики удостоверений, однако помимо них можно использовать сторонние службы удостоверений, такие как Facebook, LinkedIn и Twitter. В ASP.NET Core можно легко управлять секретами приложений, которые позволяют хранить и использовать конфиденциальные данные, не предоставляя их в коде.

## <a name="authentication-vs-authorization"></a>Проверка подлинности и Авторизация

Проверка подлинности — это процесс, когда пользователь вводит учетные данные, которые затем сравниваются с данными, хранящимися в операционной системе, базе данных, приложении или ресурсе. Если они совпадают, пользователи успешно проходят аутентификацию и во время авторизации могут выполнять разрешенные действия. Авторизация представляет собой процесс, определяющий, какие действия может выполнять пользователь.

Если взглянуть на проверку подлинности с другой стороны, ее можно считать способом входа в определенную область, например на сервер, в базу данных, приложение или ресурс, тогда как авторизация определяет, какие действия с какими объектами может выполнять пользователь в этой области (на сервере, в базе данных или приложении).

## <a name="common-vulnerabilities-in-software"></a>Распространенные уязвимости в программном обеспечении

ASP.NET Core и EF содержат средства, помогающие защитить приложения и предотвратить возникновение нарушений безопасности. Далее приводится список ссылок на документацию с описанием методов, позволяющих устранять наиболее распространенные уязвимости в веб-приложениях:

* [Атаки с использованием межузловых сценариев](xref:security/cross-site-scripting)
* [Атаки путем внедрения кода SQL](/ef/core/querying/raw-sql)
* [Подделки межсайтовых запросов (CSRF)](xref:security/anti-request-forgery)
* [Атаки с открытой переадресацией](xref:security/preventing-open-redirects)

Существует еще целый ряд уязвимостей, о которых следует знать. Дополнительные сведения см. в других статьях раздела **Безопасность и удостоверения**.
