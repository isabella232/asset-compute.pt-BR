---
title: Defina o ambiente de desenvolvimento necessário para [!DNL Asset Compute Service]
description: Configuração do ambiente do desenvolvedor para [!DNL Asset Compute Service] para começar a criar e testar o código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 3%

---

# Configurar um ambiente de desenvolvedor {#create-dev-environment}

Para criar uma configuração que permite desenvolver para [!DNL Asset Compute Service], siga estes requisitos e instruções.

1. [Adquirir acesso e credenciais](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) para [!DNL Adobe Developer App Builder].

1. [Configurar o ambiente local](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) e as ferramentas necessárias.

1. Outras ferramentas que ajudam você a começar a desenvolver-se sem problemas são:

   * [Git](https://git-scm.com/)
   * [Desktop Docker](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, versões ímpares não são recomendadas) e [NPM](https://www.npmjs.com). O usuário do HomeBrew OSX pode fazer `brew install node` para instalar ambos. Caso contrário, baixe-o do [Página de download do NodeJS](https://nodejs.org/en/)
   * Um IDE que seja bom para o NodeJS, recomendamos [Código Visual Studio (Código VS)](https://code.visualstudio.com) já que é o IDE compatível para o depurador. Você pode usar qualquer outro IDE como editor de código, mas o uso avançado (por exemplo, depurador) ainda não é suportado
   * Instale o mais recente[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Certifique-se de atender ao [pré-requisitos](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configurar um projeto do App Builder {#create-App-Builder-project}

1. Verifique se o administrador do sistema ou a função de desenvolvedor no [!DNL Experience Cloud] organização. Isso é configurado por um administrador do sistema no [Admin Console](https://adminconsole.adobe.com/overview).

1. Faça logon no [Console do Adobe Developer](https://console.adobe.io/). Certifique-se de fazer parte do mesmo [!DNL Experience Cloud] organização como [!DNL Experience Manager] como [!DNL Cloud Service] integração. Para obter mais informações sobre o Console do Adobe Developer, consulte [Documentação do console](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Criar um projeto do App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Clique em **[!UICONTROL Criar novo projeto]** > **[!UICONTROL Projeto a partir do modelo]**. Selecione App Builder. Ele cria um novo Projeto do App Builder com dois espaços de trabalho: `Production` e `Stage`. Adicionar espaços de trabalho adicionais, por exemplo `Development`, conforme necessário.

1. No Projeto do App Builder, selecione um espaço de trabalho e assine os serviços necessários ao Asset compute. Clique em **Adicionar ao projeto** > **API** e adicionar `Asset Compute`, `IO Events`e `IO Events Management` serviços. Ao adicionar a primeira API, ela solicita a criação de uma chave privada. Salve essas informações no computador, pois é necessário ter essa chave para testar seu aplicativo personalizado com a ferramenta do desenvolvedor.

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
