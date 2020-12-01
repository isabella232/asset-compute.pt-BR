---
title: Implantar  [!DNL Asset Compute Service] aplicativo personalizado.
description: Implantar  [!DNL Asset Compute Service] aplicativo personalizado.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# Implantar um aplicativo personalizado {#deploy-custom-application}

Para implantar seu aplicativo, use o comando [aio app deployar](https://github.com/adobe/aio-cli#aio-appdeploy). No terminal, o comando exibe um URL para acessar o aplicativo personalizado. O URL tem o formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obter o mesmo URL sem reimplantar o aplicativo, use o comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action).

Use o URL em um [Perfil de processamento no Experience Manager como um Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) para integrar seu aplicativo com [!DNL Experience Manager] como um Cloud Service.

Certifique-se de que o projeto e o espaço de trabalho do Firefly correspondam ao [!DNL Experience Manager] como um ambiente Cloud Service onde você deseja usar a ação. Tem diferentes ambientes para desenvolvimento, armazenamento temporário e produção. Você pode verificar o ambiente verificando as credenciais `AIO_runtime_*` definidas dentro do arquivo ENV na raiz do aplicativo Firefly. Por exemplo, para implantar em um espaço de trabalho `Stage`, `AIO_runtime_namespace` é do formato `xxxxxx_xxxxxxxxx_stage`. Para integrar com [!DNL Experience Manager] como um ambiente de produção de Cloud Service, use URLs de aplicativos de sua área de trabalho Firefly `Production`.

>[!CAUTION]
>
>Não use um espaço de trabalho pessoal em ambientes [!DNL Experience Manager] críticos.

>[!MORELIKETHIS]
>
>* [Entenda e gerencie ambientes no Experience Manager como um Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

