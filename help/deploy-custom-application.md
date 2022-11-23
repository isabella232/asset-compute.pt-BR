---
title: Implantar [!DNL Asset Compute Service] aplicativo personalizado
description: Implantar [!DNL Asset Compute Service] aplicativo personalizado.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 129651ba432b75703bc27baa7081da60302f828d
workflow-type: tm+mt
source-wordcount: '184'
ht-degree: 3%

---

# Implantar um aplicativo personalizado {#deploy-custom-application}

Para implantar seu aplicativo, use [implantação do aplicativo aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. No terminal, o comando exibe um URL para acessar o aplicativo personalizado. O URL é do formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obter o mesmo URL sem reimplantar o aplicativo, use [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Use o URL em um [Perfil de processamento em [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR) para integrar seu aplicativo ao [!DNL Experience Manager] como [!DNL Cloud Service].

Certifique-se de que o projeto e a área de trabalho do App Builder correspondam ao [!DNL Experience Manager] como [!DNL Cloud Service] ambiente em que deseja usar a ação. Ele tem ambientes diferentes para desenvolvimento, armazenamento temporário e produção. Você pode verificar o ambiente verificando `AIO_runtime_*` credenciais definidas dentro do arquivo ENV na raiz do aplicativo Firefly. Por exemplo, para implantar em um `Stage` espaço de trabalho, a variável `AIO_runtime_namespace` é do formato `xxxxxx_xxxxxxxxx_stage`. Para integrar com [!DNL Experience Manager] como [!DNL Cloud Service] Ambiente de produção, use URLs de aplicativo do Firefly `Production` espaço de trabalho.

>[!CAUTION]
>
>Não use um espaço de trabalho pessoal em [!DNL Experience Manager] ambientes .

>[!MORELIKETHIS]
>
>* [Entender e gerenciar ambientes no [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

