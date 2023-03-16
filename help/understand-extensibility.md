---
title: Entender sobre extensão [!DNL Asset Compute Service]
description: Quando e como estender o [!DNL Asset Compute Service] para fazer processamento de ativos personalizados.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introdução à extensibilidade {#introduction-to-extensibilty}

Muitos requisitos de representação, como conversão em formatos e redimensionamento de imagens, são abordados por [Processando perfis em [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=pt-BR). Os requisitos de negócios mais complexos podem precisar de uma solução personalizada que atenda às necessidades de uma organização. [!DNL Asset Compute Service] pode ser estendido criando aplicativos personalizados chamados de Perfis de processamento em [!DNL Experience Manager]. Esses aplicativos personalizados atendem ao [casos de uso suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com [!DNL Experience Manager] como [!DNL Cloud Service].

Os aplicativos personalizados não têm cabeça [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) aplicativos. Extensão [!DNL Asset Compute Service] com aplicativos personalizados, o [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk) e ferramentas de desenvolvedor do Adobe Developer App Builder. Isso permite que os desenvolvedores se concentrem na lógica de negócios. Criar aplicativos personalizados é tão simples como criar um servidor simples [!DNL Adobe I/O] Ação de tempo de execução. É uma única função JavaScript Node.js . O [exemplo básico de aplicativo personalizado](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) ilustra isso.

## Pré-requisitos e requisitos de provisionamento {#prerequisites-and-provisioning}

Siga os seguintes pré-requisitos:

* As ferramentas do Adobe Developer App Builder são instaladas na máquina.
* Um [!DNL Experience Cloud] organização. Mais informações [here](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* A organização da experiência deve ter [!DNL Experience Manager] como [!DNL Cloud Service] habilitado.
* [!DNL Adobe Experience Cloud] faz parte da [!DNL Adobe Developer App Builder] programa de visualização do desenvolvedor. Consulte [como solicitar acesso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Assegure uma função de desenvolvedor ou permissões de administrador na organização para o desenvolvedor.
* Certifique-se de que [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) está instalado localmente.

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
