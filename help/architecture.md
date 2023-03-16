---
title: Arquitetura do [!DNL Asset Compute Service]
description: How [!DNL Asset Compute Service] API, aplicativos e SDK trabalham juntos para fornecer um serviço de processamento de ativos nativo em nuvem.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 0c5ab8ab230e3f033b804849a2c005351542155f
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Arquitetura do [!DNL Asset Compute Service] {#overview}

O [!DNL Asset Compute Service] é criado sobre o servidor sem servidor [!DNL Adobe I/O] Plataforma de tempo de execução. Ela oferece suporte a serviços de conteúdo da Adobe Sensei para ativos. O cliente que invoca (apenas [!DNL Experience Manager] como [!DNL Cloud Service] é suportado) é fornecido com as informações geradas pela Adobe Sensei que ele buscou para o ativo. As informações retornadas estão no formato JSON.

[!DNL Asset Compute Service] é expansível ao criar aplicativos personalizados com base em [!DNL Project Adobe Developer App Builder]. Esses aplicativos personalizados são [!DNL Project Adobe Developer App Builder] aplicativos sem periféricos e tarefas como adicionar ferramentas de conversão personalizadas ou chamar APIs externas para executar operações de imagem.

[!DNL Project Adobe Developer App Builder] O é uma estrutura para criar e implantar aplicativos Web personalizados no [!DNL Adobe I/O] tempo de execução. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (Kit de ferramentas da interface do usuário do Adobe), crie microsserviços, crie eventos personalizados e orquestre APIs. Consulte [documentação do Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).

A base na qual a arquitetura se baseia inclui:

* A modularidade de aplicativos - contendo apenas o que é necessário para uma determinada tarefa - permite dissociar os aplicativos uns dos outros e mantê-los leves.

* O conceito sem servidor de [!DNL Adobe I/O] O tempo de execução gera vários benefícios: processamento assíncrono, altamente escalável, isolado e baseado em tarefas, que é a opção perfeita para o processamento de ativos.

* O armazenamento em nuvem binário fornece os recursos necessários para armazenar e acessar arquivos de ativos e representações individualmente, sem exigir permissões de acesso total ao armazenamento, usando referências de URL pré-assinadas. Aceleração de transferência, armazenamento em cache de CDN e co-localização de aplicativos de computação com armazenamento em nuvem permitem acesso ideal a conteúdo de baixa latência. As nuvens do AWS e do Azure são compatíveis.

![Arquitetura do Asset compute Service](assets/architecture-diagram.png)

*Figura: Arquitetura do [!DNL Asset Compute Service] e como ele se integra ao [!DNL Experience Manager], armazenamento e aplicativo de processamento.*

A arquitetura consiste nas seguintes partes:

* **Uma camada de API e orquestração** O recebe solicitações (no formato JSON) que instruem o serviço a transformar um ativo de origem em várias representações. As solicitações são assíncronas e retornam com uma ID de ativação, ou seja, uma ID de trabalho. As instruções são meramente declarativas e, para todo o trabalho de processamento padrão (por exemplo, geração de miniaturas, extração de texto), os consumidores especificam apenas o resultado desejado, mas não os aplicativos que manipulam determinadas representações. Recursos genéricos da API, como autenticação, análise, limitação de taxa, são manipulados usando o Gateway da API do Adobe na frente do serviço e gerencia todas as solicitações enviadas para [!DNL Adobe I/O] Tempo de execução. O roteamento do aplicativo é feito dinamicamente pela camada de orquestração. O aplicativo personalizado pode ser especificado pelos clientes para representações específicas e incluir parâmetros personalizados. A execução do aplicativo pode ser totalmente paralisada, pois são funções separadas sem servidor em [!DNL Adobe I/O] Tempo de execução.

* **Aplicativos para processar ativos** que se especializam em determinados tipos de formatos de arquivo ou renderizações de destino. Conceitualmente, um aplicativo é como o conceito de pipe Unix: um arquivo de entrada é transformado em um ou mais arquivos de saída.

* **A [biblioteca de aplicativos comum](https://github.com/adobe/asset-compute-sdk)** O lida com tarefas comuns, como baixar o arquivo de origem, fazer upload das representações, relatórios de erros, envio e monitoramento de eventos . Isso é projetado para que o desenvolvimento de um aplicativo seja o mais simples possível, seguindo a ideia sem servidor, e possa ser restrito às interações locais do sistema de arquivos.

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
