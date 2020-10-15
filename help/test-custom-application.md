---
title: Testar e [!DNL Asset Compute Service] depurar aplicativo personalizado.
description: Testar e [!DNL Asset Compute Service] depurar aplicativo personalizado.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# Testar e depurar um aplicativo personalizado {#test-debug-custom-worker}

## Executar testes de unidade para um aplicativo personalizado {#test-custom-worker}

Instale o [Docker Desktop](https://www.docker.com/get-started) em sua máquina. Para testar um trabalhador personalizado, execute o seguinte comando na raiz do aplicativo:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Isso executa uma estrutura de teste de unidade personalizada para ações de aplicativo do Asset Compute no projeto, conforme descrito abaixo. Ele é conectado por uma configuração no `package.json` arquivo. Também é possível ter testes de unidade JavaScript, como Jest. `aio app test` executa ambos.

O plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) está incorporado como dependência de desenvolvimento no aplicativo personalizado para que ele não precise ser instalado em sistemas de compilação/teste.

### Estrutura de teste da unidade de aplicação {#unit-test-framework}

A estrutura de teste da unidade de aplicativo Asset Compute permite testar aplicativos sem gravar nenhum código. Ele depende do princípio de arquivos de origem para execução de aplicativos. É necessário configurar determinada estrutura de arquivos e pastas para definir casos de teste com arquivos de origem de teste, parâmetros opcionais, execuções esperadas e scripts de validação personalizados. Por padrão, as representações são comparadas para igualdade de bytes. Além disso, os serviços HTTP externos podem ser facilmente monitorados usando arquivos JSON simples.

### Adicionar testes {#add-tests}

Os testes são esperados dentro da `test` pasta no nível raiz do projeto AIO. Os casos de teste para cada aplicativo devem estar no caminho `test/asset-compute/<worker-name>`, com uma pasta para cada caso de teste:

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

Veja [exemplos de aplicativos](https://github.com/adobe/asset-compute-example-workers/) personalizados para ver alguns exemplos. Veja abaixo uma referência detalhada.

### Test output {#test-output}

A saída de teste detalhada, incluindo os registros do aplicativo personalizado, é disponibilizada na `build` pasta na raiz do aplicativo Firefly, como demonstrado na `aio app test` saída.

### Mock serviços externos {#mock-external-services}

É possível fazer o teste de chamadas de serviço externo em suas ações definindo `mock-<HOST_NAME>.json` arquivos em seus casos de teste, onde HOST_NAME é o host que você deseja testar. Um exemplo de caso de uso é um aplicativo que faz uma chamada separada para S3. A nova estrutura de teste seria parecida com esta:

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

O arquivo modelo é uma resposta http formatada JSON. Para obter mais informações, consulte [esta documentação](https://www.mock-server.com/mock_server/creating_expectations.html). Se houver vários nomes de host para cópia, defina vários `mock-<mocked-host>.json` arquivos. Abaixo está um exemplo de arquivo de amostra para `google.com` o nome `mock-google.com.json`:

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

O exemplo `worker-animal-pictures` contém um arquivo [](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) modelo para o serviço Wikimedia com o qual ele interage.

#### Compartilhar arquivos em casos de teste {#share-files-across-test-cases}

É recomendável usar links simbólicos relativos se você compartilhar `file.*`, `params.json` ou `validate` scripts em vários testes. Eles são suportados com git. Certifique-se de atribuir um nome exclusivo aos seus arquivos compartilhados, pois você pode ter arquivos diferentes. No exemplo abaixo, os testes estão misturando e correspondendo a alguns arquivos compartilhados, e seus próprios:

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

Os casos de testes de erro não devem conter um `rendition.*` arquivo esperado e devem definir o arquivo esperado `errorReason` dentro do `params.json` .

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

Consulte lista completa e descrição dos motivos [de erro da Computação de](https://github.com/adobe/asset-compute-commons#error-reasons)ativos.

## Depurar um aplicativo personalizado {#debug-custom-worker}

As etapas a seguir mostram como você pode depurar seu aplicativo personalizado usando o Código do Visual Studio. Ele permite ver registros ao vivo, pontos de interrupção de ocorrência e passar pelo código, bem como recarregar ao vivo as alterações do código local em cada ativação.

Muitas dessas etapas normalmente são automatizadas `aio` prontamente, consulte a seção Depuração do aplicativo na documentação [do](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)Firefly. Por enquanto, as etapas abaixo incluem uma solução alternativa.

1. Instale a última [wskdebug](https://github.com/apache/openwhisk-wskdebug) do GitHub e o [ngrok](https://www.npmjs.com/package/ngrok)opcional.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Adicione ao seu arquivo JSON de configurações de usuário. Ele continua usando o antigo depurador de código VS, o novo tem [alguns problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) com wskdebug: `"debug.javascript.usePreview": false`.
1. Feche todas as instâncias de aplicativos abertos via `aio app run`.
1. Implante o código mais recente usando `aio app deploy`.
1. Execute somente a ferramenta de computação do Ativo usando `npx adobe-asset-compute devtool`. Mantenha aberto.
1. No editor de código VS, adicione a configuração de depuração abaixo a seu `launch.json`:

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

   Procure o NOME DA AÇÃO na saída de `aio app deploy`. Parece `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`.

1. Selecione `wskdebug worker` na configuração de execução/depuração e pressione o ícone de reprodução. Aguarde o start até que ele seja exibido **[!UICONTROL Ready for ativação]** na janela **[!UICONTROL Debug Console]** .

1. Clique em **[!UICONTROL executar]** no Devtool. É possível ver as ações executadas no editor de Código VS e o start de registros sendo exibido.

1. Defina um ponto de interrupção em seu código, execute novamente e ele deve ser acessado.

Todas as alterações de código são carregadas em tempo real e entram em vigor assim que a próxima ativação ocorrer.

>[!NOTE]
>
>Duas ativações estão presentes para cada solicitação em aplicativos personalizados. A primeira solicitação é uma ação da Web que se autochama de forma assíncrona no código do SDK. A segunda ativação é aquela que atinge seu código.
