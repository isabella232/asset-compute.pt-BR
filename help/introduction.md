---
title: Introdução ao [!DNL Asset Compute Service].
description: '[!DNL Asset Compute Service] é um serviço de processamento de ativos nativo na nuvem que reduz a complexidade e melhora a escalabilidade.'
translation-type: tm+mt
source-git-commit: 79630efa8cee2c8919d11e9bb3c14ee4ef54d0f3
workflow-type: tm+mt
source-wordcount: '321'
ht-degree: 0%

---


# Visão geral de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] é um serviço escalonável e extensível de processamento de ativos digitais [!DNL Adobe Experience Cloud] . Ele pode transformar imagens, vídeos, documentos e outros formatos de arquivo em diferentes representações, incluindo miniaturas, texto e metadados extraídos e arquivos.

Os desenvolvedores podem plugar aplicativos de ativos personalizados (também chamados de funcionários personalizados) para resolver casos de uso personalizados. O serviço funciona no [!DNL Adobe I/O] tempo de execução. É expansível por meio de aplicativos sem [!DNL Project Firefly] cabeçalho escritos em Node.js. Elas podem realizar operações personalizadas, como chamar APIs externas para executar operações de imagem ou aproveitar o [!DNL Adobe Sensei] suporte.

[!DNL Project Firefly] é uma estrutura para criar e implantar aplicativos da Web personalizados em [!DNL Adobe I/O] tempo de execução para estender as soluções da Adobe Experience Cloud. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (kit de ferramentas da interface do Adobe), criar microserviços, criar eventos personalizados e orquestrar APIs. Consulte a [documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Atualmente, o [!DNL Asset Compute Service] pode ser usado apenas via [!DNL Experience Manager] Cloud Service. Os administradores criam perfis de processamento que podem chamar o usuário para [!DNL Asset Compute Service] transmitir ativos para processamento. See [use asset microservices and processing profiles](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Casos de uso suportados de [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] apoia alguns casos comuns de utilização comercial, como o processamento básico de imagens; Conversões específicas da aplicação Adobe; e criação de aplicativos personalizados que orquestram requisitos comerciais complexos.

Você pode usar o serviço [!DNL Asset Compute] da Web para gerar miniaturas para diferentes tipos de arquivos, renderizações de imagem de alta qualidade para os formatos [de arquivo](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)suportados. Consulte casos de [uso suportados pela configuração](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)personalizada.

>[!NOTE]
>
>O serviço não fornece armazenamento de ativo. Os usuários fornecem e fornecem referências para locais de arquivos de origem e execução no armazenamento em nuvem.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use Adobe I/O Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Visão geral do processamento de ativos com microserviços de ativos no Adobe Experience Manager como Cloud Service](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Formatos de arquivo suportados para processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).
>* [Notas de versão do serviço de Asset compute](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
