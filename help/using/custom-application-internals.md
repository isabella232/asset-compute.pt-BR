---
title: Entender o funcionamento de um aplicativo personalizado
description: Funcionamento interno de [!DNL Asset Compute Service] aplicativo personalizado para ajudar a entender como funciona.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# Componentes internos de um aplicativo personalizado {#how-custom-application-works}

Use a ilustração a seguir para entender o fluxo de trabalho completo quando um ativo digital é processado usando um aplicativo personalizado por um cliente.

![Fluxo de trabalho do aplicativo personalizado](assets/customworker.svg)

*Figura: Etapas envolvidas para processar um ativo usando o [!DNL Asset Compute Service].*

## Registro {#registration}

O cliente deve chamar [`/register`](api.md#register) uma vez antes da primeira solicitação para [`/process`](api.md#process-request) para configurar e recuperar o URL do diário para recebimento [!DNL Adobe I/O] Eventos para Adobe Asset compute.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

A variável [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) A biblioteca JavaScript pode ser usada em aplicativos NodeJS para lidar com todas as etapas necessárias, desde o registro e o processamento até a manipulação de eventos assíncrona. Para obter mais informações sobre os cabeçalhos obrigatórios, consulte [Autenticação e autorização](api.md).

## Processando {#processing}

O cliente envia um [processando](api.md#process-request) solicitação.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

O cliente é responsável por formatar corretamente as representações com URLs pré-assinados. A variável [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) A biblioteca JavaScript do pode ser usada nos aplicativos NodeJS para pré-assinar URLs. Atualmente, a biblioteca é compatível apenas com o Armazenamento de blobs do Azure e com os Contêineres do AWS S3.

A solicitação de processamento retorna um `requestId` que pode ser usado para polling [!DNL Adobe I/O] Eventos.

Um exemplo de solicitação de processamento de aplicativo personalizado está abaixo.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

A variável [!DNL Asset Compute Service] envia as solicitações de representação do aplicativo personalizado para o aplicativo personalizado. Ele usa um POST HTTP para o URL de aplicativo fornecido, que é o URL de ação da Web protegido do App Builder. Todas as solicitações usam o protocolo HTTPS para maximizar a segurança dos dados.

A variável [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) usado por um aplicativo personalizado manipula a solicitação HTTP POST. Também lida com download da origem, upload de representações, envio [!DNL Adobe I/O] eventos e tratamento de erros.

<!-- TBD: Add the application diagram. -->

### Código do aplicativo {#application-code}

O código personalizado precisa apenas fornecer um retorno de chamada que use o arquivo de origem disponível localmente (`source.path`). A variável `rendition.path` é o local para colocar o resultado final de uma solicitação de processamento de ativo. O aplicativo personalizado usa o retorno de chamada para transformar os arquivos de origem disponíveis localmente em um arquivo de representação usando o nome transmitido (`rendition.path`). Um aplicativo personalizado deve gravar em `rendition.path` para criar uma representação:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Baixar arquivos de origem {#download-source}

Um aplicativo personalizado só lida com arquivos locais. O download do arquivo de origem é tratado pelo [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Criação de representação {#rendition-creation}

O SDK chama uma instância [função de retorno de chamada de representação](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) para cada representação.

A função de retorno de chamada tem acesso à variável [origem](https://github.com/adobe/asset-compute-sdk#source) e [representação](https://github.com/adobe/asset-compute-sdk#rendition) objetos. A variável `source.path` já existe e é o caminho para a cópia local do arquivo de origem. A variável `rendition.path` é o caminho onde a representação processada deve ser armazenada. A menos que a variável [sinalizador disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) for definido, o aplicativo deverá usar exatamente o `rendition.path`. Caso contrário, o SDK não poderá localizar ou identificar o arquivo de representação e falhará.

A simplificação excessiva do exemplo é feita para ilustrar e se concentrar na anatomia de um aplicativo personalizado. O aplicativo apenas copia o arquivo de origem para o destino da representação.

Para obter mais informações sobre os parâmetros de retorno de chamada de representação, consulte [API do SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Fazer upload de representações {#upload-rendition}

Após cada representação ser criada e armazenada em um arquivo com o caminho fornecido por `rendition.path`, o [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) O carrega cada representação em um armazenamento na nuvem (AWS ou Azure). Um aplicativo personalizado obtém várias representações ao mesmo tempo se, e somente se, a solicitação recebida tiver várias representações apontando para o mesmo URL de aplicativo. O upload para o armazenamento na nuvem é feito após cada representação e antes de executar o retorno de chamada para a próxima representação.

A variável `batchWorker()` O tem um comportamento diferente, pois processa todas as representações e somente depois que todas são processadas as carrega.

## [!DNL Adobe I/O] Eventos {#aio-events}

O SDK envia [!DNL Adobe I/O] Eventos para cada representação. Esses eventos são do tipo `rendition_created` ou `rendition_failed` dependendo do resultado. Consulte [Asset compute de eventos assíncronos](api.md#asynchronous-events) para obter detalhes sobre eventos.

## Receber [!DNL Adobe I/O] Eventos {#receive-aio-events}

O cliente pesquisa a variável [[!DNL Adobe I/O] Diário de eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) de acordo com sua lógica de consumo. O URL inicial do diário é o fornecido na variável `/register` Resposta da API. Os eventos podem ser identificados usando o `requestId` que está presente nos eventos e é igual ao retornado em `/process`. Cada representação tem um evento separado que é enviado assim que a representação é carregada (ou falha). Depois que ele receber um evento correspondente, o cliente poderá exibir ou manipular as representações resultantes.

A biblioteca do JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) torna a pesquisa de diário simples usando o `waitActivation()` para obter todos os eventos.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Para obter detalhes sobre como obter eventos de diário, consulte [[!DNL Adobe I/O] API de eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
