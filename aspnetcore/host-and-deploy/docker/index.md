---
title: Размещение ASP.NET Core в контейнерах Docker
author: rick-anderson
description: Ссылки на ресурсы для изучения способов размещения приложений ASP.NET Core в контейнерах Docker.
ms.author: riande
ms.custom: mvc
ms.date: 01/08/2018
uid: host-and-deploy/docker/index
ms.openlocfilehash: 9189e1fbb21abcc8c8bdea947e672ee53b59bc4f
ms.sourcegitcommit: dd9c73db7853d87b566eef136d2162f648a43b85
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "65085476"
---
# <a name="host-aspnet-core-in-docker-containers"></a>Размещение ASP.NET Core в контейнерах Docker

В следующих статьях содержатся сведения о размещении приложений ASP.NET Core в Docker.

[Общие сведения о контейнерах и Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index)  
Узнайте о том, что контейнеризация — это подход к разработке программного обеспечения, при котором приложение или служба, их зависимости и конфигурация упаковываются вместе в образ контейнера. Образ можно протестировать и затем развернуть на узле.

[Что такое Docker?](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-defined)  
Узнайте, о том, что Docker — это проект с открытым исходным кодом для автоматизации развертывания приложений в виде переносимых автономных контейнеров, выполняемых в облаке или локальной среде.

[Терминология Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-terminology)  
Изучите термины и определения для технологии Docker.

[Контейнеры, образы и реестры Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-containers-images-registries)  
Узнайте о хранении образов контейнеров Docker в реестре образов для согласованного развертывания в средах.

<xref:host-and-deploy/docker/building-net-docker-images> Ознакомьтесь со сведениями о том, как создать приложение ASP.NET Core и преобразовать его для Docker. Изучите образы Docker, поддерживаемые корпорацией Майкрософт, и ознакомьтесь с вариантами использования.

[Инструменты Visual Studio для Docker](xref:host-and-deploy/docker/visual-studio-tools-for-docker)  
Узнайте, как Visual Studio 2017 поддерживает создание, отладку и запуск приложений ASP.NET Core, предназначенных для .NET Framework или .NET Core, в Docker для Windows. Поддерживаются контейнеры Windows и Linux.

[Публикация в образ Docker](/azure/vs-azure-tools-docker-hosting-web-apps-in-docker)  
Узнайте, как использовать расширение Visual Studio Tools for Docker для развертывания приложения ASP.NET Core на узле Docker в Azure с помощью PowerShell.

[Настройка ASP.NET Core для работы с прокси-серверами и подсистемами балансировки нагрузки](xref:host-and-deploy/proxy-load-balancer)  
Для приложений, размещенных за прокси-серверами и подсистемами балансировки нагрузки, может потребоваться дополнительная настройка. При передаче запросов через прокси-сервер сведения об исходном запросе, например схема и IP-адрес клиента, часто бывают скрыты. Иногда необходимо вручную переслать некоторые сведения о запросе в приложение.
