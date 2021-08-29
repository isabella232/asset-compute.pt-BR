---
title: Arquitetura de [!DNL Asset Compute Service]
description: 'Como a API, os aplicativos e o SDK trabalham juntos para fornecer um serviço de processamento de ativos nativo em nuvem. [!DNL Asset Compute Service] '
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '485'
ht-degree: 0%

---

# Arquitetura de [!DNL Asset Compute Service] {#overview}

O [!DNL Asset Compute Service] é criado sobre a plataforma sem servidor [!DNL Adobe I/O] Runtime. Ela oferece suporte a serviços de conteúdo da Adobe Sensei para ativos. O cliente que invoca (somente [!DNL Experience Manager] como [!DNL Cloud Service] é suportado) é fornecido com as informações geradas pela Adobe Sensei que ele buscou para o ativo. As informações retornadas estão no formato JSON.

[!DNL Asset Compute Service] pode ser estendida ao criar aplicativos personalizados com base em  [!DNL Project Firefly]. Esses aplicativos personalizados são [!DNL Project Firefly] aplicativos sem periféricos e fazem tarefas como adicionar ferramentas de conversão personalizadas ou chamar APIs externas para executar operações de imagem.

[!DNL Project Firefly] O é uma estrutura para criar e implantar aplicativos Web personalizados no  [!DNL Adobe I/O] tempo de execução. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (kit de ferramentas da interface do usuário do Adobe), criar microsserviços, criar eventos personalizados e orquestrar APIs. Consulte a [documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

A base na qual a arquitetura se baseia inclui:

* A modularidade de aplicativos - contendo apenas o que é necessário para uma determinada tarefa - permite dissociar os aplicativos uns dos outros e mantê-los leves.

* O conceito sem servidor de [!DNL Adobe I/O] Tempo de execução gera vários benefícios: processamento assíncrono, altamente escalável, isolado e baseado em tarefas, que é a opção perfeita para o processamento de ativos.

* O armazenamento em nuvem binário fornece os recursos necessários para armazenar e acessar arquivos de ativos e representações individualmente, sem exigir permissões de acesso total ao armazenamento, usando referências de URL pré-assinadas. Aceleração de transferência, armazenamento em cache de CDN e co-localização de aplicativos de computação com armazenamento em nuvem permitem acesso ideal a conteúdo de baixa latência. As nuvens do AWS e do Azure são compatíveis.

![Arquitetura do serviço Asset compute](assets/architecture-diagram.png)

*Figura: Arquitetura  [!DNL Asset Compute Service] e como ela se integra ao aplicativo de  [!DNL Experience Manager], armazenamento e processamento.*

A arquitetura consiste nas seguintes partes:

* **Uma** camada de API e orquestração recebe solicitações (em formato JSON) que instruem o serviço a transformar um ativo de origem em várias representações. As solicitações são assíncronas e retornam com uma ID de ativação, ou seja, uma ID de trabalho. As instruções são meramente declarativas e, para todo o trabalho de processamento padrão (por exemplo, geração de miniaturas, extração de texto), os consumidores especificam apenas o resultado desejado, mas não os aplicativos que manipulam determinadas representações. Recursos genéricos da API, como autenticação, análise, limitação de taxa, são manipulados usando o Gateway da API do Adobe na frente do serviço e gerencia todas as solicitações em [!DNL Adobe I/O] Tempo de execução. O roteamento do aplicativo é feito dinamicamente pela camada de orquestração. O aplicativo personalizado pode ser especificado pelos clientes para representações específicas e incluir parâmetros personalizados. A execução do aplicativo pode ser totalmente paralelizada, pois são funções sem servidor separadas em [!DNL Adobe I/O] Tempo de execução.

* **Aplicativos para processar** ativos especializados em determinados tipos de formatos de arquivo ou renderizações de destino. Conceitualmente, um aplicativo é como o conceito de pipe Unix: um arquivo de entrada é transformado em um ou mais arquivos de saída.

* **Uma  [biblioteca de aplicativos ](https://github.com/adobe/asset-compute-sdk)** comum lida com tarefas comuns, como baixar o arquivo de origem, carregar as representações, relatar erros, enviar e monitorar eventos. Isso é projetado para que o desenvolvimento de um aplicativo seja o mais simples possível, seguindo a ideia sem servidor, e possa ser restrito às interações locais do sistema de arquivos.

<!-- TBD:

* About the YAML file?
* See [https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
