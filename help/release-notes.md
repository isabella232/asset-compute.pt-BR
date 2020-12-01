---
title: Notas de versão de [!DNL Asset Compute Service].
description: Novos recursos, melhorias e problemas conhecidos em [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: c57867cd896e4ccb9402e6eeb0eea133faaa0e5d
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 1%

---


# Notas de versão de [!DNL Asset Compute Service] {#release-notes}

A versão mais recente do [!DNL Asset Compute Service] foi lançada em 30 de julho de 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Novidades {#what-is-new}

Esta é a primeira versão do [!DNL Asset Compute Service]. É um serviço escalonável e extensível de [!DNL Adobe Experience Cloud] para processar ativos digitais. Ele pode transformar imagens, vídeos, documentos e outros formatos de arquivo em diferentes representações, incluindo miniaturas, texto e metadados extraídos e arquivos.

Atualmente, [!DNL Asset Compute Service] só pode ser usado em [!DNL Experience Manager] como [!DNL Cloud Service].

## Limitações e problemas conhecidos {#known-limitations}

Para testar seu aplicativo personalizado com a [ferramenta do desenvolvedor](https://github.com/adobe/asset-compute-devtool), você precisa acessar um [container do armazenamento na nuvem](https://github.com/adobe/asset-compute-devtool#prerequisites).

* O acesso ao armazenamento em nuvem (diferente do armazenamento em blob [!DNL Experience Manager]) é necessário apenas para a ferramenta do desenvolvedor. Você ainda pode criar, testar e implantar aplicativos personalizados sem a ferramenta para desenvolvedores.
* Pode ser um container compartilhado usado por vários desenvolvedores em diferentes projetos.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] a extensibilidade é desenvolvida sob um modelo de desenvolvimento aberto em  [github.com/](https://github.com/adobe) adobeque acolhe as contribuições dos desenvolvedores de extensões. Todos os componentes relevantes para desenvolver, criar, testar e implantar aplicativos personalizados são de código aberto. Consulte [como e onde contribuir para o Serviço de computação](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
