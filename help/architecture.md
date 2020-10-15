---
title: Arquitetura [!DNL Asset Compute Service].
description: A [!DNL Asset Compute Service] HowAPI, os aplicativos e o SDK trabalham juntos para fornecer um serviço de processamento de ativos nativo na nuvem.
translation-type: tm+mt
source-git-commit: 0fb256f7d9f83fbae564d9fd52ee6b2f34c5d7e9
workflow-type: tm+mt
source-wordcount: '496'
ht-degree: 0%

---


# Arquitetura de [!DNL Asset Compute Service] {#overview}

O [!DNL Asset Compute Service] é construído sobre a plataforma sem servidor Adobe I/O Runtime. Ele oferece suporte aos serviços de conteúdo da Adobe Sensei para ativos. O cliente que invoca (somente [!DNL Experience Manager] como um Cloud Service é suportado) é fornecido com as informações geradas pela Adobe Sensei que ele procurou para o ativo. As informações retornadas estão no formato JSON.

[!DNL Asset Compute Service] é extensível ao criar aplicativos personalizados baseados em [!DNL Project Firefly]. Esses aplicativos personalizados são aplicativos sem [!DNL Project Firefly] cabeçalho e fazem tarefas, como adicionar ferramentas de conversão personalizadas ou chamar APIs externas para executar operações de imagem.

[!DNL Project Firefly] é uma estrutura para criar e implantar aplicativos da Web personalizados em [!DNL Adobe I/O] tempo de execução. Para criar aplicativos personalizados, os desenvolvedores podem aproveitar [!DNL React Spectrum] (kit de ferramentas da interface do Adobe), criar microserviços, criar eventos personalizados e orquestrar APIs. Consulte a [documentação do Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

A base na qual a arquitetura se baseia inclui:

* A modularidade das aplicações - que contêm apenas o que é necessário para uma determinada tarefa - permite dissociar as aplicações entre si e mantê-las leves.

* O conceito sem servidor da Adobe I/O Runtime traz inúmeros benefícios: processamento assíncrono, altamente escalável, isolado, baseado em trabalho, que é a melhor opção para o processamento de ativos.

* O armazenamento da nuvem binária fornece os recursos necessários para armazenar e acessar arquivos de ativos e execuções individualmente, sem exigir permissões de acesso total ao armazenamento, usando referências de URL pré-assinadas. Aceleração de transferência, armazenamento em cache de CDN e co-localização de aplicativos de computação com armazenamento em nuvem permitem acesso ideal a conteúdo de baixa latência. Há suporte para nuvens AWS e Azure.

![Arquitetura do Serviço de Computação de Ativos](assets/architecture-diagram.png)

*Figura: Arquitetura de [!DNL Asset Compute Service] e como ela se integra ao aplicativo [!DNL Experience Manager], armazenamento e processamento.*

A arquitetura consiste nas seguintes partes:

* **Uma API e uma camada** de orquestração recebem solicitações (no formato JSON) que instruem o serviço a transformar um ativo de origem em várias execuções. Essas solicitações são assíncronas e retornam com uma ID de ativação (também conhecida como &quot;id de trabalho&quot;). As instruções são puramente declarativas e para todo o trabalho de processamento padrão (por exemplo, geração de miniaturas, extração de texto) os consumidores especificam apenas o resultado desejado, mas não os aplicativos que manipulam determinadas execuções. Recursos de API genéricos, como autenticação, análise, limitação de taxa, são manipulados usando o Gateway de API Adobe na frente do serviço e gerencia todas as solicitações que vão para o I/O Runtime. O roteamento do aplicativo é feito dinamicamente pela camada de orquestração. O aplicativo personalizado pode ser especificado pelos clientes para execuções específicas e incluir parâmetros personalizados. A execução do aplicativo pode ser totalmente paralelizada, pois são funções separadas sem servidor no I/O Runtime.

* **Aplicativos para processar ativos** especializados em determinados tipos de formatos de arquivo ou execuções de público alvo. Conceitualmente, um aplicativo é como o conceito Unix pipe: um arquivo de entrada é transformado em um ou mais arquivos de saída.

* **Uma biblioteca [de aplicativos](https://github.com/adobe/asset-compute-sdk)** comum lida com tarefas comuns, como baixar o arquivo de origem, fazer upload das execuções, relatórios de erro, envio e monitoramento de eventos. Isso é projetado para que o desenvolvimento de um aplicativo permaneça o mais simples possível, seguindo a ideia sem servidor, e possa ser restrito às interações locais do sistema de arquivos.

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
