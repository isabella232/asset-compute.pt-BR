---
title: Implantar aplicativo [!DNL Asset Compute Service] personalizado.
description: Implantar aplicativo [!DNL Asset Compute Service] personalizado.
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '197'
ht-degree: 0%

---


# Implantar um aplicativo personalizado {#deploy-custom-application}

Para implantar seu aplicativo, use o comando [de implantação](https://github.com/adobe/aio-cli#aio-appdeploy) do aplicativo rádio. No terminal, o comando exibe um URL para acessar o aplicativo personalizado. O URL está no formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obter o mesmo URL sem reimplantar o aplicativo, use [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) command.

Use o URL em um Perfil de [processamento no Experience Manager como Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) para integrar seu aplicativo [!DNL Experience Manager] como Cloud Service.

Certifique-se de que o projeto e o espaço de trabalho do Firefly correspondam ao [!DNL Experience Manager] como um ambiente Cloud Service onde você deseja usar a ação. Tem diferentes ambientes para desenvolvimento, armazenamento temporário e produção. Você pode verificar o ambiente verificando `AIO_runtime_*` as credenciais definidas dentro do arquivo ENV na raiz do aplicativo Firefly. Por exemplo, para implantar em um `Stage` espaço de trabalho, o formato `AIO_runtime_namespace` é `xxxxxx_xxxxxxxxx_stage`. Para integrar com [!DNL Experience Manager] um ambiente de produção de Cloud Service, use URLs de aplicativo de sua `Production` área de trabalho do Firefly.

>[!CAUTION]
>
>Não use um espaço de trabalho pessoal em [!DNL Experience Manager] ambientes críticos.

>[!MORELIKETHIS]
>
>* [Entenda e gerencie ambientes no Experience Manager como um Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

