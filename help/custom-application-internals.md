---
title: Entender o funcionamento de um aplicativo personalizado
description: Trabalho interno [!DNL Asset Compute Service] aplicativo personalizado para ajudar a entender como funciona.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: a121b48d480b45405259c2061ac86b9ab46b89cb
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Internais de um aplicativo personalizado {#how-custom-application-works}

Use a ilustração a seguir para entender o fluxo de trabalho completo quando um ativo digital for processado usando um aplicativo personalizado por um cliente.

![Fluxo de trabalho do aplicativo personalizado](assets/customworker.png)

*Figura: Etapas envolvidas para processar um ativo usando [!DNL Asset Compute Service].*

## Registro {#registration}

O cliente deve chamar [`/register`](api.md#register) uma vez antes da primeira solicitação para [`/process`](api.md#process-request) para configurar e recuperar o URL do diário para recebimento [!DNL Adobe I/O] Eventos para Asset compute Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

O [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) A biblioteca JavaScript pode ser usada em aplicativos NodeJS para lidar com todas as etapas necessárias, desde o registro, o processamento até a manipulação assíncrona de eventos. Para obter mais informações sobre os cabeçalhos necessários, consulte [Autenticação e autorização](api.md).

## Processando {#processing}

O cliente envia um [processamento](api.md#process-request) solicitação.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

O cliente é responsável por formatar corretamente as representações com URLs pré-assinados. O [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) A biblioteca JavaScript pode ser usada em aplicativos NodeJS para assinar previamente URLs. Atualmente, a biblioteca só oferece suporte ao Armazenamento de blobs do Azure e aos Contêineres do AWS S3.

A solicitação de processamento retorna um `requestId` que pode ser utilizado para sondar [!DNL Adobe I/O] Eventos.

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

O [!DNL Asset Compute Service] envia as solicitações de representação do aplicativo personalizado para o aplicativo personalizado. Ele usa um POST HTTP para o URL do aplicativo fornecido, que é o URL de ação da Web seguro do App Builder. Todas as solicitações usam o protocolo HTTPS para maximizar a segurança de dados.

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) usado por um aplicativo personalizado lida com a solicitação POST HTTP. Ele também lida com o download da fonte, o upload de renderizações, o envio [!DNL Adobe I/O] eventos e tratamento de erros.

<!-- TBD: Add the application diagram. -->

### Código do aplicativo {#application-code}

O código personalizado só precisa fornecer um retorno de chamada que recebe o arquivo de origem disponível localmente (`source.path`). O `rendition.path` é o local para colocar o resultado final de uma solicitação de processamento de ativos. O aplicativo personalizado usa o retorno de chamada para transformar os arquivos de origem disponíveis localmente em um arquivo de representação usando o nome passado (`rendition.path`). Um aplicativo personalizado deve gravar em `rendition.path` para criar uma representação:

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

Um aplicativo personalizado lida somente com arquivos locais. O download do arquivo de origem é feito pela [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Criação da representação {#rendition-creation}

O SDK chama um [função de retorno de chamada de representação](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) para cada representação.

A função de retorno de chamada tem acesso ao [source](https://github.com/adobe/asset-compute-sdk#source) e [representação](https://github.com/adobe/asset-compute-sdk#rendition) objetos. O `source.path` já existe e é o caminho para a cópia local do arquivo de origem. O `rendition.path` é o caminho onde a representação processada deve ser armazenada. A menos que [sinalizador disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) estiver definido, o aplicativo deverá usar exatamente o `rendition.path`. Caso contrário, o SDK não poderá localizar ou identificar o arquivo de representação e falhará.

A simplificação excessiva do exemplo é feita para ilustrar e focar na anatomia de um aplicativo personalizado. O aplicativo apenas copia o arquivo de origem para o destino de representação.

Para obter mais informações sobre os parâmetros de retorno de chamada de representação, consulte [API do SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Fazer upload de representações {#upload-rendition}

Depois que cada representação é criada e armazenada em um arquivo com o caminho fornecido por `rendition.path`, o [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carrega cada representação em um armazenamento na nuvem (AWS ou Azure). Um aplicativo personalizado obtém várias representações ao mesmo tempo se, e somente se, a solicitação recebida tiver várias representações apontando para o mesmo URL do aplicativo. O upload para o armazenamento em nuvem é feito após cada renderização e antes de executar o retorno de chamada para a próxima renderização.

O `batchWorker()` O tem um comportamento diferente, pois processa todas as renderizações e somente depois que todas foram processadas as carrega.

## [!DNL Adobe I/O] Eventos {#aio-events}

O SDK envia [!DNL Adobe I/O] Eventos para cada representação. Esses eventos são do tipo `rendition_created` ou `rendition_failed` dependendo do resultado. Consulte [Eventos assíncronos do Asset compute](api.md#asynchronous-events) para obter detalhes sobre eventos.

## Receber [!DNL Adobe I/O] Eventos {#receive-aio-events}

O cliente pesquisa a [[!DNL Adobe I/O] Diário de Eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) de acordo com a sua lógica de consumo. A URL inicial do diário é a fornecida no `/register` Resposta da API. Os eventos podem ser identificados usando a variável `requestId` que está presente nos eventos e é igual ao retornado em `/process`. Cada representação tem um evento separado que é enviado assim que a representação é carregada (ou falha). Depois de receber um evento correspondente, o cliente pode exibir ou manipular as representações resultantes.

A biblioteca do JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) torna a pesquisa de jornal simples usando o `waitActivation()` para obter todos os eventos.

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

Para obter detalhes sobre como obter eventos de diário, consulte [[!DNL Adobe I/O] API Eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
