---
title: Defina o ambiente de desenvolvimento necessário para [!DNL Asset Compute Service]
description: Configuração do ambiente do desenvolvedor para [!DNL Asset Compute Service] começar a criar e testar o código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: tm+mt
source-wordcount: '370'
ht-degree: 3%

---

# Configurar um ambiente de desenvolvedor {#create-dev-environment}

Para criar uma configuração que permita desenvolver para [!DNL Asset Compute Service], siga estes requisitos e instruções.

1. [Adquira acesso e ](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) credenciais para  [!DNL Project Firefly].

1. [Configure o ](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) ambiente local e as ferramentas necessárias.

1. Outras ferramentas que ajudam você a começar a desenvolver-se sem problemas são:

   * [Git](https://git-scm.com/).
   * [Desktop Docker](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org)  (v10 para v12 LTS, versões ímpares não são recomendadas) e  [NPM](https://www.npmjs.com). O usuário do OSX HomeBrew pode fazer `brew install node` para instalar ambos. Caso contrário, baixe-o da [página de download do NodeJS](https://nodejs.org/en/).
   * Um IDE que seja bom para o NodeJS, recomendamos [Visual Studio Code (VS Code)](https://code.visualstudio.com), pois ele é o IDE compatível para o depurador. Você pode usar qualquer outro IDE como editor de código, mas o uso avançado (por exemplo, depurador) ainda não é suportado.
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`) - instale usando  `npm install -g @adobe/aio-cli@7.1.0`.

1. Certifique-se de atender aos [pré-requisitos](/help/understand-extensibility.md#prerequisites-and-provisioning).

>[!NOTE]
>
>Por enquanto, use [!DNL Adobe I/O] CLI v7.1.0 de e não use [!DNL Adobe I/O] CLI v8.

## Configurar um projeto do Firefly {#create-firefly-project}

1. Assegure o administrador do sistema ou a função de desenvolvedor na organização [!DNL Experience Cloud]. Isso é configurado por um administrador de sistema no [Admin Console](https://adminconsole.adobe.com/overview).

1. Faça logon no [Console do desenvolvedor do Adobe](https://console.adobe.io/). Certifique-se de fazer parte da mesma organização [!DNL Experience Cloud] que a [!DNL Experience Manager] como uma integração [!DNL Cloud Service]. Para obter mais informações sobre o Console do Desenvolvedor do Adobe, consulte a [Documentação do Console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Criar um projeto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md) do Firefly. Clique em **[!UICONTROL Criar novo projeto]** > **[!UICONTROL Projeto a partir do modelo]**. Selecione Firefly. Ele cria um novo Projeto Firefly com dois espaços de trabalho: `Production` e `Stage`. Adicione espaços de trabalho adicionais, por exemplo `Development`, conforme necessário.

1. No Projeto Firefly, selecione um espaço de trabalho e assine os serviços necessários para o Asset compute. Clique em **Adicionar ao Projeto** > **API** e adicione serviços `Asset Compute`, `IO Events` e `IO Events Management`. Ao adicionar a primeira API, ela solicita a criação de uma chave privada. Salve essas informações no computador, pois é necessário ter essa chave para testar seu aplicativo personalizado com a ferramenta do desenvolvedor.

## Próxima etapa {#next-step}

Agora que seu ambiente está configurado, você está pronto para [criar um aplicativo personalizado](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
