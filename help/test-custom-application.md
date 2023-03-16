---
title: Testar e depurar [!DNL Asset Compute Service] aplicativo personalizado
description: Testar e depurar [!DNL Asset Compute Service] aplicativo personalizado.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# Testar e depurar um aplicativo personalizado {#test-debug-custom-worker}

## Executar testes de unidade para um aplicativo personalizado {#test-custom-worker}

Instalar [Desktop Docker](https://www.docker.com/get-started) na sua máquina. Para testar um trabalhador personalizado, execute o seguinte comando na raiz do aplicativo:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Isso executa uma estrutura de teste de unidade personalizada para ações do aplicativo Asset compute no projeto, conforme descrito abaixo. Ele é acoplado por meio de uma configuração no `package.json` arquivo. Também é possível ter testes de unidade JavaScript, como Jest. `aio app test` O executa ambos.

O [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) O plug-in é incorporado como dependência de desenvolvimento no aplicativo personalizado para que não precise ser instalado em sistemas de build/teste.

### Estrutura de teste da unidade de aplicação {#unit-test-framework}

A estrutura de teste da unidade de aplicativo do Asset compute permite testar aplicativos sem gravar nenhum código. Ele depende do princípio de arquivo de origem para representação de aplicativos. Uma determinada estrutura de arquivos e pastas deve ser configurada para definir casos de teste com arquivos de origem de teste, parâmetros opcionais, representações esperadas e scripts de validação personalizados. Por padrão, as representações são comparadas para igualdade de bytes. Além disso, os serviços HTTP externos podem ser facilmente monitorados usando arquivos JSON simples.

### Adicionar testes {#add-tests}

Os testes são esperados dentro da `test` no nível raiz da [!DNL Adobe I/O] projeto. Os casos de teste para cada aplicativo devem estar no caminho `test/asset-compute/<worker-name>`, com uma pasta para cada caso de teste:

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

Dê uma olhada em [exemplo de aplicativos personalizados](https://github.com/adobe/asset-compute-example-workers/) para obter alguns exemplos. Veja abaixo uma referência detalhada.

### Saída de teste {#test-output}

Os resultados detalhados do teste, incluindo os registros do aplicativo personalizado, são disponibilizados no `build` na raiz do aplicativo Adobe Developer App Builder, conforme demonstrado no `aio app test` saída.

### Empilhar serviços externos {#mock-external-services}

É possível monitorar as chamadas de serviço externo em suas ações ao definir `mock-<HOST_NAME>.json` nos seus casos de teste, onde HOST_NAME é o host que você deseja fazer o modelo. Um exemplo de caso de uso é um aplicativo que faz uma chamada separada para S3. A nova estrutura de teste seria assim:

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

O arquivo modelo é uma resposta http formatada em JSON. Para obter mais informações, consulte [esta documentação](https://www.mock-server.com/mock_server/creating_expectations.html). Se houver vários nomes de host para modelar, defina vários `mock-<mocked-host>.json` arquivos. Abaixo está um exemplo de arquivo de modelo para `google.com` nomeado `mock-google.com.json`:

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

O exemplo `worker-animal-pictures` contém um [arquivo modelo](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para o serviço Wikimedia, interage.

#### Compartilhar arquivos em casos de teste {#share-files-across-test-cases}

É recomendável usar links simbólicos relativos se você compartilhar `file.*`, `params.json` ou `validate` scripts em vários testes. Eles são compatíveis com git. Certifique-se de dar um nome exclusivo aos arquivos compartilhados, pois você pode ter arquivos diferentes. No exemplo abaixo, os testes estão misturando e correspondendo a alguns arquivos compartilhados, e seus próprios:

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

Os casos de testes de erro não devem conter uma `rendition.*` e deve definir o `errorReason` dentro do `params.json` arquivo.

>[!NOTE]
>
>Se um caso de teste não contiver um valor esperado `rendition.*` e não define o `errorReason` dentro do `params.json` é considerado um caso de erro com qualquer `errorReason`.

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

Consulte a lista completa e a descrição [Motivos de erro do asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depurar um aplicativo personalizado {#debug-custom-worker}

As etapas a seguir mostram como você pode depurar seu aplicativo personalizado usando o Código do Visual Studio. Ele permite ver registros ao vivo, pontos de interrupção de ocorrência e passar pelo código, bem como recarregar ao vivo as alterações do código local em cada ativação.

Muitas dessas etapas geralmente são automatizadas por `aio` para uso imediato, consulte a seção Depuração do aplicativo no [Documentação do Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Por enquanto, as etapas abaixo incluem uma solução alternativa.

1. Instale o mais recente [wskdebug](https://github.com/apache/openwhisk-wskdebug) do GitHub e do [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Adicionar ao arquivo JSON das configurações do usuário. Ele continua usando o antigo depurador de código VS, o novo possui [alguns problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) com wskdebug: `"debug.javascript.usePreview": false`.
1. Feche todas as instâncias de aplicativos abertas por `aio app run`.
1. Implante o código mais recente usando `aio app deploy`.
1. Execute apenas o Asset compute Devtool usando `aio asset-compute devtool`. Mantenha aberto.
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

   Busque a `ACTION NAME` da saída de `aio app deploy`.

1. Selecionar `wskdebug worker` na configuração executar/depurar e pressione o ícone reproduzir . Aguarde até que ele inicie até que seja exibido **[!UICONTROL Pronto para ativações]** no **[!UICONTROL Console de depuração]** janela.

1. Clique em **[!UICONTROL executar]** no Devtool. Você pode ver as ações executadas no editor de Código VS e os logs começam a ser exibidos.

1. Defina um ponto de interrupção no seu código, execute novamente e ele deverá atingir.

Todas as alterações de código são carregadas em tempo real e entrarão em vigor assim que a próxima ativação ocorrer.

>[!NOTE]
>
>Duas ativações estão presentes para cada solicitação em aplicativos personalizados. A primeira solicitação é uma ação da Web que se autochama de forma assíncrona no código do SDK. A segunda ativação é aquela que atinge seu código.
