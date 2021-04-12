---
title: Testar e depurar  [!DNL Asset Compute Service] aplicativo personalizado
description: Teste e depure [!DNL Asset Compute Service] aplicativo personalizado.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
translation-type: tm+mt
source-git-commit: 9bc1534671c81a05798f98ae556d348bc771d975
workflow-type: tm+mt
source-wordcount: '782'
ht-degree: 0%

---

# Testar e depurar um aplicativo personalizado {#test-debug-custom-worker}

## Executar testes de unidade para um aplicativo personalizado {#test-custom-worker}

Instale o [Docker Desktop](https://www.docker.com/get-started) no computador. Para testar um trabalhador personalizado, execute o seguinte comando na raiz do aplicativo:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Isso executa uma estrutura de teste de unidade personalizada para ações do aplicativo Asset compute no projeto, conforme descrito abaixo. Ela é conectada por meio de uma configuração no arquivo `package.json`. Também é possível ter testes de unidade JavaScript, como Jest. `aio app test` O executa ambos.

O plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) é incorporado como dependência de desenvolvimento no aplicativo personalizado para que não precise ser instalado em sistemas de build/teste.

### Estrutura de teste da unidade de aplicação {#unit-test-framework}

A estrutura de teste da unidade de aplicativo do Asset compute permite testar aplicativos sem gravar nenhum código. Ele depende do princípio de arquivo de origem para representação de aplicativos. Uma determinada estrutura de arquivos e pastas deve ser configurada para definir casos de teste com arquivos de origem de teste, parâmetros opcionais, representações esperadas e scripts de validação personalizados. Por padrão, as representações são comparadas para igualdade de bytes. Além disso, os serviços HTTP externos podem ser facilmente monitorados usando arquivos JSON simples.

### Adicionar testes {#add-tests}

Os testes são esperados dentro da pasta `test` no nível raiz do projeto [!DNL Adobe I/O]. Os casos de teste para cada aplicativo devem estar no caminho `test/asset-compute/<worker-name>`, com uma pasta para cada caso de teste:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Consulte [exemplo de aplicativos personalizados](https://github.com/adobe/asset-compute-example-workers/) para obter alguns exemplos. Veja abaixo uma referência detalhada.

### Testar saída {#test-output}

A saída de teste detalhada, incluindo os logs do aplicativo personalizado, é disponibilizada na pasta `build` na raiz do aplicativo Firefly, conforme demonstrado na saída `aio app test`.

### Mover serviços externos {#mock-external-services}

É possível fazer o mock das chamadas de serviço externo em suas ações definindo arquivos `mock-<HOST_NAME>.json` em seus casos de teste, onde HOST_NAME é o host que você deseja fazer o mock. Um exemplo de caso de uso é um aplicativo que faz uma chamada separada para S3. A nova estrutura de teste seria assim:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

O arquivo modelo é uma resposta http formatada em JSON. Para obter mais informações, consulte [esta documentação](https://www.mock-server.com/mock_server/creating_expectations.html). Se houver vários nomes de host para modelar, defina vários arquivos `mock-<mocked-host>.json`. Abaixo está um exemplo de arquivo de amostra para `google.com` chamado `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

O exemplo `worker-animal-pictures` contém um [mock file](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para o serviço Wikimedia com o qual ele interage.

#### Compartilhar arquivos em casos de teste {#share-files-across-test-cases}

É recomendável usar links simbólicos relativos se você compartilhar scripts `file.*`, `params.json` ou `validate` em vários testes. Eles são compatíveis com git. Certifique-se de dar um nome exclusivo aos arquivos compartilhados, pois você pode ter arquivos diferentes. No exemplo abaixo, os testes estão misturando e correspondendo a alguns arquivos compartilhados, e seus próprios:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Testar erros esperados {#test-unexpected-errors}

Os casos de testes de erro não devem conter um arquivo `rendition.*` esperado e devem definir o `errorReason` esperado dentro do arquivo `params.json`.

Estrutura do caso de teste de erro:

```json
<error_test_case>/
    file.jpg
    params.json
```

Arquivo de parâmetro com motivo de erro:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Consulte a lista completa e a descrição dos [motivos de erro de Asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depurar um aplicativo personalizado {#debug-custom-worker}

As etapas a seguir mostram como você pode depurar seu aplicativo personalizado usando o Código do Visual Studio. Ele permite ver registros ao vivo, pontos de interrupção de ocorrência e passar pelo código, bem como recarregar ao vivo as alterações do código local em cada ativação.

Muitas dessas etapas geralmente são automatizadas por `aio` prontas para uso, consulte a seção Depuração do aplicativo na [documentação do Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Por enquanto, as etapas abaixo incluem uma solução alternativa.

1. Instale o mais recente [wskdebug](https://github.com/apache/openwhisk-wskdebug) do GitHub e o [ngrok](https://www.npmjs.com/package/ngrok) opcional.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Adicionar ao arquivo JSON das configurações do usuário. Ele continua usando o antigo depurador de código VS, o novo tem [alguns problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) com o wskdebug: `"debug.javascript.usePreview": false`.
1. Feche todas as instâncias de aplicativos abertas por `aio app run`.
1. Implante o código mais recente usando `aio app deploy`.
1. Execute somente o Asset compute Devtool usando `aio asset-compute devtool`. Mantenha aberto.
1. No Editor de código VS, adicione a seguinte configuração de depuração ao `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Busque o `ACTION NAME` da saída de `aio app deploy`.

1. Selecione `wskdebug worker` na configuração de execução/depuração e pressione o ícone de reprodução. Aguarde até que ele inicie até que exiba **[!UICONTROL Pronto para ativações]** na janela **[!UICONTROL Debug Console]**.

1. Clique em **[!UICONTROL executar]** no Devtool. Você pode ver as ações executadas no editor de Código VS e os logs começam a ser exibidos.

1. Defina um ponto de interrupção no seu código, execute novamente e ele deverá atingir.

Todas as alterações de código são carregadas em tempo real e entrarão em vigor assim que a próxima ativação ocorrer.

>[!NOTE]
>
>Duas ativações estão presentes para cada solicitação em aplicativos personalizados. A primeira solicitação é uma ação da Web que se autochama de forma assíncrona no código do SDK. A segunda ativação é aquela que atinge seu código.
