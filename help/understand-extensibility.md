---
title: Entenda sobre como estender [!DNL Asset Compute Service]
description: Quando e como estender a funcionalidade [!DNL Asset Compute Service] para fazer processamento de ativos personalizados.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '254'
ht-degree: 1%

---

# Introdução à extensibilidade {#introduction-to-extensibilty}

Muitos requisitos de representação, como conversão em formatos e redimensionamento de imagens, são endereçados por [Processando Perfis em [!DNL Experience Manager] como a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). Os requisitos de negócios mais complexos podem precisar de uma solução personalizada que atenda às necessidades de uma organização. [!DNL Asset Compute Service] O pode ser estendido criando aplicativos personalizados que são chamados de Perfis de processamento no  [!DNL Experience Manager]. Esses aplicativos personalizados atendem aos [casos de uso suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com  [!DNL Experience Manager] as a  [!DNL Cloud Service].

Os aplicativos personalizados são aplicativos sem cabeçalho [Project Firefly](https://github.com/AdobeDocs/project-firefly). A extensão [!DNL Asset Compute Service] com aplicativos personalizados é simplificada por meio das ferramentas de desenvolvedor [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) e Project Firefly. Isso permite que os desenvolvedores se concentrem na lógica de negócios. Criar aplicativos personalizados é tão simples como criar uma ação sem servidor simples [!DNL Adobe I/O] Runtime. É uma única função JavaScript Node.js . O [exemplo de aplicativo personalizado básico](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) ilustra isso.

## Pré-requisitos e requisitos de provisionamento {#prerequisites-and-provisioning}

Siga os seguintes pré-requisitos:

* As ferramentas do Project Firefly são instaladas na máquina.
* Uma organização [!DNL Experience Cloud]. Mais informações [aqui](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials).
* A organização da experiência deve ter [!DNL Experience Manager] como um [!DNL Cloud Service] ativado.
* [!DNL Adobe Experience Cloud] organização faz parte do programa de visualização do  [!DNL Project Firefly] desenvolvedor. Consulte [como solicitar acesso](https://www.adobe.io/project-firefly/docs/overview/getting_access/).
* Assegure uma função de desenvolvedor ou permissões de administrador na organização para o desenvolvedor.
* Certifique-se de que [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) esteja instalado localmente.

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
