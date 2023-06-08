---
title: Entenda sobre a extensão [!DNL Asset Compute Service]
description: Quando e como estender a [!DNL Asset Compute Service] funcionalidade para fazer o processamento de ativos personalizados.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introdução à extensibilidade {#introduction-to-extensibilty}

Muitos requisitos de representação, como a conversão para formatos e o redimensionamento de imagens, são abordados pelo [Processamento de perfis no [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=pt-BR). Necessidades de negócios mais complexas podem precisar de uma solução criada de forma personalizada que atenda às necessidades de uma organização. [!DNL Asset Compute Service] pode ser estendido criando aplicativos personalizados chamados de Perfis de processamento no [!DNL Experience Manager]. Esses aplicativos personalizados atendem aos [casos de uso suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com [!DNL Experience Manager] as a [!DNL Cloud Service].

Os aplicativos personalizados são headless [Construtor de aplicativos Adobe Developer](https://github.com/AdobeDocs/app-builder) aplicativos. Extensão [!DNL Asset Compute Service] com aplicativos personalizados é simplificado por meio do [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) e ferramentas de desenvolvedor do Adobe Developer App Builder. Isso permite que os desenvolvedores se concentrem na lógica de negócios. Criar aplicativos personalizados é tão simples quanto criar um simples servidor [!DNL Adobe I/O] Ação em tempo de execução. É uma única função JavaScript Node.js. A variável [exemplo de aplicativo personalizado básico](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) O ilustra isso.

## Pré-requisitos e requisitos de provisionamento {#prerequisites-and-provisioning}

Certifique-se de atender aos seguintes pré-requisitos:

* As ferramentas do Adobe Developer App Builder estão instaladas no computador.
* Um [!DNL Experience Cloud] organização. Mais informações [aqui](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* A Organização da experiência deve ter [!DNL Experience Manager] as a [!DNL Cloud Service] ativado.
* [!DNL Adobe Experience Cloud] a organização faz parte da [!DNL Adobe Developer App Builder] programa de visualização do desenvolvedor. Consulte [como solicitar acesso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Garanta uma função de desenvolvedor ou permissões de administrador na organização para o desenvolvedor.
* Assegure que [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) O é instalado localmente.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
