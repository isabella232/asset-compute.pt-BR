---
title: '"[!DNL Asset Compute Service] API HTTP"'
description: '"[!DNL Asset Compute Service] API HTTP para criar aplicativos personalizados."'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

O uso da API é limitado a propósitos de desenvolvimento. A API é fornecida como um contexto ao desenvolver aplicativos personalizados. [!DNL Adobe Experience Manager] como [!DNL Cloud Service] O usa a API para transmitir as informações de processamento para um aplicativo personalizado. Para obter mais informações, consulte [Usar microsserviços de ativos e perfis de processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com [!DNL Experience Manager] como [!DNL Cloud Service].

Qualquer cliente da [!DNL Asset Compute Service] A API HTTP deve seguir esse fluxo de alto nível:

1. Um cliente é provisionado como [!DNL Adobe Developer Console] projeto em uma organização IMS. Cada cliente separado (sistema ou ambiente) requer seu próprio projeto separado para separar o fluxo de dados do evento.

1. Um cliente gera um token de acesso para a conta técnica usando o [Autenticação JWT (Conta de Serviço)](https://www.adobe.io/authentication/auth-methods.html).

1. Um cliente chama [`/register`](#register) somente uma vez para recuperar o URL do diário.

1. Um cliente chama [`/process`](#process-request) para cada ativo para o qual deseja gerar representações. A chamada é assíncrona.

1. Um cliente pesquisa regularmente o diário para [receber eventos](#asynchronous-events). Ele recebe eventos para cada renderização solicitada quando a renderização é processada com êxito (`rendition_created` tipo de evento) ou se houver um erro (`rendition_failed` tipo de evento).

O [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) O módulo facilita o uso da API no código Node.js.

## Autenticação e autorização {#authentication-and-authorization}

Todas as APIs exigem autenticação de token de acesso. As solicitações devem definir os seguintes cabeçalhos:

1. `Authorization` cabeçalho com token portador, que é o token de conta técnica, recebido por meio de [Troca de JWT](https://www.adobe.io/authentication/auth-methods.html) do projeto do Console do desenvolvedor do Adobe. O [escopos](#scopes) estão documentadas abaixo.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` com a ID da organização IMS.

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

Isso exige que [!DNL Adobe Developer Console] projeto a ser inscrito no `Asset Compute`, `I/O Events`e `I/O Management API` serviços. O detalhamento de escopos individuais é:

* Básico
   * escopos: `openid,AdobeID`

* asset compute
   * metascópio: `asset_compute_meta`
   * escopos: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Eventos
   * metascópio: `event_receiver_api`
   * escopos: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] API de gerenciamento
   * metascópio: `ent_adobeio_sdk`
   * escopos: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registro {#register}

Cada cliente do [!DNL Asset Compute service] - um [!DNL Adobe Developer Console] projeto subscrito no serviço - deve [registrar](#register-request) antes de fazer solicitações de processamento. A etapa de registro retorna o diário de eventos exclusivo, que é necessário para recuperar os eventos assíncronos do processamento de representação.

No final de seu ciclo de vida, um cliente pode [cancelar registro](#unregister-request).

### Solicitação de registro {#register-request}

Essa chamada de API define um [!DNL Asset Compute] e fornece o URL do diário de eventos. Esta é uma operação idempotente e só precisa ser chamada uma vez para cada cliente. Ele pode ser chamado novamente para recuperar o URL do diário.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/register` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Deve estar vazio. |

### Registrar resposta {#register-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Ou o mesmo que a variável `X-Request-Id` cabeçalho de solicitação ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo de resposta | Um objeto JSON com `journal`, `ok` e/ou `requestId` campos. |

Os códigos do status HTTP são:

* **Sucesso 200**: Quando a solicitação é bem-sucedida. Ele contém a variável `journal` URL que é notificado sobre quaisquer resultados do processamento assíncrono acionados por meio de `/process` (como tipo de evento `rendition_created` quando bem-sucedido, ou `rendition_failed` quando falhar).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Não Autorizado**: ocorre quando a solicitação não tem um valor válido [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem um valor válido [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não é inscrito em todos os serviços necessários.

* **429 Demasiados pedidos**: ocorre quando o sistema é sobrecarregado por este cliente ou de outra forma. Os clientes devem tentar novamente com um [recuo exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro 4xx**: Quando havia outro erro de cliente e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro 5xx**: ocorre quando houve qualquer outro erro do lado do servidor e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Solicitação de cancelamento de registro {#unregister-request}

Essa chamada de API cancela o registro de uma [!DNL Asset Compute] cliente. Depois disso, não é mais possível chamar `/process`. Usar a chamada de API para um cliente não registrado ou um cliente ainda não registrado retorna um `404` erro.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/unregister` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Vazio. |

### Cancelar registro da resposta {#unregister-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Ou o mesmo que a variável `X-Request-Id` cabeçalho de solicitação ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo de resposta | Um objeto JSON com `ok` e `requestId` campos. |

Os códigos de status são:

* **Sucesso 200**: ocorre quando o registro e o diário são encontrados e removidos.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Não Autorizado**: ocorre quando a solicitação não tem um valor válido [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem um valor válido [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não é inscrito em todos os serviços necessários.

* **404 Não encontrado**: ocorre quando não há registro atual para as credenciais fornecidas.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Demasiados pedidos**: ocorre quando o sistema é sobrecarregado. Os clientes devem tentar novamente com um [recuo exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.

* **Erro 4xx**: ocorre quando houve outro erro de cliente e o cancelamento de registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro 5xx**: ocorre quando houve qualquer outro erro do lado do servidor e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Solicitação de processo {#process-request}

O `process` A operação envia uma tarefa que transforma um ativo de origem em várias representações, com base nas instruções na solicitação. Notificações sobre conclusão com êxito (tipo de evento) `rendition_created`) ou qualquer erro (tipo de evento) `rendition_failed`) são enviadas para um diário de Eventos que deve ser recuperado usando [/registro](#register) uma vez antes de fazer qualquer número de `/process` solicitações. As solicitações formadas incorretamente falham imediatamente com um código de erro 400.

Os binários são referenciados usando URLs, como URLs pré-assinados do Amazon AWS S3 ou URLs SAS do Armazenamento Azure Blob, para leitura da variável `source` ativo (`GET` URLs) e gravando as representações (`PUT` URLs). O cliente é responsável pela geração desses URLs pré-assinados.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/process` |
| Tipo MIME | `application/json` |
| Cabeçalho `Authorization` | Todos [cabeçalhos relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento em todos os sistemas. |
| Corpo da solicitação | Deve estar no formato JSON de solicitação de processo, conforme descrito abaixo. Ele fornece instruções sobre qual ativo processar e quais execuções gerar. |

### JSON de solicitação de processo {#process-request-json}

O corpo da solicitação de `/process` é um objeto JSON com esse esquema de alto nível:

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
| `source` | `object` | Descrição do ativo de origem a ser processado. Consulte a descrição de [Campos de objeto de origem](#source-object-fields) abaixo. Opcional com base no formato de representação solicitado (por exemplo, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representações a serem geradas a partir do arquivo de origem. Cada objeto de representação suporta [instrução de representação](#rendition-instructions). Obrigatório. | `[{ "target": "https://....", "fmt": "png" }]` |

O `source` pode ser um `<string>` que é visto como um URL ou pode ser um `<object>` com um campo adicional. As seguintes variantes são semelhantes:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campos de objeto de origem {#source-object-fields}

| Nome | Tipo | Descrição | Exemplo |
|-----------|----------|-------------|---------|
| `url` | `string` | URL do ativo de origem a ser processado. Obrigatório. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nome do arquivo de ativo de origem. A extensão de arquivo no nome pode ser usada se nenhum tipo MIME puder ser detectado. Tem precedência sobre o nome do arquivo no caminho do URL ou no nome do arquivo em `content-disposition` cabeçalho do recurso binário. O padrão é &quot;arquivo&quot;. | `"image.jpg"` |
| `size` | `number` | Tamanho do arquivo de ativo de origem em bytes. Tem precedência sobre `content-length` cabeçalho do recurso binário. | `10234` |
| `mimetype` | `string` | Tipo MIME do arquivo de ativo de origem. Tem precedência sobre a `content-type` cabeçalho do recurso binário. | `"image/jpeg"` |

### Uma conclusão `process` exemplo de solicitação {#complete-process-request-example}

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

## Resposta do processo {#process-response}

O `/process` A solicitação retorna imediatamente com um sucesso ou uma falha com base na validação básica da solicitação. O processamento de ativos reais ocorre de forma assíncrona.

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | Ou o mesmo que a variável `X-Request-Id` cabeçalho de solicitação ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo de resposta | Um objeto JSON com `ok` e `requestId` campos. |

Códigos de status:

* **Sucesso 200**: Se a solicitação foi enviada com êxito. GSON de resposta `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Solicitação inválida**: Se a solicitação estiver formada incorretamente, como campos obrigatórios ausentes no JSON da solicitação. GSON de resposta `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Não Autorizado**: Quando a solicitação não tiver um valor válido [autenticação](#authentication-and-authorization). Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.
* **403 Proibido**: Quando a solicitação não tiver um valor válido [autorização](#authentication-and-authorization). Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não é inscrito em todos os serviços necessários.
* **429 Demasiados pedidos**: Quando o sistema é sobrecarregado por este cliente ou em geral. Os clientes podem tentar novamente com um [recuo exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro 4xx**: Quando havia outro erro de cliente. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro 5xx**: Quando havia outro erro no lado do servidor. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

A maioria dos clientes provavelmente tem tendência para repetir exatamente a mesma solicitação com [recuo exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) em qualquer erro *Except* problemas de configuração, como 401 ou 403, ou solicitações inválidas, como 400. Além da limitação regular da taxa por meio de respostas 429, uma interrupção ou limitação temporária do serviço pode resultar em erros 5xx. Seria então aconselhável tentar de novo após um período de tempo.

Todas as respostas JSON (se presentes) incluem a variável `requestId` que tem o mesmo valor que `X-Request-Id` cabeçalho. É recomendável ler a partir do cabeçalho, já que ele está sempre presente. O `requestId` também é retornado em todos os eventos relacionados ao processamento de solicitações como `requestId`. Os clientes não devem assumir nenhuma suposição sobre o formato dessa string, ele é um identificador de string opaco.

## Aceitar pós-processamento {#opt-in-to-post-processing}

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk) O suporta um conjunto de opções básicas de pós-processamento de imagem. Os trabalhadores personalizados podem aderir explicitamente ao pós-processamento, definindo o campo `postProcess` no objeto de representação para `true`.

Os casos de uso compatíveis são:

* Recorte uma representação em um retângulo cujos limites são definidos por cortar.w, cortar.h, cortar.x e cortar.y. É definido por `instructions.crop` no objeto de representação.
* Redimensione as imagens usando largura, altura ou ambas. É definido por `instructions.width` e `instructions.height` no objeto de representação. Para redimensionar usando somente largura ou altura, defina apenas um valor. O Serviço de Computação conserva a taxa de proporção.
* Defina a qualidade de uma imagem JPEG. É definido por `instructions.quality` no objeto de representação. A melhor qualidade é indicada por `100` e valores menores indicam qualidade reduzida.
* Crie imagens entrelaçadas. É definido por `instructions.interlace` no objeto de representação.
* Defina o DPI para ajustar o tamanho renderizado para fins de publicação de desktop ajustando a escala aplicada aos pixels. É definido por `instructions.dpi` no objeto de representação para alterar a resolução da dpi. No entanto, para redimensionar a imagem de modo que ela tenha o mesmo tamanho em uma resolução diferente, use a opção `convertToDpi` instruções.
* Redimensione a imagem de forma que sua largura ou altura renderizada permaneça a mesma do original na DPI (resolução de destino especificada). É definido por `instructions.convertToDpi` no objeto de representação.

## Inserir marca d&#39;água em ativos {#add-watermark}

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk) O suporta a adição de uma marca d&#39;água aos arquivos PNG, JPEG, TIFF e GIF image. A marca d&#39;água é adicionada seguindo as instruções de representação na `watermark` na representação.

A marca d&#39;água é feita durante o pós-processamento da representação. Para adicionar ativos de marca d&#39;água, o trabalhador personalizado [opts no pós-processamento](#opt-in-to-post-processing) definindo o campo `postProcess` no objeto de representação para `true`. Se o trabalhador não aceitar, a marca d&#39;água não será aplicada, mesmo que o objeto da marca d&#39;água esteja definido no objeto de representação na solicitação.

## Instruções de representação {#rendition-instructions}

Essas são as opções disponíveis para a variável `renditions` array em [/process](#process-request).

### Campos comuns {#common-fields}

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | O formato de destino das representações também pode ser `text` para extração de texto e `xmp` para extrair metadados de XMP como xml. Consulte [formatos compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL de um [aplicativo personalizado](develop-custom-application.md). Deve ser um `https://` URL. Se esse campo estiver presente, a representação será criada por um aplicativo personalizado. Qualquer outro campo de representação definido é usado no aplicativo personalizado. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL para o qual a representação gerada deve ser carregada usando HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Multiplica informações de upload de URL pré-assinado para a representação gerada. Isso é para [Upload binário direto AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) com [comportamento de upload de várias partes](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fields:<ul><li>`urls`: matriz de strings, uma para cada URL de parte pré-assinada</li><li>`minPartSize`: o tamanho mínimo a ser usado para uma parte = url</li><li>`maxPartSize`: o tamanho máximo a ser usado para uma parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Espaço reservado opcional controlado pelo cliente e passado como está para eventos de representação. Permite que os clientes adicionem informações personalizadas para identificar eventos de representação. Não deve ser modificado ou confiável em aplicativos personalizados, pois os clientes podem alterá-lo a qualquer momento. | `{ ... }` |

### Campos específicos da representação {#rendition-specific-fields}

Para obter uma lista de formatos de arquivo compatíveis no momento, consulte [formatos de arquivo compatíveis](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `*` | `*` | Campos avançados e personalizados podem ser adicionados de forma que uma [aplicativo personalizado](develop-custom-application.md) entende. |  |
| `embedBinaryLimit` | `number` em bytes | Se esse valor for definido e o tamanho do arquivo da representação for menor que esse valor, a representação será incorporada no evento enviado após a conclusão da geração da representação. O tamanho máximo permitido para incorporação é 32 KB (32 x 1024 bytes). Se uma representação tiver um tamanho maior que o `embedBinaryLimit` limite, ele é colocado em um local no armazenamento em nuvem e não é incorporado no evento. | `3276` |
| `width` | `number` | Largura em pixels. somente para representações de imagem. | `200` |
| `height` | `number` | Altura em pixels. somente para representações de imagem. | `200` |
|  |  | A taxa de proporção é sempre mantida se: <ul> <li> Ambos `width` e `height` são especificadas, em seguida, a imagem se ajusta ao tamanho, mantendo a proporção </li><li> Somente `width` ou somente `height` for especificada, a imagem resultante usará a dimensão correspondente, mantendo a proporção</li><li> Se nem `width` nor `height` for especificado, o tamanho do pixel da imagem original será usado. Depende do tipo de origem. Para alguns formatos, como arquivos PDF, um tamanho padrão é usado. Pode haver um limite máximo de tamanho.</li></ul> |  |
| `quality` | `number` | Especificar a qualidade do jpeg no intervalo de `1` para `100`. Aplicável somente para representações de imagem. | `90` |
| `xmp` | `string` | Usado apenas XMP write-back de metadados, é XMP codificado em base64 para gravar de volta na representação especificada. |  |
| `interlace` | `bool` | Crie PNG ou GIF ou JPEG progressivo entrelaçado ao configurá-lo como `true`. Não tem efeito sobre outros formatos de arquivo. |  |
| `jpegSize` | `number` | Tamanho aproximado do arquivo JPEG em bytes. Substitui qualquer `quality` configuração. Não afeta outros formatos. |  |
| `dpi` | `number` ou `object` | Defina x e y DPI. Para simplificar, também pode ser definido para um único número que é usado para x e y. Não tem efeito na própria imagem. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` ou `object` | x e y DPI refazem valores, mantendo o tamanho físico. Para simplificar, também pode ser definido para um único número que é usado para x e y. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de arquivos a serem incluídos no arquivo ZIP (`fmt=zip`). Cada entrada pode ser uma string de URL ou um objeto com os campos:<ul><li>`url`: URL para baixar arquivo</li><li>`path`: Armazenar arquivo sob este caminho no ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Manuseio de duplicatas para arquivos ZIP (`fmt=zip`). Por padrão, vários arquivos armazenados no mesmo caminho no ZIP gera um erro. Configuração `duplicate` para `ignore` resulta em apenas o primeiro ativo a ser armazenado e o restante a ser ignorado. | `ignore` |
| `watermark` | `object` | Contém instruções sobre o [marca d&#39;água](#watermark-specific-fields). |  |

### Campos específicos de marca d&#39;água {#watermark-specific-fields}

O formato PNG é usado como uma marca d&#39;água.

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala da marca de água, entre `0.0` e `1.0`. `1.0` significa que a marca de água tem a escala original (1:1) e os valores inferiores reduzem o tamanho da marca de água. | Um valor de `0.5` significa metade do tamanho original. |
| `image` | `url` | URL do arquivo PNG a ser usado para marca d&#39;água. |  |

## Eventos assíncronos {#asynchronous-events}

Quando o processamento de uma representação é concluído ou um erro ocorre, um evento é enviado para um [[!DNL Adobe I/O] Diário de Eventos](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Os clientes devem ouvir o URL do diário fornecido por meio de [/registro](#register). A resposta do diário inclui um `event` matriz que consiste em um objeto para cada evento, sendo que `event` inclui a carga real do evento.

O [!DNL Adobe I/O] Tipo de evento para todos os eventos do [!DNL Asset Compute Service] é `asset_compute`. O diário é automaticamente inscrito somente nesse tipo de evento e não há nenhum outro requisito para filtrar com base no [!DNL Adobe I/O] Tipo de evento. Os tipos de evento específicos de serviço estão disponíveis na variável `type` propriedade do evento.

### Tipos de evento {#event-types}

| Evento | Descrição |
|---------------------|-------------|
| `rendition_created` | Enviado para cada renderização processada e carregada com êxito. |
| `rendition_failed` | Enviado para cada representação que não processou ou fez upload. |

### Atributos do evento {#event-attributes}

| Atributo | Tipo | Evento | Descrição |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Carimbo de data e hora quando o evento foi enviado com extensão simplificada [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , conforme definido pelo JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | O ID da solicitação original para `/process`, o mesmo que `X-Request-Id` cabeçalho. |
| `source` | `object` | `*` | O `source` do `/process` solicitação. |
| `userData` | `object` | `*` | O `userData` da representação da `/process` solicitação, se definida. |
| `rendition` | `object` | `rendition_*` | O objeto de representação correspondente transmitido em `/process`. |
| `metadata` | `object` | `rendition_created` | O [metadados](#metadata) propriedades da representação. |
| `errorReason` | `string` | `rendition_failed` | Falha na representação [reason](#error-reasons) se houver. |
| `errorMessage` | `string` | `rendition_failed` | Texto que fornece mais detalhes sobre a falha de representação, se houver. |

### Metadados {#metadata}

| Propriedade | Descrição |
|--------|-------------|
| `repo:size` | O tamanho da representação em bytes. |
| `repo:sha1` | O resumo sha1 da representação. |
| `dc:format` | O tipo MIME da representação. |
| `repo:encoding` | A codificação charset da representação, caso seja um formato baseado em texto. |
| `tiff:ImageWidth` | A largura da representação em pixels. Apresentado apenas para representações de imagem. |
| `tiff:ImageLength` | O comprimento da representação em pixels. Apresentado apenas para representações de imagem. |

### Motivos de erro {#error-reasons}

| Motivo | Descrição |
|---------|-------------|
| `RenditionFormatUnsupported` | Não há suporte para o formato de representação solicitado para a fonte em questão. |
| `SourceUnsupported` | A origem específica não é suportada mesmo que o tipo seja suportado. |
| `SourceCorrupt` | Os dados de origem estão corrompidos. Inclui arquivos vazios. |
| `RenditionTooLarge` | A representação não pôde ser carregada usando os URLs pré-assinados fornecidos em `target`. O tamanho da representação real está disponível como metadados em `repo:size` e podem ser usados pelo cliente para reprocessar essa representação com o número correto de URLs pré-assinados. |
| `GenericError` | Qualquer outro erro inesperado. |
