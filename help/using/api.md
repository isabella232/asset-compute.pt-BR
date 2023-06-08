---
title: "[!DNL Asset Compute Service] API HTTP"
description: "[!DNL Asset Compute Service] API HTTP para criar aplicativos personalizados."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 3%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

O uso da API é limitado a fins de desenvolvimento. A API é fornecida como um contexto ao desenvolver aplicativos personalizados. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] O usa a API para transmitir as informações de processamento a um aplicativo personalizado. Para obter mais informações, consulte [Usar microsserviços de ativos e perfis de processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=pt-BR).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com [!DNL Experience Manager] as a [!DNL Cloud Service].

Qualquer cliente do [!DNL Asset Compute Service] A API HTTP deve seguir esse fluxo de alto nível:

1. Um cliente é provisionado como [!DNL Adobe Developer Console] em uma organização IMS. Cada cliente separado (sistema ou ambiente) requer seu próprio projeto separado para separar o fluxo de dados do evento.

1. Um cliente gera um token de acesso para a conta técnica usando o [Autenticação JWT (conta de serviço)](https://www.adobe.io/authentication/auth-methods.html).

1. Um cliente chama [`/register`](#register) recupere o URL do diário apenas uma vez.

1. Um cliente chama [`/process`](#process-request) para cada ativo para o qual deseja gerar representações. A chamada é assíncrona.

1. Um cliente faz polling regular do journal para [receber eventos](#asynchronous-events). Ele recebe eventos para cada representação solicitada quando ela é processada com êxito (`rendition_created` tipo de evento) ou se houver um erro (`rendition_failed` tipo de evento).

A variável [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) O módulo facilita o uso da API no código Node.js.

## Autenticação e autorização {#authentication-and-authorization}

Todas as APIs exigem autenticação de token de acesso. As solicitações devem definir os seguintes cabeçalhos:

1. `Authorization` cabeçalho com token de portador, que é o token de conta técnica, recebido via [Troca de JWT](https://www.adobe.io/authentication/auth-methods.html) do projeto do Adobe Developer Console. A variável [escopos](#scopes) estão documentados abaixo.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` cabeçalho com a ID da organização IMS.

1. `x-api-key` com a ID do cliente do [!DNL Adobe Developers Console] projeto.

### Escopos {#scopes}

Verifique os seguintes escopos para o token de acesso:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Estes requisitos exigem a [!DNL Adobe Developer Console] projeto que deve ser assinado `Asset Compute`, `I/O Events`, e `I/O Management API` serviços. O detalhamento de escopos individuais é:

* Básico
   * escopos: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * escopos: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Eventos
   * metascope: `event_receiver_api`
   * escopos: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] API de gerenciamento
   * metascope: `ent_adobeio_sdk`
   * escopos: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registro {#register}

Cada cliente do [!DNL Asset Compute service] - um único [!DNL Adobe Developer Console] projeto inscrito no serviço - deve [registrar](#register-request) antes de fazer solicitações de processamento. A etapa de registro retorna o diário de eventos exclusivo, que é necessário para recuperar os eventos assíncronos do processamento de representação.

No final de seu ciclo de vida, um cliente pode [cancelar registro](#unregister-request).

### Registrar solicitação {#register-request}

Esta chamada de API configura um [!DNL Asset Compute] e fornece o URL do diário de eventos. Esta é uma operação idempotente e só precisa ser chamada uma vez para cada cliente. Ele pode ser chamado novamente para recuperar o URL do diário.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho  | `/register` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Deve estar vazio. |

### Registrar resposta {#register-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Igual ao `X-Request-Id` cabeçalho de solicitação ou um cabeçalho gerado exclusivamente. Use para identificar solicitações entre sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com `journal`, `ok` e/ou `requestId` campos. |

Os códigos de status HTTP são:

* **200 Êxito**: quando a solicitação é bem-sucedida. Contém a `journal` O URL que deve ser notificado sobre qualquer resultado do processamento assíncrono acionado via `/process` (como tipo de eventos `rendition_created` quando bem-sucedido, ou `rendition_failed` ao falhar).

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Não autorizado**: ocorre quando a solicitação não tem um válido [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem um válido [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Adobe Developer Console (conta técnica) não é assinado por todos os serviços necessários.

* **429 Muitas solicitações**: ocorre quando o sistema é sobrecarregado por esse cliente ou por outra razão. Os clientes devem tentar novamente com um [retrocesso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro 4xx**: quando ocorria qualquer outro erro de cliente e o registro falhava. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Erro 5xx**: ocorre quando ocorreu qualquer outro erro no lado do servidor e o registro falhou. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Cancelar registro da solicitação {#unregister-request}

Essa chamada de API cancela o registro de um [!DNL Asset Compute] cliente. Depois disso, não será mais possível chamar `/process`. O uso da chamada de API para um cliente não registrado ou para um cliente a ser registrado retorna um `404` erro.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho  | `/unregister` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Vazio. |

### Cancelar registro da resposta {#unregister-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Igual ao `X-Request-Id` cabeçalho de solicitação ou um cabeçalho gerado exclusivamente. Use para identificar solicitações entre sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com `ok` e `requestId` campos. |

Os códigos de status são:

* **200 Êxito**: ocorre quando o registro e o journal são encontrados e removidos.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Não autorizado**: ocorre quando a solicitação não tem um válido [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem um válido [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Adobe Developer Console (conta técnica) não é assinado por todos os serviços necessários.

* **404 Não encontrado**: ocorre quando não há registro atual para as credenciais fornecidas.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Muitas solicitações**: ocorre quando o sistema está sobrecarregado. Os clientes devem tentar novamente com um [retrocesso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.

* **Erro 4xx**: ocorre quando ocorreu qualquer outro erro de cliente e o cancelamento de registro falhou. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Erro 5xx**: ocorre quando ocorreu qualquer outro erro no lado do servidor e o registro falhou. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Processar solicitação {#process-request}

A variável `process` A operação envia um trabalho que transforma um ativo de origem em várias representações, com base nas instruções na solicitação. Notificações sobre conclusão com êxito (tipo de evento) `rendition_created`) ou qualquer erro (tipo de evento `rendition_failed`) são enviados para um Diário de eventos que deve ser recuperado usando [/register](#register) uma vez antes de efetuar qualquer número de `/process` solicitações. Solicitações formadas incorretamente falham imediatamente com um código de erro 400.

Os binários são referenciados usando URLs, como URLs pré-assinados do Amazon AWS S3 ou URLs SAS do Armazenamento Azure Blob, para ler o `source` ativo (`GET` URLs) e gravação das representações (`PUT` URLs) O cliente é responsável por gerar esses URLs pré-assinados.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho  | `/process` |
| Tipo MIME | `application/json` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Deve estar no formato JSON da solicitação do processo, conforme descrito abaixo. Ele fornece instruções sobre qual ativo processar e quais representações gerar. |

### Processar solicitação JSON {#process-request-json}

O corpo da solicitação de `/process` é um objeto JSON com este esquema de alto nível:

```json
{
    "source": "",
    "renditions" : []
}
```

Os campos disponíveis são:

| Nome | Tipo | Descrição | Exemplo |
|--------------|----------|-------------|---------|
| `source` | `string` | URL do ativo de origem a ser processado. Opcional com base no formato de representação solicitado (por exemplo, `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descrição do ativo de origem a ser processado. Consulte a descrição de [Campos do objeto de origem](#source-object-fields) abaixo. Opcional com base no formato de representação solicitado (por exemplo, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representações a serem geradas a partir do arquivo de origem. Cada objeto de representação suporta [instrução de representação](#rendition-instructions). Obrigatório. | `[{ "target": "https://....", "fmt": "png" }]` |

A variável `source` pode ser um `<string>` que é visto como um URL ou pode ser um `<object>` com um campo adicional. As seguintes variantes são semelhantes:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campos do objeto de origem {#source-object-fields}

| Nome | Tipo | Descrição | Exemplo |
|-----------|----------|-------------|---------|
| `url` | `string` | URL do ativo de origem a ser processado. Obrigatório. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nome do arquivo do ativo de origem. A extensão de arquivo no nome pode ser usada se nenhum tipo MIME for detectado. Tem prioridade sobre o nome de arquivo no caminho do URL ou nome de arquivo em `content-disposition` cabeçalho do recurso binário. O padrão é &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Tamanho do arquivo de ativo de origem em bytes. Tem precedência sobre `content-length` cabeçalho do recurso binário. | `10234` |
| `mimetype` | `string` | Tipo MIME do arquivo de ativo de origem. Tem precedência sobre o `content-type` cabeçalho do recurso binário. | `"image/jpeg"` |

### Um `process` exemplo de solicitação {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Processar resposta {#process-response}

A variável `/process` A solicitação do retorna imediatamente com êxito ou falha com base na validação básica da solicitação. O processamento de ativos real ocorre de forma assíncrona.

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Igual ao `X-Request-Id` cabeçalho de solicitação ou um cabeçalho gerado exclusivamente. Use para identificar solicitações entre sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com `ok` e `requestId` campos. |

Códigos de status:

* **200 Êxito**: se a solicitação foi enviada com êxito. A resposta JSON inclui `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Solicitação inválida**: se a solicitação estiver formada incorretamente, como campos obrigatórios ausentes no JSON da solicitação. A resposta JSON inclui `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Não autorizado**: quando a solicitação não tiver uma conta válida [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.
* **403 Proibido**: quando a solicitação não tiver uma conta válida [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Adobe Developer Console (conta técnica) não é assinado por todos os serviços necessários.
* **429 Muitas solicitações**: quando o sistema é sobrecarregado por esse cliente ou, em geral, Os clientes podem tentar novamente com um [retrocesso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro 4xx**: quando ocorria qualquer outro erro de cliente. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Erro 5xx**: quando havia qualquer outro erro do lado do servidor. Normalmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

Provavelmente, a maioria dos clientes está inclinada a repetir exatamente a mesma solicitação com [retrocesso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) em qualquer erro *exceto* problemas de configuração como 401 ou 403, ou solicitações inválidas como 400. Além da limitação de taxa regular por meio de respostas 429, uma interrupção ou limitação temporária de serviço pode resultar em erros 5xx. Em seguida, seria aconselhável tentar novamente após um período.

Todas as respostas JSON (se presentes) incluem o `requestId` que tem o mesmo valor que a variável `X-Request-Id` cabeçalho. É recomendável ler a partir do cabeçalho, pois ele está sempre presente. A variável `requestId` também é retornado em todos os eventos relacionados às solicitações de processamento como `requestId`. Os clientes não devem fazer qualquer suposição sobre o formato dessa cadeia de caracteres. Ele é um identificador de cadeia de caracteres opaco.

## Aceitar o pós-processamento {#opt-in-to-post-processing}

A variável [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) O oferece suporte a um conjunto de opções básicas de pós-processamento de imagem. Os trabalhadores personalizados podem aceitar explicitamente o pós-processamento definindo o campo `postProcess` no objeto de representação para `true`.

Os casos de uso compatíveis são:

* Cortar uma representação em um retângulo cujos limites são definidos por crop.w, crop.h, crop.x e crop.y. É definido por `instructions.crop` no objeto de representação.
* Redimensionar imagens usando largura, altura ou ambos. É definido por `instructions.width` e `instructions.height` no objeto de representação. Para redimensionar usando apenas largura ou altura, defina apenas um valor. O serviço de computação conserva a taxa de proporção.
* Defina a qualidade de uma imagem JPEG. É definido por `instructions.quality` no objeto de representação. A melhor qualidade é indicada por `100` e valores menores indicam qualidade reduzida.
* Criar imagens entrelaçadas. É definido por `instructions.interlace` no objeto de representação.
* Defina DPI para ajustar o tamanho renderizado para fins de publicação em desktop, ajustando a escala aplicada aos pixels. É definido por `instructions.dpi` no objeto de representação para alterar a resolução da dpi. No entanto, para redimensionar a imagem para que fique com o mesmo tamanho em uma resolução diferente, use o `convertToDpi` instruções.
* Redimensionar a imagem de forma que sua largura ou altura renderizada permaneça a mesma que a original na resolução de destino especificada (DPI). É definido por `instructions.convertToDpi` no objeto de representação.

## Inserir marca d&#39;água em ativos {#add-watermark}

A variável [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) O suporta a adição de uma marca d&#39;água aos arquivos de imagem PNG, JPEG, TIFF e GIF. A marca d&#39;água é adicionada seguindo as instruções de representação na `watermark` objeto na representação.

A marca d&#39;água é feita durante o pós-processamento da representação. Para criar uma marca d&#39;água nos ativos, o trabalhador personalizado [opta pelo pós-processamento](#opt-in-to-post-processing) definindo o campo `postProcess` no objeto de representação para `true`. Se o trabalhador não aceitar, a marca d&#39;água não será aplicada, mesmo que o objeto de marca d&#39;água esteja definido no objeto de representação na solicitação.

## Instruções de representação {#rendition-instructions}

Estas são as opções disponíveis para o `renditions` matriz em [/process](#process-request).

### Campos comuns {#common-fields}

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | O formato de destino das representações também pode ser `text` para extração de texto e `xmp` para extrair metadados de XMP como xml. Consulte [formatos compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL de um [aplicativo personalizado](develop-custom-application.md). Deve ser um `https://` URL. Se esse campo estiver presente, a representação será criada por um aplicativo personalizado. Qualquer outro campo de representação definido é usado no aplicativo personalizado. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL para o qual a representação gerada deve ser carregada usando o PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Informações de upload de URL pré-assinado de várias partes para a representação gerada. Este é para [Upload binário direto do AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) com este [comportamento de upload de várias partes](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Campos:<ul><li>`urls`: matriz de cadeias de caracteres, uma para cada URL de parte pré-assinado</li><li>`minPartSize`: o tamanho mínimo a ser usado para uma parte = url</li><li>`maxPartSize`: o tamanho máximo a ser usado para uma parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Espaço reservado opcional controlado pelo cliente e transmitido como está para eventos de representação. Permite que os clientes adicionem informações personalizadas para identificar eventos de representação. Não deve ser modificado ou confiado em aplicativos personalizados, pois os clientes podem alterar isso a qualquer momento. | `{ ... }` |

### Campos específicos da representação {#rendition-specific-fields}

Para obter uma lista de formatos de arquivo compatíveis no momento, consulte [formatos de arquivo compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `*` | `*` | Campos personalizados e avançados podem ser adicionados que uma [aplicativo personalizado](develop-custom-application.md) O entende. | |
| `embedBinaryLimit` | `number` em bytes | Se esse valor estiver definido e o tamanho do arquivo da representação for menor que esse valor, a representação será incorporada ao evento enviado após a conclusão da geração da representação. O tamanho máximo permitido para a incorporação é de 32 KB (32 x 1024 bytes). Se uma representação for maior que o tamanho da `embedBinaryLimit` limite, ele deve ser colocado em um local no armazenamento na nuvem e não está incorporado no evento. | `3276` |
| `width` | `number` | Largura em pixels. somente para representações de imagem. | `200` |
| `height` | `number` | Altura em pixels. somente para representações de imagem. | `200` |
|                   |          | A proporção será sempre mantida se: <ul> <li> Ambos `width` e `height` forem especificadas, a imagem se ajustará no tamanho, mantendo a proporção </li><li> Somente `width` ou somente `height` for especificada, a imagem resultante usará a dimensão correspondente, mantendo a proporção</li><li> Se nenhuma delas `width` nem `height` for especificado, o tamanho do pixel da imagem original será usado. Depende do tipo de origem. Para alguns formatos, como arquivos PDF, um tamanho padrão é usado. Pode haver um limite de tamanho máximo.</li></ul> | |
| `quality` | `number` | Especificar a qualidade do jpeg no intervalo de `1` para `100`. Aplicável somente para representações de imagem. | `90` |
| `xmp` | `string` | Usado apenas pelo writeback de metadados XMP, é codificado em base64 XMP para gravar de volta na representação especificada. | |
| `interlace` | `bool` | Crie um PNG entrelaçado ou um GIF ou JPEG progressivo definindo-o como `true`. Não tem efeito em outros formatos de arquivo. | |
| `jpegSize` | `number` | Tamanho aproximado do arquivo JPEG em bytes. Ele substitui qualquer `quality` configuração. Não tem efeito em outros formatos. | |
| `dpi` | `number` ou `object` | Defina x e y DPI. Para simplificar, também pode ser definido como um único número que é usado para x e y. Ela não tem efeito na própria imagem. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` ou `object` | x e y DPI reexemplificam os valores mantendo o tamanho físico. Para simplificar, também pode ser definido como um único número que é usado para x e y. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de arquivos a serem incluídos no arquivo ZIP (`fmt=zip`). Cada entrada pode ser uma string de URL ou um objeto com os campos:<ul><li>`url`: URL para baixar o arquivo</li><li>`path`: armazene o arquivo nesse caminho no ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Tratamento de duplicatas para arquivos ZIP (`fmt=zip`). Por padrão, vários arquivos armazenados no mesmo caminho no ZIP geram um erro. Configuração `duplicate` para `ignore` resulta no armazenamento somente do primeiro ativo e o restante é ignorado. | `ignore` |
| `watermark` | `object` | Contém instruções sobre o [marca d&#39;água](#watermark-specific-fields). |  |

### Campos específicos de marca d&#39;água {#watermark-specific-fields}

O formato PNG é usado como marca d&#39;água.

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala da marca d&#39;água, entre `0.0` e `1.0`. `1.0` significa que a marca d&#39;água tem sua escala original (1:1) e os valores mais baixos reduzem o tamanho da marca d&#39;água. | Um valor de `0.5` significa metade do tamanho original. |
| `image` | `url` | URL do arquivo PNG a ser usado para marca d&#39;água. | |

## Eventos assíncronos {#asynchronous-events}

Quando o processamento de uma representação for concluído ou quando ocorrer um erro, um evento será enviado para um [[!DNL Adobe I/O] Diário de eventos](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Os clientes devem escutar o URL do diário fornecido por meio de [/register](#register). A resposta do diário inclui uma `event` matriz que consiste em um objeto para cada evento, do qual a variável `event` inclui a carga útil real do evento.

A variável [!DNL Adobe I/O] Tipo de evento para todos os eventos do [!DNL Asset Compute Service] é `asset_compute`. O journal é automaticamente inscrito somente neste tipo de evento e não há mais necessidade de filtrar com base no [!DNL Adobe I/O] Tipo de evento. Os tipos de evento específicos do serviço estão disponíveis no `type` propriedade do evento.

### Tipos de evento {#event-types}

| Evento | Descrição |
|---------------------|-------------|
| `rendition_created` | Enviado para cada representação processada e carregada com sucesso. |
| `rendition_failed` | Enviado para cada representação que falhou no processamento ou upload. |

### Atributos do evento {#event-attributes}

| Atributo | Tipo | Evento | Descrição |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Carimbo de data e hora quando o evento foi enviado em formato estendido simplificado [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) conforme definido pelo JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | A ID da solicitação original para `/process`, igual a `X-Request-Id` cabeçalho. |
| `source` | `object` | `*` | A variável `source` do `/process` solicitação. |
| `userData` | `object` | `*` | A variável `userData` da representação da `/process` solicitação se definida. |
| `rendition` | `object` | `rendition_*` | O objeto de representação correspondente transmitido `/process`. |
| `metadata` | `object` | `rendition_created` | A variável [metadados](#metadata) propriedades da representação. |
| `errorReason` | `string` | `rendition_failed` | Falha de representação [motivo](#error-reasons) se houver. |
| `errorMessage` | `string` | `rendition_failed` | Texto que fornece mais detalhes sobre a falha de representação, se houver. |

### Metadados {#metadata}

| Propriedade | Descrição |
|--------|-------------|
| `repo:size` | O tamanho da representação em bytes. |
| `repo:sha1` | O resumo sha1 da representação. |
| `dc:format` | O tipo MIME da representação. |
| `repo:encoding` | A codificação de conjunto de caracteres da representação, caso seja um formato baseado em texto. |
| `tiff:ImageWidth` | A largura da representação em pixels. Presente somente para representações de imagem. |
| `tiff:ImageLength` | O comprimento da representação em pixels. Presente somente para representações de imagem. |

### Motivos de erro {#error-reasons}

| Motivo | Descrição |
|---------|-------------|
| `RenditionFormatUnsupported` | O formato de representação solicitado não tem suporte para a origem fornecida. |
| `SourceUnsupported` | A origem específica não é compatível, mesmo que o tipo seja. |
| `SourceCorrupt` | Os dados de origem estão corrompidos. Inclui arquivos vazios. |
| `RenditionTooLarge` | Não foi possível fazer upload da representação usando os URLs pré-assinados fornecidos em `target`. O tamanho real da representação está disponível como metadados no `repo:size` e podem ser usados pelo cliente para processar novamente essa representação com o número correto de URLs pré-assinados. |
| `GenericError` | Qualquer outro erro inesperado. |
