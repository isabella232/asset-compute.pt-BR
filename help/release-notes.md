---
title: Notas de versão [!DNL Asset Compute Service].
description: Novos recursos, melhorias e problemas conhecidos no [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 1%

---


# Notas de versão de [!DNL Asset Compute Service] {#release-notes}

A versão mais recente do é lançada [!DNL Asset Compute Service] em 30 de julho de 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Novidades {#what-is-new}

Esta é a primeira versão do [!DNL Asset Compute Service]. É um serviço escalonável e extensível de processamento de ativos digitais. [!DNL Adobe Experience Cloud] Ele pode transformar imagens, vídeos, documentos e outros formatos de arquivo em diferentes representações, incluindo miniaturas, texto e metadados extraídos e arquivos.

Atualmente, o [!DNL Asset Compute Service] pode ser usado apenas [!DNL Experience Manager] como Cloud Service.

## Limitações e problemas conhecidos {#known-limitations}

Para testar seu aplicativo personalizado com a ferramenta [](https://github.com/adobe/asset-compute-devtool)desenvolvedor, é necessário acessar um container [de armazenamento](https://github.com/adobe/asset-compute-devtool#prerequisites)em nuvem.

* O acesso ao armazenamento de nuvem (diferente do armazenamento de [!DNL Experience Manager] blob) é necessário apenas para a ferramenta do desenvolvedor. Você ainda pode criar, testar e implantar aplicativos personalizados sem a ferramenta para desenvolvedores.
* Pode ser um container compartilhado usado por vários desenvolvedores em diferentes projetos.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] a extensibilidade é desenvolvida sob um modelo de desenvolvimento aberto em [github.com/adobe](https://github.com/adobe) , que acolhe com agrado as contribuições dos desenvolvedores de extensões. Todos os componentes relevantes para desenvolver, criar, testar e implantar aplicativos personalizados são de código aberto. Veja [como e onde contribuir para o Serviço](contribute-to-compute-service.md)de computação.

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
