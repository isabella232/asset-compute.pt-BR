---
title: Desenvolver para [!DNL Asset Compute Service].
description: Crie aplicativos personalizados usando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 127895cf1bab59546f9ba0be2b3b7a935627effb
workflow-type: tm+mt
source-wordcount: '1496'
ht-degree: 0%

---


# Desenvolver um aplicativo personalizado {#develop}

Antes de começar a desenvolver um aplicativo personalizado:

* Verifique se todos os [pré-requisitos](/help/understand-extensibility.md#prerequisites-and-provisioning) foram atendidos.
* Instale as ferramentas [de software](/help/setup-environment.md#create-dev-environment)necessárias.
* Consulte [Configurar seu ambiente](setup-environment.md) para garantir que você esteja pronto para criar um aplicativo personalizado.

## Criar um aplicativo personalizado {#create-custom-application}

Verifique se a CLI [de E/S do](https://github.com/adobe/aio-cli) Adobe está instalada localmente.

1. Para criar um aplicativo personalizado, [crie um aplicativo](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)Firefly. Para fazer isso, execute `aio app init <app-name>` no terminal.

   Se você ainda não tiver feito logon, este comando solicitará que você entre no [Adobe Developer Console](https://console.adobe.io/) com seu Adobe ID. Consulte [aqui](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) para obter mais informações sobre como fazer logon a partir do cli.

   O Adobe recomenda que você faça logon. Se tiver problemas, siga as instruções [para criar um aplicativo sem fazer logon](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Depois de fazer logon, siga os prompts na CLI e selecione o `Organization`, `Project`e `Workspace` para usar no aplicativo. Escolha o projeto e o espaço de trabalho criados ao [configurar seu ambiente](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Quando solicitado com `Which Adobe I/O App features do you want to enable for this project?`, selecione pelo menos `Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Quando solicitado `Which type of sample actions do you want to create?`, selecione `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Siga os outros prompts e abra o novo aplicativo no Código do Visual Studio (ou seu editor de código favorito). Ele contém o andaime e o código de amostra de um aplicativo personalizado.

   Leia aqui sobre os componentes [principais de um aplicativo](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)Firefly.

   O aplicativo modelo aproveita nosso SDK [de Computação de](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) ativos para fazer upload, download e orquestração de execuções de aplicativos, de modo que os desenvolvedores precisam apenas implementar a lógica personalizada do aplicativo. Dentro da `actions/<worker-name>` pasta, o `index.js` arquivo é onde adicionar o código do aplicativo personalizado.

Consulte [exemplos de aplicativos](#try-sample) personalizados para obter exemplos e ideias para aplicativos personalizados.

### Adicionar credenciais {#add-credentials}

Conforme você faz logon ao criar o aplicativo, a maioria das credenciais do Firefly são coletadas no arquivo ENV. Entretanto, o uso da ferramenta para desenvolvedores requer credenciais adicionais.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenciais do armazenamento da ferramenta para desenvolvedores {#developer-tool-credentials}

A ferramenta para desenvolvedores usada para testar aplicativos personalizados com o armazenamento real [!DNL Asset Compute service] requer um container de nuvem para hospedar arquivos de teste e para receber e exibir execuções geradas por aplicativos.

>[!NOTE]
>
>Isso é separado do armazenamento de nuvem [!DNL Adobe Experience Manager] como um Cloud Service. Ela se aplica somente ao desenvolvimento e teste com a ferramenta para desenvolvedores Asset Compute.

Certifique-se de ter acesso a um container [de armazenamento em nuvem](https://github.com/adobe/asset-compute-devtool#prerequisites)suportado. Este container pode ser compartilhado por vários desenvolvedores em diferentes projetos, conforme necessário.

#### Adicionar credenciais ao arquivo ENV {#add-credentials-env-file}

Adicione as seguintes credenciais para a ferramenta de desenvolvedor ao arquivo ENV na raiz do projeto Firefly:

1. Adicione o caminho absoluto ao arquivo de chave privada criado ao adicionar serviços ao projeto Firefly:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Adicione credenciais de Armazenamento S3 ou Azure. Você só precisa acessar uma solução de armazenamento em nuvem.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

## Executar o aplicativo {#run-custom-application}

Antes de executar o aplicativo com a ferramenta para desenvolvedores Asset Compute, configure as [credenciais](#developer-tool-credentials)corretamente.

Para executar o aplicativo na ferramenta para desenvolvedor, use `aio app run` o comando. Ele implanta a ação na Adobe I/O Runtime e start a ferramenta de desenvolvimento na máquina local. Essa ferramenta é usada para testar solicitações de aplicativos durante o desenvolvimento. Veja um exemplo de solicitação de representação:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Não use o sinalizador `--local` com o `run` comando. Ele não funciona com aplicativos [!DNL Asset Compute] personalizados e com a ferramenta Asset Compute Developer. Os aplicativos personalizados são chamados pelo [!DNL Asset Compute Service] que não pode acessar ações executadas nos computadores locais do desenvolvedor.

Consulte [aqui](test-custom-application.md) como testar e depurar seu aplicativo. Quando terminar de desenvolver seu aplicativo personalizado, [implante seu aplicativo](deploy-custom-application.md)personalizado.

## Experimente o aplicativo de amostra fornecido pelo Adobe {#try-sample}

Veja a seguir exemplos de aplicativos personalizados:

* [trabalhadora básica](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [imagens de animais de estimação](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aplicativo personalizado de modelo {#template-custom-application}

O [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) é um aplicativo de modelo. Ele gera uma execução simplesmente copiando o arquivo de origem. O conteúdo deste aplicativo é o modelo recebido ao escolher `Adobe Asset Compute` na criação do aplicativo de rádio.

O arquivo do aplicativo [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) usa o [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) para baixar o arquivo de origem, orquestrar cada processamento de execução e fazer upload das execuções resultantes de volta ao armazenamento da nuvem.

O [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definido dentro do código do aplicativo é onde executar toda a lógica de processamento do aplicativo. O retorno de chamada de execução em `worker-basic` simplesmente copia o conteúdo do arquivo de origem para o arquivo de renderização.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Chamar uma API externa {#call-external-api}

No código do aplicativo, você pode fazer chamadas de API externas para ajudar no processamento do aplicativo. Um exemplo de arquivo de aplicativo que chama a API externa está abaixo.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Por exemplo, o [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) faz uma solicitação de busca para um URL estático da Wikimedia usando a [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) biblioteca.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Enviar parâmetros personalizados {#pass-custom-parameters}

Você pode passar parâmetros definidos personalizados pelos objetos de execução. Eles podem ser referenciados dentro do aplicativo em [`rendition` instruções](https://github.com/adobe/asset-compute-sdk#rendition). Um exemplo de um objeto de representação é:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Um exemplo de um arquivo de aplicativo acessando um parâmetro personalizado é:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

O `example-worker-animal-pictures` envia um parâmetro personalizado [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) para determinar qual arquivo buscar na Wikimedia.

## Suporte de autenticação e autorização {#authentication-authorization-support}

Por padrão, os aplicativos personalizados do Asset Compute vêm com verificações de autorização e autenticação para aplicativos Firefly. Isso é ativado ao configurar a `require-adobe-auth` anotação como `true` no `manifest.yml`.

### Acessar outras APIs de Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Adicione os serviços da API à área de trabalho do [!DNL Asset Compute] Console criada na configuração. Esses serviços são parte do token de acesso JWT gerado pelo [!DNL Asset Compute Service]. O token e outras credenciais estão acessíveis dentro do `params` objeto de ação do aplicativo.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Enviar credenciais para sistemas de terceiros {#pass-credentials-for-tp}

Para manipular credenciais para outros serviços externos, passe-as como parâmetros padrão nas ações. Eles são automaticamente criptografados em trânsito. Para obter mais informações, consulte [Criação de ações no guia](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)do desenvolvedor do Runtime. Em seguida, defina-os usando variáveis de ambiente durante a implantação. Esses parâmetros podem ser acessados no `params` objeto dentro da ação.

Defina os parâmetros padrão dentro do `inputs` na `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

A `$VAR` expressão lê o valor de uma variável de ambiente chamada `VAR`.

Durante o desenvolvimento, o valor pode ser definido no arquivo ENV local, pois lê `aio` automaticamente as variáveis do ambiente dos arquivos ENV, além das variáveis definidas do shell de chamada. Neste exemplo, o arquivo ENV tem a seguinte aparência:

```CONF
#...
SECRET_KEY=secret-value
```

Para implantação de produção, é possível definir as variáveis de ambiente no sistema CI, por exemplo, usando segredos nas Ações GitHub. Por fim, acesse os parâmetros padrão dentro do aplicativo como tal:

```javascript
const key = params.secretKey;
```

## Dimensionamento de aplicativos {#sizing-workers}

Um aplicativo é executado em um container no Adobe I/O Runtime com [limites](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) que podem ser configurados por meio do `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Devido ao processamento mais extenso normalmente feito pelos aplicativos de Computação de ativos, é mais provável que seja necessário ajustar esses limites para obter desempenho ideal (grande o suficiente para lidar com ativos binários) e eficiência (não desperdiçar recursos devido à memória não utilizada do container).

O tempo limite padrão para ações em Tempo de execução é de um minuto, mas pode ser aumentado pela definição do `timeout` limite (em milissegundos). Se você espera processar arquivos maiores, aumente esse tempo. Considere o tempo total necessário para baixar a fonte, processar o arquivo e fazer upload da execução. Se uma ação expirar, ou seja, não retornar a ativação antes do limite de tempo limite especificado, o Tempo de execução descartará o container e não o reutilizará.

Os aplicativos de computação de ativos, por natureza, tendem a ser conectados à rede e E/S de disco. O arquivo de origem deve ser baixado primeiro, o processamento é frequentemente de E/S pesada e as execuções resultantes são carregadas novamente.

A memória disponível para um container de ação é especificada por `memorySize` em MB. Atualmente, isso também define a quantidade de acesso da CPU que o container recebe, e mais importante, é um elemento chave do custo de uso do Tempo de execução (container maiores custam mais). Use um valor maior aqui quando o seu processamento exigir mais memória ou CPU, mas tenha cuidado para não desperdiçar recursos, quanto maior for o container, menor será o throughput geral.

Além disso, é possível controlar a simultaneidade de ações dentro de um container usando a `concurrency` configuração. Esse é o número de ativações simultâneas que um único container recebe (da mesma ação). Neste modelo, o container de ação é como um servidor Node.js que recebe várias solicitações simultâneas, até esse limite. Se não estiver definido, o padrão em Tempo de execução é 200, o que é ótimo para ações menores do Firefly, mas geralmente muito grande para aplicativos do Asset Compute, devido ao processamento local mais intenso e à atividade do disco. Alguns aplicativos, dependendo de sua implementação, também podem não funcionar bem com atividades simultâneas. O SDK do Asset Compute garante que as ativações sejam separadas gravando arquivos em diferentes pastas exclusivas.

Teste os aplicativos para encontrar os números ideais para `concurrency` e `memorySize`. Container maiores = o limite de memória mais alto pode permitir mais simultaneidade, mas também pode ser um desperdício para tráfego mais baixo.
