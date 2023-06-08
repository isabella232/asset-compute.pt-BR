---
title: Teste e depuração [!DNL Asset Compute Service] aplicativo personalizado
description: Teste e depuração [!DNL Asset Compute Service] aplicativo personalizado.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
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

Isso executa uma estrutura de teste de unidade personalizada para ações de aplicativo do Asset compute no projeto, conforme descrito abaixo. Ele é conectado por meio de uma configuração no `package.json` arquivo. Também é possível ter testes de unidade JavaScript, como o Jest. `aio app test` O executa ambos.

A variável [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) o plug-in é incorporado como dependência de desenvolvimento no aplicativo de aplicativo personalizado, de modo que não precise ser instalado em sistemas de compilação/teste.

### Estrutura de teste da unidade de aplicativos {#unit-test-framework}

A estrutura de teste da unidade de aplicativo do Asset compute permite testar aplicativos sem gravar nenhum código. Depende do princípio de origem para representação de arquivos de aplicativos. Uma determinada estrutura de arquivos e pastas deve ser configurada para definir casos de teste com arquivos de origem de teste, parâmetros opcionais, representações esperadas e scripts de validação personalizados. Por padrão, as representações são comparadas em relação à igualdade de bytes. Além disso, os serviços HTTP externos podem ser facilmente ridicularizados usando arquivos JSON simples.

### Adicionar testes {#add-tests}

Os testes são esperados dentro do `test` pasta no nível raiz da variável [!DNL Adobe I/O] projeto. Os casos de teste para cada aplicativo devem estar no caminho `test/asset-compute/<worker-name>`, com uma pasta para cada caso de teste:

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

Veja [exemplo de aplicativos personalizados](https://github.com/adobe/asset-compute-example-workers/) para obter alguns exemplos. Abaixo está uma referência detalhada.

### Saída de teste {#test-output}

Os resultados detalhados do teste, incluindo os logs do aplicativo personalizado, são disponibilizados na `build` pasta na raiz do aplicativo Adobe Developer App Builder, conforme demonstrado na `aio app test` saída.

### Serviços externos fictícios {#mock-external-services}

É possível simular chamadas de serviço externas em suas ações definindo `mock-<HOST_NAME>.json` em seus casos de teste, onde HOST_NAME é o host que você gostaria de simular. Um exemplo de caso de uso é um aplicativo que faz uma chamada separada para S3. A nova estrutura de teste seria semelhante a:

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

O arquivo modelo é uma resposta http formatada em JSON. Para obter mais informações, consulte [esta documentação](https://www.mock-server.com/mock_server/creating_expectations.html). Se houver vários nomes de host a serem simulados, defina vários `mock-<mocked-host>.json` arquivos. Abaixo está um exemplo de arquivo modelo para `google.com` nomeado `mock-google.com.json`:

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

O exemplo `worker-animal-pictures` contém um [arquivo fictício](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para o serviço Wikimedia com o qual interage.

#### Compartilhar arquivos entre casos de teste {#share-files-across-test-cases}

É recomendável usar ligações simbólicas relativas se você compartilhar `file.*`, `params.json` ou `validate` scripts em vários testes. Eles são compatíveis com o Git. Atribua um nome exclusivo aos arquivos compartilhados, pois você pode ter arquivos diferentes. No exemplo abaixo, os testes estão misturando e correspondendo a alguns arquivos compartilhados e seus próprios:

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

### Erros esperados do teste {#test-unexpected-errors}

Os casos de testes de erro não devem conter um valor esperado `rendition.*` arquivo e deve definir o valor esperado `errorReason` dentro do `params.json` arquivo.

>[!NOTE]
>
>Se um caso de teste não contiver um esperado `rendition.*` arquivo e não define o esperado `errorReason` dentro do `params.json` arquivo, presume-se que seja um caso de erro com `errorReason`.

Estrutura de caso de teste de erro:

```json
<error_test_case>/
    file.jpg
    params.json
```

Arquivo de parâmetros com motivo do erro:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Veja a lista completa e a descrição de [Motivos de erro de asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depurar um aplicativo personalizado {#debug-custom-worker}

As etapas a seguir mostram como você pode depurar seu aplicativo personalizado usando o Visual Studio Code. Ele permite a visualização de logs em tempo real, pontos de interrupção de ocorrência e código de passo a passo, bem como o recarregamento em tempo real de alterações de código local em cada ativação.

Muitas dessas etapas geralmente são automatizadas pelo `aio` pronto para uso, consulte a seção Depuração do aplicativo no [Documentação do Construtor de aplicativos Adobe Developer](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Por enquanto, as etapas abaixo incluem uma solução alternativa.

1. Instalar o mais recente [wskdebug](https://github.com/apache/openwhisk-wskdebug) do GitHub e do módulo opcional [grok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Adicione ao arquivo JSON de configurações do usuário. Ele continua usando o depurador de código VS antigo, o novo tem [alguns problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) com wskdebug: `"debug.javascript.usePreview": false`.
1. Feche todas as instâncias dos aplicativos abertos via `aio app run`.
1. Implante o código mais recente usando o `aio app deploy`.
1. Execute somente o Asset compute Devtool usando `aio asset-compute devtool`. Mantenha-o aberto.
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

   Busque o `ACTION NAME` a partir da saída de `aio app deploy`.

1. Selecionar `wskdebug worker` na configuração executar/depurar e pressione o ícone reproduzir. Aguarde até que ele seja iniciado **[!UICONTROL Pronto para ativações]** no **[!UICONTROL Console de depuração]** janela.

1. Clique em **[!UICONTROL executar]** no Devtool. Você pode ver as ações executadas no editor de código VS e os registros começarem a ser exibidos.

1. Defina um ponto de interrupção no código, execute novamente e ele deverá ocorrer.

Quaisquer alterações de código são carregadas em tempo real e entram em vigor assim que ocorrer a próxima ativação.

>[!NOTE]
>
>Duas ativações estão presentes para cada solicitação em aplicativos personalizados. A primeira solicitação é uma ação da Web chamada a si mesma de forma assíncrona no código do SDK. A segunda ativação é aquela que atinge seu código.
