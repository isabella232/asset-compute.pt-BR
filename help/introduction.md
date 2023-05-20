---
title: Introdução ao [!DNL Asset Compute Service]
description: "[!DNL Asset Compute Service] O é um serviço de processamento de ativos em nuvem que reduz a complexidade e melhora a escalabilidade."
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# Visão geral do [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] O é um serviço escalonável e extensível de [!DNL Adobe Experience Cloud] para processar ativos digitais. Ele pode transformar imagens, vídeos, documentos e outros formatos de arquivo em diferentes representações, incluindo miniaturas, texto e metadados extraídos e arquivos.

Os desenvolvedores podem usar aplicativos de ativos personalizados como plug-in (também chamados de trabalhadores personalizados) para tratar de casos de uso personalizados. O serviço funciona no [!DNL Adobe I/O] tempo de execução. É extensível por meio de [!DNL Adobe Developer App Builder] aplicativos headless escritos em Node.js. Eles podem realizar operações personalizadas, como chamar APIs externas para executar operações de imagem ou aproveitar [!DNL Adobe Sensei] suporte.

[!DNL Adobe Developer App Builder] O é uma estrutura para criar e implantar aplicativos Web personalizados no [!DNL Adobe I/O] tempo de execução para estender as soluções da Adobe Experience Cloud. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (Kit de ferramentas da interface do Adobe), crie microsserviços, eventos personalizados e organize APIs. Consulte [documentação do Construtor de aplicativos Adobe Developer](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Atualmente, a [!DNL Asset Compute Service] pode ser usado somente via [!DNL Experience Manager] as a [!DNL Cloud Service]. Os administradores criam perfis de processamento que podem chamar o [!DNL Asset Compute Service] para transferir ativos para processamento. Consulte [usar microsserviços de ativos e perfis de processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR).

## Casos de uso compatíveis do [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] O oferece suporte a alguns casos de uso comerciais comuns, como processamento básico de imagens, conversões específicas de aplicativos Adobe e criação de aplicativos personalizados que organizam requisitos comerciais complexos.

Você pode usar [!DNL Asset Compute] serviço da web para gerar miniaturas para diferentes tipos de arquivos, renderizações de imagem de alta qualidade para o [formatos de arquivo compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Consulte [casos de uso compatíveis por meio da configuração personalizada](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR).

>[!NOTE]
>
>O serviço não fornece armazenamento de ativos. Os usuários o fornecem e fornecem referências a locais de arquivos de origem e representação no armazenamento na nuvem.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Visão geral do processamento de ativos com os microsserviços de ativos no [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=pt-BR).
>* [Documentação do Construtor de aplicativos Adobe Developer](https://developer.adobe.com/app-builder/docs/overview).
>* [Formatos de arquivo com suporte para processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
