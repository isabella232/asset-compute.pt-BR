---
title: Entenda o funcionamento de um aplicativo personalizado.
description: Funcionamento interno do aplicativo personalizado  [!DNL Asset Compute Service] para ajudar a entender como ele funciona.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# Internais de um aplicativo personalizado {#how-custom-application-works}

Use a ilustração a seguir para entender o fluxo de trabalho completo quando um ativo digital for processado usando um aplicativo personalizado por um cliente.

![Fluxo de trabalho do aplicativo personalizado](assets/customworker.png)

*Figura: Etapas envolvidas para processar um Ativo usando  [!DNL Asset Compute Service].*

## Registro {#registration}

O cliente deve chamar [`/register`](api.md#register) uma vez antes da primeira solicitação para [`/process`](api.md#process-request) a fim de configurar e recuperar o URL do journal para receber Eventos Adobe I/O para Asset compute Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

A biblioteca JavaScript [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) pode ser usada em aplicativos NodeJS para manipular todas as etapas necessárias, desde o registro, o processamento até a manipulação assíncrona de eventos. Para obter mais informações sobre os cabeçalhos necessários, consulte [Autenticação e autorização](api.md).

## Processando {#processing}

O cliente envia uma solicitação [processing](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

O cliente é responsável por formatar corretamente as execuções com URLs pré-assinados. A biblioteca JavaScript [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) pode ser usada em aplicativos NodeJS para pré-assinar URLs. Atualmente, a biblioteca suporta apenas Container Blob do Azure e Armazenamentos AWS S3.

A solicitação de processamento retorna um `requestId` que pode ser usado para sondar Eventos Adobe I/O.

Uma amostra de solicitação de processamento de aplicativo personalizado está abaixo.

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

O [!DNL Asset Compute Service] envia as solicitações de execução do aplicativo personalizado para o aplicativo personalizado. Ele usa um POST HTTP para o URL do aplicativo fornecido, que é o URL de ação da Web protegido do Project Firefly. Todas as solicitações usam o protocolo HTTPS para maximizar a segurança dos dados.

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) usado por um aplicativo personalizado lida com a solicitação de POST HTTP. Ele também lida com o download da origem, o upload de execuções, o envio de eventos de E/S e a manipulação de erros.

<!-- TBD: Add the application diagram. -->

### Código do aplicativo {#application-code}

O código personalizado só precisa fornecer um retorno de chamada que leve o arquivo de origem disponível localmente (`source.path`). O `rendition.path` é o local para colocar o resultado final de uma solicitação de processamento de ativos. O aplicativo personalizado usa o retorno de chamada para transformar os arquivos de origem disponíveis localmente em um arquivo de execução usando o nome passado (`rendition.path`). Um aplicativo personalizado deve gravar em `rendition.path` para criar uma execução:

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

Um aplicativo personalizado lida somente com arquivos locais. O download do arquivo de origem é realizado pelo [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Criação da execução {#rendition-creation}

O SDK chama uma [função de retorno de chamada de representação assíncrona](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) para cada execução.

A função de retorno de chamada tem acesso aos objetos [source](https://github.com/adobe/asset-compute-sdk#source) e [rendition](https://github.com/adobe/asset-compute-sdk#rendition). O `source.path` já existe e é o caminho para a cópia local do arquivo de origem. O `rendition.path` é o caminho no qual a representação processada deve ser armazenada. A menos que o sinalizador [disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) esteja definido, o aplicativo deve usar exatamente o `rendition.path`. Caso contrário, o SDK não poderá localizar ou identificar o arquivo de execução e falhará.

A simplificação excessiva do exemplo é feita para ilustrar e focar na anatomia de um aplicativo personalizado. O aplicativo simplesmente copia o arquivo de origem para o destino de execução.

Para obter mais informações sobre os parâmetros de retorno de chamada de representação, consulte [API SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Carregar execuções {#upload-rendition}

Depois que cada renderização é criada e armazenada em um arquivo com o caminho fornecido por `rendition.path`, o [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carrega cada renderização em um armazenamento de nuvem (AWS ou Azure). Um aplicativo personalizado obtém várias execuções ao mesmo tempo se, e somente se, a solicitação recebida tiver várias execuções apontando para o mesmo URL do aplicativo. O upload para o armazenamento da nuvem é feito após cada execução e antes de executar o retorno de chamada para a próxima execução.

O `batchWorker()` tem um comportamento diferente, já que processa todas as execuções e somente depois de todas terem sido processadas as carrega.

## eventos Adobe I/O {#aio-events}

O SDK envia Eventos do Adobe I/O para cada execução. Esses eventos são do tipo `rendition_created` ou `rendition_failed`, dependendo do resultado. Consulte [eventos assíncronos do Asset compute](api.md#asynchronous-events) para obter detalhes sobre os eventos.

## Receber Eventos Adobe I/O {#receive-aio-events}

O cliente pesquisa o Journal [Eventos Adobe I/O](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) de acordo com sua lógica de consumo. O URL do journal inicial é aquele fornecido na resposta da API `/register`. Os eventos podem ser identificados usando o `requestId` que está presente nos eventos e é o mesmo que retornado em `/process`. Cada execução tem um evento separado que é enviado assim que a execução é carregada (ou falha). Depois de receber um evento correspondente, o cliente pode exibir ou manipular as execuções resultantes.

A biblioteca JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) torna a pesquisa de journais simples usando o método `waitActivation()` para obter todos os eventos.

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

Para obter detalhes sobre como obter eventos de journais, consulte [API Eventos Adobe I/O](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
