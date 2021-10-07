---
title: Introdução ao  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] é um serviço de processamento de ativos nativo em nuvem que reduz a complexidade e melhora a escalabilidade.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: a2460a0719f8c585ed72e44c904aa0df33301032
workflow-type: tm+mt
source-wordcount: '307'
ht-degree: 0%

---

# Visão geral de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] O é um serviço dimensionável e extensível do  [!DNL Adobe Experience Cloud] para processar ativos digitais. Ele pode transformar imagens, vídeos, documentos e outros formatos de arquivo em diferentes representações, incluindo miniaturas, texto e metadados extraídos e arquivos.

Os desenvolvedores podem plugin de aplicativos de ativos personalizados (também chamados de funcionários personalizados) para tratar de casos de uso personalizados. O serviço funciona no tempo de execução [!DNL Adobe I/O]. Ele pode ser estendido por meio de [!DNL Project Firefly] aplicativos sem cabeçalho gravados em Node.js. Eles podem realizar operações personalizadas, como chamar APIs externas para executar operações de imagem ou aproveitar o suporte [!DNL Adobe Sensei].

[!DNL Project Firefly] O é uma estrutura para criar e implantar aplicativos Web personalizados em  [!DNL Adobe I/O] tempo de execução para estender as soluções da Adobe Experience Cloud. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (kit de ferramentas da interface do usuário do Adobe), criar microsserviços, criar eventos personalizados e orquestrar APIs. Consulte a [documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Atualmente, o [!DNL Asset Compute Service] pode ser usado somente via [!DNL Experience Manager] como um [!DNL Cloud Service]. Os administradores criam perfis de processamento que podem chamar o [!DNL Asset Compute Service] para transmitir ativos para processamento. Consulte [usar microsserviços de ativos e perfis de processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Casos de uso suportados de [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] apoia alguns casos de uso comercial comuns, como o processamento básico de imagens; Conversões específicas da aplicação Adobe; e criação de aplicativos personalizados que orquestram requisitos comerciais complexos.

Você pode usar o [!DNL Asset Compute] serviço da Web para gerar miniaturas para diferentes tipos de arquivos, renderizações de imagem de alta qualidade para os [formatos de arquivo compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Consulte [casos de uso suportados por configuração personalizada](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>O serviço não fornece armazenamento de ativos. Os usuários o fornecem e fornecem referências para locais de arquivo de origem e renderização no armazenamento na nuvem.

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
>* [Visão geral do processamento de ativos com microsserviços de ativos  [!DNL Adobe Experience Manager] em a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Formatos de arquivo compatíveis com o processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
