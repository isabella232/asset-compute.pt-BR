---
title: '[!DNL Asset Compute Service] API HTTP.'
description: '[!DNL Asset Compute Service] API HTTP para criar aplicativos personalizados.'
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---


# [!DNL Asset Compute Service] API HTTP  {#asset-compute-http-api}

O uso da API é limitado a fins de desenvolvimento. A API é fornecida como um contexto ao desenvolver aplicativos personalizados. [!DNL Adobe Experience Manager] como um  [!DNL Cloud Service] usa a API para passar as informações de processamento para um aplicativo personalizado. Para obter mais informações, consulte [Usar microserviços de ativos e Perfis de processamento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] está disponível somente para uso com  [!DNL Experience Manager] como  [!DNL Cloud Service].

Qualquer cliente da API HTTP [!DNL Asset Compute Service] deve seguir esse fluxo de alto nível:

1. Um cliente é provisionado como [!DNL Adobe Developer Console] projeto em uma organização IMS. Cada cliente separado (sistema ou ambiente) exige seu próprio projeto separado para separar o fluxo de dados do evento.

1. Um cliente gera um token de acesso para a conta técnica usando a [Autenticação JWT (Conta de Serviço)](https://www.adobe.io/authentication/auth-methods.html).

1. Um cliente chama [`/register`](#register) apenas uma vez para recuperar o URL do journal.

1. Um cliente chama [`/process`](#process-request) para cada ativo para o qual deseja gerar representações. A chamada é assíncrona.

1. Um cliente consulta regularmente o journal para [receber eventos](#asynchronous-events). Ele recebe eventos para cada representação solicitada quando a representação é processada com êxito (`rendition_created` tipo de evento) ou se há um erro (`rendition_failed` tipo de evento).

O módulo [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) facilita o uso da API no código Node.js.

## Autenticação e autorização {#authentication-and-authorization}

Todas as APIs exigem autenticação de token de acesso. As solicitações devem definir os seguintes cabeçalhos:

1. `Authorization` cabeçalho com token do portador, que é o token da conta técnica, recebido por meio do  [JWT ](https://www.adobe.io/authentication/auth-methods.html) Exchange do projeto do Console do desenvolvedor do Adobe. Os escopos [](#scopes) estão documentados abaixo.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` com a ID da organização IMS.

1. `x-api-key` com a ID do cliente do  [!DNL Adobe Developers Console] projeto.

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

Eles exigem que o projeto [!DNL Adobe Developer Console] seja inscrito nos serviços `Asset Compute`, `I/O Events` e `I/O Management API`. A repartição dos âmbitos individuais é:

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

Cada cliente do [!DNL Asset Compute service] - um projeto exclusivo [!DNL Adobe Developer Console] inscrito no serviço - deve [registrar](#register-request) antes de efetuar solicitações de processamento. A etapa de registro retorna o journal de evento único, necessário para recuperar os eventos assíncronos do processamento de execução.

Ao final de seu ciclo de vida, um cliente pode [cancelar o registro](#unregister-request).

### Solicitação de registro {#register-request}

Esta chamada de API configura um cliente [!DNL Asset Compute] e fornece o URL do journal do evento. Esta é uma operação impotente e só precisa ser chamada uma vez para cada cliente. Pode ser chamado novamente para recuperar o URL do journal.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/register` |
| Cabeçalho `Authorization` | Todos os cabeçalhos [relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento nos sistemas. |
| Corpo da solicitação | Deve estar vazio. |

### Registrar resposta {#register-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | O mesmo que o cabeçalho de solicitação `X-Request-Id` ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com os campos `journal`, `ok` e/ou `requestId`. |

Os códigos de status HTTP são:

* **200 Sucesso**: Quando a solicitação for bem-sucedida. Ele contém o URL `journal` que é notificado sobre quaisquer resultados do processamento assíncrono disparado via `/process` (como eventos, digite `rendition_created` quando bem-sucedidos, ou `rendition_failed` quando houver falha).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 Não autorizado**: ocorre quando a solicitação não tem  [autenticação](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem uma  [autorização](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não está inscrito em todos os serviços necessários.

* **429 Demasiadas solicitações**: ocorre quando o sistema é sobrecarregado por esse cliente ou de outra forma. Os clientes devem tentar novamente com um [backoff exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro** 4xx: Quando houve qualquer outro erro de cliente e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro** 5xx: ocorre quando há qualquer outro erro no lado do servidor e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Cancelar registro da solicitação {#unregister-request}

Esta chamada de API cancela o registro de um cliente [!DNL Asset Compute]. Depois disso, não é mais possível chamar `/process`. Usar a chamada da API para um cliente não registrado ou um cliente registrado ainda não registrado retorna um erro `404`.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/unregister` |
| Cabeçalho `Authorization` | Todos os cabeçalhos [relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento nos sistemas. |
| Corpo da solicitação | Vazio. |

### Cancelar registro da resposta {#unregister-response}

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | O mesmo que o cabeçalho de solicitação `X-Request-Id` ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com os campos `ok` e `requestId`. |

Os códigos de status são:

* **200 Sucesso**: ocorre quando o registro e o journal são encontrados e removidos.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 Não autorizado**: ocorre quando a solicitação não tem  [autenticação](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.

* **403 Proibido**: ocorre quando a solicitação não tem uma  [autorização](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não está inscrito em todos os serviços necessários.

* **404 Não encontrado**: ocorre quando não há registro atual para as credenciais fornecidas.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Demasiadas solicitações**: ocorre quando o sistema é sobrecarregado. Os clientes devem tentar novamente com um [backoff exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.

* **Erro** 4xx: ocorre quando ocorre qualquer outro erro de cliente e falha ao cancelar o registro. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro** 5xx: ocorre quando há qualquer outro erro no lado do servidor e o registro falhou. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Solicitação de processo {#process-request}

A operação `process` envia um trabalho que transforma um ativo de origem em várias representações, com base nas instruções na solicitação. As notificações sobre conclusão com êxito (tipo de evento `rendition_created`) ou quaisquer erros (tipo de evento `rendition_failed`) são enviadas para um journal de Evento que deve ser recuperado usando [/register](#register) uma vez antes de fazer qualquer número de solicitações `/process`. As solicitações formadas incorretamente falham imediatamente com um código de erro 400.

Os binários são referenciados usando URLs, como URLs pré-assinados do Amazon AWS S3 ou URLs SAS do Armazenamento Blob do Azure, para ler o ativo `source` (`GET` URLs) e gravar as execuções (`PUT` URLs). O cliente é responsável por gerar esses URLs pré-assinados.

| Parâmetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Caminho | `/process` |
| Tipo MIME | `application/json` |
| Cabeçalho `Authorization` | Todos os cabeçalhos [relacionados à autorização](#authentication-and-authorization). |
| Cabeçalho `x-request-id` | Opcional, pode ser definido pelos clientes para um identificador completo exclusivo das solicitações de processamento nos sistemas. |
| Corpo da solicitação | Deve estar no formato JSON de solicitação de processo, conforme descrito abaixo. Fornece instruções sobre qual ativo processar e quais execuções gerar. |

### Solicitação de processo JSON {#process-request-json}

O corpo da solicitação de `/process` é um objeto JSON com esse schema de alto nível:

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
| `source` | `object` | Descrever o ativo de origem a ser processado. Consulte a descrição de [campos de objeto de origem](#source-object-fields) abaixo. Opcional com base no formato de representação solicitado (por exemplo, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representações para gerar a partir do arquivo de origem. Cada objeto de representação suporta [instruções de representação](#rendition-instructions). Obrigatório. | `[{ "target": "https://....", "fmt": "png" }]` |

O `source` pode ser um `<string>` que é visto como um URL ou um `<object>` com um campo adicional. As seguintes variantes são semelhantes:

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
| `name` | `string` | Nome do arquivo do ativo de origem. A extensão de arquivo no nome pode ser usada se nenhum tipo MIME puder ser detectado. Tem prioridade sobre o nome do arquivo no caminho do URL ou nome do arquivo no cabeçalho `content-disposition` do recurso binário. O padrão é &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Tamanho do arquivo do ativo de origem em bytes. Tem prioridade sobre o cabeçalho `content-length` do recurso binário. | `10234` |
| `mimetype` | `string` | Tipo MIME do arquivo de ativo de origem. Tem precedência sobre o cabeçalho `content-type` do recurso binário. | `"image/jpeg"` |

### Um exemplo completo de solicitação `process` {#complete-process-request-example}

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

A solicitação `/process` retorna imediatamente com um sucesso ou uma falha com base na validação da solicitação básica. O processamento de ativos reais ocorre de forma assíncrona.

| Parâmetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Cabeçalho `X-Request-Id` | O mesmo que o cabeçalho de solicitação `X-Request-Id` ou um único gerado. Use para identificar solicitações em sistemas e/ou solicitações de suporte. |
| Corpo da resposta | Um objeto JSON com os campos `ok` e `requestId`. |

Códigos de status:

* **200 Sucesso**: Se a solicitação foi enviada com êxito. O JSON de resposta inclui `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Solicitação** inválida: Se a solicitação estiver formada incorretamente, como campos obrigatórios ausentes no JSON da solicitação. O JSON de resposta inclui `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 Não autorizado**: Quando a solicitação não tiver uma  [autenticação](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso inválido ou uma chave de API inválida.
* **403 Proibido**: Quando a solicitação não tiver uma  [autorização](#authentication-and-authorization) válida. Um exemplo pode ser um token de acesso válido, mas o projeto do Console do desenvolvedor do Adobe (conta técnica) não está inscrito em todos os serviços necessários.
* **429 Demasiadas solicitações**: Quando o sistema é sobrecarregado por este cliente ou em geral. Os clientes podem tentar novamente com um [backoff exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). O corpo está vazio.
* **Erro** 4xx: Quando havia algum outro erro de cliente. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Erro** 5xx: Quando havia outro erro no lado do servidor. Geralmente, uma resposta JSON como essa é retornada, embora isso não seja garantido para todos os erros:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

A maioria dos clientes provavelmente tendem a repetir a mesma solicitação com [backoff exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) em qualquer erro *exceto* problemas de configuração como 401 ou 403, ou solicitações inválidas como 400. Além da limitação regular da taxa por meio de respostas 429, uma interrupção ou limitação temporária do serviço pode resultar em erros 5xx. Seria então aconselhável tentar novamente após um período de tempo.

Todas as respostas JSON (se presentes) incluem `requestId`, que é o mesmo valor do cabeçalho `X-Request-Id`. É recomendável ler a partir do cabeçalho, pois ele está sempre presente. O `requestId` também é retornado em todos os eventos relacionados ao processamento de solicitações como `requestId`. Os clientes não devem assumir nenhuma suposição sobre o formato dessa string, pois ele é um identificador de string opaco.

## Inclusão no pós-processamento {#opt-in-to-post-processing}

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk) suporta um conjunto de opções básicas de pós-processamento de imagem. Os funcionários personalizados podem opt in explicitamente para pós-processamento definindo o campo `postProcess` no objeto de representação como `true`.

Os casos de uso suportados são:

* Recorte uma representação em um retângulo cujos limites são definidos por recorte.w, recorte.h, recorte.x e recorte.y. É definido por `instructions.crop` no objeto de representação.
* Redimensione imagens usando largura, altura ou ambos. É definido por `instructions.width` e `instructions.height` no objeto de representação. Para redimensionar usando apenas largura ou altura, defina apenas um valor. O Serviço de computação conserva a proporção.
* Defina a qualidade de uma imagem JPEG. É definido por `instructions.quality` no objeto de representação. A melhor qualidade é indicada por `100` e valores menores indicam qualidade reduzida.
* Crie imagens entrelaçadas. É definido por `instructions.interlace` no objeto de representação.
* Defina DPI para ajustar o tamanho renderizado para fins de publicação na área de trabalho ajustando a escala aplicada aos pixels. É definido por `instructions.dpi` no objeto de representação para alterar a resolução dpi. No entanto, para redimensionar a imagem de modo que ela tenha o mesmo tamanho em uma resolução diferente, use as instruções `convertToDpi`.
* Redimensione a imagem de forma que sua largura ou altura renderizada permaneça a mesma do original na DPI (resolução de público alvo especificada). É definido por `instructions.convertToDpi` no objeto de representação.

## Ativos de marca d&#39;água {#add-watermark}

O [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk) oferece suporte à adição de uma marca d&#39;água aos arquivos de imagem PNG, JPEG, TIFF e GIF. A marca d&#39;água é adicionada após as instruções de representação no objeto `watermark` na representação.

A marca d&#39;água é feita durante o pós-processamento da representação. Para marcar ativos, o trabalhador personalizado [opta pelo pós-processamento](#opt-in-to-post-processing) definindo o campo `postProcess` no objeto de representação como `true`. Se o trabalhador não aceitar, a marca d&#39;água não será aplicada, mesmo se o objeto de marca d&#39;água estiver definido no objeto de representação na solicitação.

## Instruções de execução {#rendition-instructions}

Estas são as opções disponíveis para a matriz `renditions` em [/process](#process-request).

### Campos comuns {#common-fields}

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | O formato de público alvo de execuções também pode ser `text` para extração de texto e `xmp` para extrair metadados XMP como xml. Consulte [formatos suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL de um [aplicativo personalizado](develop-custom-application.md). Deve ser um URL `https://`. Se esse campo estiver presente, a execução será criada por um aplicativo personalizado. Qualquer outro campo de representação definido é usado no aplicativo personalizado. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | URL para o qual a representação gerada deve ser carregada usando o PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Multipeça informações de upload de URL pré-assinado para a execução gerada. Este é para [Upload binário AEM/Oak Direct](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) com este [comportamento de carregamento multiparte](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fields:<ul><li>`urls`: matriz de strings, uma para cada URL de parte pré-assinada</li><li>`minPartSize`: o tamanho mínimo a ser usado para uma parte = url</li><li>`maxPartSize`: o tamanho máximo a ser usado para uma parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Espaço reservado opcional controlado pelo cliente e transmitido como está para eventos de execução. Permite que os clientes adicionem informações personalizadas para identificar eventos de execução. Não deve ser modificado ou confiável em aplicativos personalizados, pois os clientes podem alterar isso a qualquer momento. | `{ ... }` |

### Campos específicos de representação {#rendition-specific-fields}

Para obter uma lista dos formatos de arquivo suportados atualmente, consulte [formatos de arquivo suportados](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `*` | `*` | É possível adicionar campos avançados e personalizados que um [aplicativo personalizado](develop-custom-application.md) entende. |  |
| `embedBinaryLimit` | `number` em bytes | Se esse valor for definido e o tamanho do arquivo da representação for menor que esse valor, a representação será incorporada ao evento enviado assim que a geração da execução for concluída. O tamanho máximo permitido para incorporação é de 32 KB (32 x 1024 bytes). Se uma representação tiver um tamanho maior que o limite `embedBinaryLimit`, ela será colocada em um local no armazenamento da nuvem e não será incorporada no evento. | `3276` |
| `width` | `number` | Largura em pixels. somente para representações de imagem. | `200` |
| `height` | `number` | Altura em pixels. somente para representações de imagem. | `200` |
|  |  | A proporção é sempre mantida se: <ul> <li> Ambos `width` e `height` são especificados, e a imagem se ajusta ao tamanho enquanto mantém a proporção </li><li> Somente `width` ou apenas `height` é especificado, a imagem resultante usa a dimensão correspondente enquanto mantém a proporção</li><li> Se `width` ou `height` não for especificado, o tamanho original do pixel da imagem será usado. Depende do tipo de origem. Para alguns formatos, como arquivos PDF, um tamanho padrão é usado. Pode haver um limite máximo de tamanho.</li></ul> |  |
| `quality` | `number` | Especifique a qualidade jpeg no intervalo de `1` a `100`. Aplicável somente para representações de imagem. | `90` |
| `xmp` | `string` | Usado somente por XMP write-back de metadados, é XMP codificado em base64 para gravar de volta na representação especificada. |  |
| `interlace` | `bool` | Crie PNG ou GIF entrelaçado ou JPEG progressivo definindo-o como `true`. Não tem efeito em outros formatos de arquivo. |  |
| `jpegSize` | `number` | Tamanho aproximado do arquivo JPEG em bytes. Substitui qualquer configuração `quality`. Não tem efeito em outros formatos. |  |
| `dpi` | `number` ou `object` | Defina x e y DPI. Para simplificar, também pode ser definido para um único número que é usado para x e y. Não tem efeito na própria imagem. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` ou `object` | x e y DPI reamostram valores enquanto mantêm o tamanho físico. Para simplificar, também pode ser definido para um único número que é usado para x e y. | `96` ou `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de arquivos a serem incluídos no arquivo ZIP (`fmt=zip`). Cada entrada pode ser uma string de URL ou um objeto com os campos:<ul><li>`url`: URL para baixar o arquivo</li><li>`path`: Armazenar arquivo neste caminho no ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Manuseio de duplicados para arquivos ZIP (`fmt=zip`). Por padrão, vários arquivos armazenados no mesmo caminho no ZIP gera um erro. Definir `duplicate` como `ignore` resulta apenas no primeiro ativo a ser armazenado e no restante a ser ignorado. | `ignore` |
| `watermark` | `object` | Contém instruções sobre a [marca d&#39;água](#watermark-specific-fields). |  |

### Campos específicos de marca d&#39;água {#watermark-specific-fields}

O formato PNG é usado como uma marca d&#39;água.

| Nome | Tipo | Descrição | Exemplo |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala da marca d&#39;água, entre `0.0` e `1.0`. `1.0` significa que a marca de água tem a sua escala original (1:1) e os valores inferiores reduzem a dimensão da marca de água. | Um valor de `0.5` significa metade do tamanho original. |
| `image` | `url` | URL para o arquivo PNG a ser usado para a marca d&#39;água. |  |

## Eventos assíncronos {#asynchronous-events}

Quando o processamento de uma representação é concluído ou quando ocorre um erro, um evento é enviado para um Journal [[!DNL Adobe I/O] Eventos](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Os clientes devem escutar o URL do journal fornecido por meio de [/register](#register). A resposta do journal inclui uma matriz `event` que consiste em um objeto para cada evento, do qual o campo `event` inclui a carga real do evento.

O Tipo de evento [!DNL Adobe I/O] para todos os eventos de [!DNL Asset Compute Service] é `asset_compute`. O journal é automaticamente inscrito apenas neste tipo de evento e não há nenhum requisito adicional para filtrar com base no Tipo de evento [!DNL Adobe I/O]. Os tipos de evento específicos do serviço estão disponíveis na propriedade `type` do evento.

### Tipos de evento {#event-types}

| Evento | Descrição |
|---------------------|-------------|
| `rendition_created` | Enviado para cada execução processada e carregada com êxito. |
| `rendition_failed` | Enviado para cada execução que falhou ao processar ou carregar. |

### Atributos do evento {#event-attributes}

| Atributo | Tipo | Evento | Descrição |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Carimbo de data e hora quando o evento foi enviado no formato estendido simplificado [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601), conforme definido pelo JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | A id da solicitação original para `/process`, a mesma do cabeçalho `X-Request-Id`. |
| `source` | `object` | `*` | A `source` da solicitação `/process`. |
| `userData` | `object` | `*` | O `userData` da representação da solicitação `/process`, se definido. |
| `rendition` | `object` | `rendition_*` | O objeto de representação correspondente foi transmitido em `/process`. |
| `metadata` | `object` | `rendition_created` | As propriedades [metadata](#metadata) da representação. |
| `errorReason` | `string` | `rendition_failed` | Falha de execução [motivo](#error-reasons), se houver. |
| `errorMessage` | `string` | `rendition_failed` | Texto que fornece mais detalhes sobre a falha de representação, se houver. |

### Metadados {#metadata}

| Propriedade | Descrição |
|--------|-------------|
| `repo:size` | O tamanho da representação em bytes. |
| `repo:sha1` | O resumo sha1 da representação. |
| `dc:format` | O tipo MIME da execução. |
| `repo:encoding` | A codificação charset da representação, caso seja um formato baseado em texto. |
| `tiff:ImageWidth` | A largura da representação em pixels. Somente presente para representações de imagem. |
| `tiff:ImageLength` | O comprimento da representação em pixels. Somente presente para representações de imagem. |

### Motivos de erro {#error-reasons}

| Motivo | Descrição |
|---------|-------------|
| `RenditionFormatUnsupported` | Não há suporte para o formato de representação solicitado para a fonte em questão. |
| `SourceUnsupported` | A fonte específica não é suportada mesmo que o tipo seja suportado. |
| `SourceCorrupt` | Os dados de origem estão corrompidos. Inclui arquivos vazios. |
| `RenditionTooLarge` | Não foi possível fazer upload da representação usando os URLs pré-assinados fornecidos em `target`. O tamanho da representação real está disponível como metadados em `repo:size` e pode ser usado pelo cliente para reprocessar essa execução com o número certo de URLs pré-assinados. |
| `GenericError` | Qualquer outro erro inesperado. |
