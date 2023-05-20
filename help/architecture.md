---
title: Arquitetura de [!DNL Asset Compute Service]
description: Como [!DNL Asset Compute Service] API, aplicativos e SDK trabalham juntos para fornecer um serviço de processamento de ativos nativo em nuvem.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 0c5ab8ab230e3f033b804849a2c005351542155f
workflow-type: tm+mt
source-wordcount: '486'
ht-degree: 0%

---

# Arquitetura de [!DNL Asset Compute Service] {#overview}

A variável [!DNL Asset Compute Service] é construído sobre um servidor sem servidor [!DNL Adobe I/O] Plataforma de tempo de execução. Ele oferece suporte aos serviços de conteúdo da Adobe Sensei para ativos. O cliente que efetua a chamada (apenas [!DNL Experience Manager] as a [!DNL Cloud Service] é compatível) é fornecido com as informações geradas pela Adobe Sensei que ela buscou para o ativo. As informações retornadas estão no formato JSON.

[!DNL Asset Compute Service] é extensível criando aplicativos personalizados com base em [!DNL Project Adobe Developer App Builder]. Esses aplicativos personalizados são [!DNL Project Adobe Developer App Builder] aplicativos headless e executam tarefas, como adicionar ferramentas de conversão personalizadas ou chamar APIs externas para executar operações de imagem.

[!DNL Project Adobe Developer App Builder] O é uma estrutura para criar e implantar aplicativos Web personalizados no [!DNL Adobe I/O] tempo de execução. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (Kit de ferramentas da interface do Adobe), crie microsserviços, eventos personalizados e organize APIs. Consulte [documentação do Construtor de aplicativos Adobe Developer](https://developer.adobe.com/app-builder/docs/overview).

A base na qual a arquitetura se baseia inclui:

* A modularidade dos aplicativos - contendo apenas o que é necessário para uma determinada tarefa - permite dissociar os aplicativos uns dos outros e mantê-los mais leves.

* O conceito sem servidor do [!DNL Adobe I/O] O tempo de execução gera vários benefícios: processamento assíncrono, altamente escalável, isolado e baseado em trabalho, que é perfeito para o processamento de ativos.

* O armazenamento binário em nuvem fornece os recursos necessários para armazenar e acessar arquivos de ativos e representações individualmente, sem exigir permissões de acesso total ao armazenamento, usando referências de URL pré-assinadas. A aceleração de transferência, o armazenamento em cache de CDN e a co-localização de aplicativos de computação com o armazenamento em nuvem permitem o acesso ideal ao conteúdo de baixa latência. As nuvens do AWS e do Azure são compatíveis.

![Arquitetura do serviço de Asset compute](assets/architecture-diagram.png)

*Figura: Arquitetura do [!DNL Asset Compute Service] e como ele se integra ao [!DNL Experience Manager], armazenamento e processamento de aplicativos.*

A arquitetura consiste nas seguintes partes:

* **Uma camada de API e orquestração** O recebe solicitações (no formato JSON) que instruem o serviço a transformar um ativo de origem em várias representações. As solicitações são assíncronas e retornam com uma ID de ativação, ou seja, ID do trabalho. As instruções são puramente declarativas e, para todo o trabalho de processamento padrão (por exemplo, geração de miniaturas, extração de texto), os consumidores especificam apenas o resultado desejado, mas não os aplicativos que lidam com determinadas representações. Os recursos genéricos da API, como autenticação, análise, limitação de taxa, são tratados usando o Gateway da API de Adobe na frente do serviço e gerenciam todas as solicitações em [!DNL Adobe I/O] Tempo de execução. O roteamento de aplicativos é feito dinamicamente pela camada de orquestração. O aplicativo personalizado pode ser especificado por clientes para representações específicas e incluir parâmetros personalizados. A execução de aplicativos pode ser totalmente paralelizada, pois são funções sem servidor separadas no [!DNL Adobe I/O] Tempo de execução.

* **Aplicativos para processar ativos** especializados em determinados tipos de formatos de arquivo ou representações de destino. Conceitualmente, um aplicativo é como o conceito de pipe Unix: um arquivo de entrada é transformado em um ou mais arquivos de saída.

* **A [biblioteca de aplicativos comum](https://github.com/adobe/asset-compute-sdk)** O lida com tarefas comuns, como baixar o arquivo de origem, fazer upload das representações, relatórios de erros, envio e monitoramento de eventos. Ele foi projetado para que o desenvolvimento de um aplicativo permaneça o mais simples possível, seguindo a ideia de um sistema sem servidor, e possa ser restrito às interações do sistema de arquivos local.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
