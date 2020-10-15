---
title: Defina o ambiente de desenvolvimento necessário para [!DNL Asset Compute Service].
description: Configuração do ambiente do desenvolvedor [!DNL Asset Compute Service] para criar e testar o código personalizado do start.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 3%

---


# Configurar um ambiente para desenvolvedores {#create-dev-environment}

Para criar uma configuração que permita desenvolver para [!DNL Asset Compute Service], siga estes requisitos e instruções.

1. [Adquira acesso e credenciais](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) para o Project Firefly.

1. [Configure o ambiente](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) local e as ferramentas necessárias.

1. Algumas outras ferramentas que ajudam você a começar a se desenvolver sem problemas são:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 a v12 LTS, versões ímpares não são recomendadas) e [NPM](https://www.npmjs.com). O usuário do OSX HomeBrew pode fazer `brew install node` para instalar ambos. Caso contrário, baixe-o da página [de download do](https://nodejs.org/en/)NodeJS.
   * Um IDE que é bom para NodeJS, recomendamos o Código [Visual Studio (Código VS)](https://code.visualstudio.com) , pois é o IDE compatível para o depurador. Você pode usar qualquer outro IDE como editor de código, mas a utilização avançada (por exemplo, depurador) ainda não é suportada.
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`) - instale usando `npm install -g @adobe/aio-cli`.

1. Certifique-se de atender aos [pré-requisitos](/help/understand-extensibility.md#prerequisites-and-provisioning).

## Configurar um projeto do Firefly {#create-firefly-project}

1. Receba acesso de Administrador do sistema ou Função do desenvolvedor na Organização da experiência. Isso pode ser definido por um administrador do sistema no [Admin Console](https://adminconsole.adobe.com/overview).

1. Faça logon no [Adobe Developer Console](https://console.adobe.io/). Certifique-se de fazer parte da mesma Organização Adobe Experience Cloud que o AEM como uma integração Cloud Service. Para obter mais informações sobre o Console do desenvolvedor do Adobe, consulte a documentação [do](https://www.adobe.io/apis/experienceplatform/console/docs.html)console.

1. [Criar um projeto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)Firefly. Clique em **[!UICONTROL Criar novo projeto]** > **[!UICONTROL Projeto do modelo]**. Selecione Firefly. Ele cria um novo projeto Firefly com dois espaços de trabalho: `Production` e `Stage`. Adicione outros espaços de trabalho, por exemplo `Development`, conforme necessário.

1. No Firefly Project, selecione um espaço de trabalho e assine os serviços necessários para o Asset Compute. Clique em **Adicionar ao projeto** > **API** e adicione `Asset Compute`, `IO Events`e `IO Events Management` serviços. Ao adicionar a primeira API, ele solicita a criação de uma chave privada. Salve essas informações em seu computador, pois é necessário usar essa chave para testar seu aplicativo personalizado com a ferramenta para desenvolvedores.

## Next step {#next-step}

Agora que seu ambiente está configurado, você está pronto para [criar um aplicativo](develop-custom-application.md)personalizado.

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
