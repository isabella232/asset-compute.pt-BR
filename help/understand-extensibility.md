---
title: Entenda sobre como estender [!DNL Asset Compute Service]
description: Quando e como estender a funcionalidade [!DNL Asset Compute Service] para fazer o processamento de ativos personalizados.
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 1%

---


# Introdução à extensibilidade {#introduction-to-extensibilty}

Muitos requisitos de execução, como conversão em formatos e redimensionamento de imagens, são abordados por [Perfis de processamento em [!DNL Experience Manager] como  [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). Os requisitos de negócios mais complexos podem precisar de uma solução personalizada que atenda às necessidades de uma organização. [!DNL Asset Compute Service] pode ser estendido criando aplicativos personalizados chamados de Perfis de processamento em  [!DNL Experience Manager]. Esses aplicativos personalizados atendem aos [casos de uso suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com  [!DNL Experience Manager] como  [!DNL Cloud Service].

Os aplicativos personalizados são aplicativos sem cabeçalho [Project Firefly](https://github.com/AdobeDocs/project-firefly). A extensão [!DNL Asset Compute Service] com aplicativos personalizados é simplificada por meio do SDK [Asset compute](https://github.com/adobe/asset-compute-sdk) e das ferramentas do desenvolvedor do Project Firefly. Isso permite que os desenvolvedores se concentrem na lógica comercial. Criar aplicativos personalizados é tão simples quanto criar uma ação sem servidor simples [!DNL Adobe I/O] Runtime. É uma única função JavaScript Node.js. O [exemplo básico do aplicativo personalizado](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) ilustra isso.

## Pré-requisitos e requisitos de provisionamento {#prerequisites-and-provisioning}

Certifique-se de atender aos seguintes pré-requisitos:

* As ferramentas do Project Firefly são instaladas em sua máquina.
* Uma organização [!DNL Experience Cloud]. Mais informações [aqui](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* A organização da experiência deve ter [!DNL Experience Manager] como um [!DNL Cloud Service] ativado.
* [!DNL Adobe Experience Cloud] organização é parte do programa de pré-visualização do  [!DNL Project Firefly] desenvolvedor. Consulte [como solicitar acesso](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Certifique-se de uma função de desenvolvedor ou de permissões de administrador na organização para o desenvolvedor.
* Verifique se [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) está instalado localmente.

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
