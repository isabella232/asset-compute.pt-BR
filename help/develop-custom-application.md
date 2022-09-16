---
title: Desenvolver para [!DNL Asset Compute Service]
description: Crie aplicativos personalizados usando [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: a121b48d480b45405259c2061ac86b9ab46b89cb
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Desenvolver um aplicativo personalizado {#develop}

Antes de começar a desenvolver um aplicativo personalizado:

* Certifique-se de que todas as [pré-requisitos](/help/understand-extensibility.md#prerequisites-and-provisioning) são atendidas.
* Instale o [ferramentas de software necessárias](/help/setup-environment.md#create-dev-environment).
* Consulte [configurar seu ambiente](setup-environment.md) para certificar-se de que você está pronto para criar um aplicativo personalizado.

## Criar um aplicativo personalizado {#create-custom-application}

Certifique-se de ter a variável [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) instalado localmente.

1. Para criar um aplicativo personalizado, [criar um projeto do App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Para fazer isso, execute `aio app init <app-name>` no terminal.

   Se você ainda não tiver feito logon, este comando solicitará que você entre no [Console do Adobe Developer](https://console.adobe.io/) com sua Adobe ID. Consulte [here](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) para obter mais informações sobre como fazer logon pela cli.

   O Adobe recomenda fazer logon. Se tiver problemas, siga as instruções [para criar um aplicativo sem fazer logon](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Depois de fazer logon, siga os prompts na CLI e selecione o `Organization`, `Project`e `Workspace` para usar no aplicativo. Escolha o projeto e o espaço de trabalho criados ao [configurar seu ambiente](setup-environment.md). Quando solicitado `Which extension point(s) do you wish to implement ?`, certifique-se de selecionar `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Quando solicitado com `Which Adobe I/O App features do you want to enable for this project?`, selecione `Actions`. Certifique-se de desmarcar `Web Assets` como ativos da web usam diferentes verificações de autenticação e autorização.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Quando solicitado `Which type of sample actions do you want to create?`, certifique-se de selecionar `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Siga o resto dos prompts e abra o novo aplicativo no Visual Studio Code (ou seu editor de código favorito). Ele contém o scaffolding e o código de amostra de um aplicativo personalizado.

   Leia aqui sobre o [principais componentes de um aplicativo App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   O aplicativo de modelo aproveita nosso [SDK do Asset compute](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) para fazer upload, download e orquestração de execuções de aplicativos, os desenvolvedores precisam apenas implementar a lógica do aplicativo personalizado. Dentro do `actions/<worker-name>` , a `index.js` é onde adicionar o código de aplicativo personalizado.

Consulte [exemplo de aplicativos personalizados](#try-sample) para obter exemplos e ideias para aplicativos personalizados.

### Adicionar credenciais {#add-credentials}

À medida que você faz logon ao criar o aplicativo, a maioria das credenciais do App Builder é coletada em seu arquivo ENV. No entanto, usar a ferramenta de desenvolvedor requer credenciais adicionais.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenciais de armazenamento da ferramenta de desenvolvedor {#developer-tool-credentials}

A ferramenta de desenvolvedor usada para testar aplicativos personalizados com o [!DNL Asset Compute service] O requer um contêiner de armazenamento em nuvem para hospedar arquivos de teste e para receber e exibir representações geradas por aplicativos.

>[!NOTE]
>
>Isso é separado do armazenamento em nuvem do [!DNL Adobe Experience Manager] como [!DNL Cloud Service]. Ela só se aplica ao desenvolvimento e teste com a ferramenta de desenvolvedor do Asset compute.

Certifique-se de ter acesso a um [contêiner de armazenamento em nuvem suportado](https://github.com/adobe/asset-compute-devtool#prerequisites). Esse contêiner pode ser compartilhado por vários desenvolvedores em diferentes projetos, conforme necessário.

#### Adicionar credenciais ao arquivo ENV {#add-credentials-env-file}

Adicione as seguintes credenciais para a ferramenta de desenvolvedor ao arquivo ENV na raiz do seu projeto do App Builder:

1. Adicione o caminho absoluto ao arquivo de chave privada criado ao adicionar serviços ao projeto do App Builder:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Baixe o arquivo no Adobe Developer Console. Vá para a raiz do projeto e clique em &quot;Baixar tudo&quot; no canto superior direito. O arquivo é baixado com `<namespace>-<workspace>.json` como o nome do arquivo. Siga uma das seguintes opções:

   * Renomeie o arquivo como `console.json` e mova-a para a raiz do seu projeto.
   * Opcionalmente, é possível adicionar o caminho absoluto ao arquivo JSON de integração do Console do Adobe Developer. É o mesmo [`console.json`](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) arquivo que é baixado no espaço de trabalho do projeto.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. Adicione as credenciais de armazenamento do S3 ou do Azure. Você só precisa acessar uma solução de armazenamento em nuvem.

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

>[!TIP]
>
>O `config.json` O arquivo contém credenciais. No seu projeto, adicione o arquivo JSON a `.gitignore` para impedir o compartilhamento. O mesmo se aplica aos arquivos .env e .aio.

## Execute o aplicativo {#run-custom-application}

Antes de executar o aplicativo com a ferramenta Asset compute Developer, configure corretamente a variável [credenciais](#developer-tool-credentials).

Para executar o aplicativo na ferramenta do desenvolvedor, use `aio app run` comando. Ele implanta a ação para [!DNL Adobe I/O] Tempo de execução e iniciar a ferramenta de desenvolvimento no computador local. Essa ferramenta é usada para testar solicitações de aplicativos durante o desenvolvimento. Veja um exemplo de solicitação de representação:

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
>Não utilize o `--local` sinalizador com `run` comando. Não funciona com [!DNL Asset Compute] aplicativos personalizados e a ferramenta Asset compute Developer. Os aplicativos personalizados são chamados pela função [!DNL Asset Compute Service] que não pode acessar ações executadas nos computadores locais do desenvolvedor.

Consulte [here](test-custom-application.md) como testar e depurar seu aplicativo. Quando terminar de desenvolver seu aplicativo personalizado, [implantar seu aplicativo personalizado](deploy-custom-application.md).

## Experimente o aplicativo de amostra fornecido pelo Adobe {#try-sample}

Veja a seguir exemplos de aplicativos personalizados:

* [trabalhadora básica](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [imagens de animais-de-estimação](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aplicativo personalizado de modelo {#template-custom-application}

O [trabalhadora básica](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) é um aplicativo de modelo. Ele gera uma representação simplesmente copiando o arquivo de origem. O conteúdo deste aplicativo é o modelo recebido ao escolher `Adobe Asset Compute` na criação do aplicativo aio.

O arquivo do aplicativo, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) usa a variável [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) para baixar o arquivo de origem, orquestrar cada processamento de representação e fazer upload das representações resultantes de volta para o armazenamento na nuvem.

O [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definido dentro do código do aplicativo, é onde executar toda a lógica de processamento do aplicativo. O retorno de chamada de representação em `worker-basic` simplesmente copia o conteúdo do arquivo de origem para o arquivo de representação.

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

Por exemplo, a variável [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) faz uma solicitação de busca para um URL estático da Wikimedia usando o [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) biblioteca.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Envio de parâmetros personalizados {#pass-custom-parameters}

Você pode passar parâmetros definidos personalizados por meio dos objetos de representação. Eles podem ser referenciados dentro do aplicativo em [`rendition` instruções](https://github.com/adobe/asset-compute-sdk#rendition). Um exemplo de objeto de representação é:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Um exemplo de um arquivo de aplicativo que acessa um parâmetro personalizado é:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

O `example-worker-animal-pictures` passa um parâmetro personalizado [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) para determinar qual arquivo buscar na Wikimedia.

## Suporte à autenticação e autorização {#authentication-authorization-support}

Por padrão, os aplicativos personalizados do Asset compute vêm com verificações de Autorização e Autenticação para o projeto do App Builder. Isso é ativado ao configurar a variável `require-adobe-auth` anotação para `true` no `manifest.yml`.

### Acesse outras APIs do Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Adicione os serviços de API ao [!DNL Asset Compute] Espaço de trabalho do console criado na configuração. Esses serviços fazem parte do token de acesso JWT gerado pelo [!DNL Asset Compute Service]. O token e outras credenciais são acessíveis dentro da ação do aplicativo `params` objeto.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Enviar credenciais para sistemas de terceiros {#pass-credentials-for-tp}

Para manipular credenciais para outros serviços externos, passe-as como parâmetros padrão nas ações. Eles são automaticamente criptografados em trânsito. Para obter mais informações, consulte [criação de ações no guia do desenvolvedor do Runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). Em seguida, defina-os usando variáveis de ambiente durante a implantação. Esses parâmetros podem ser acessados na função `params` dentro da ação.

Defina os parâmetros padrão dentro do `inputs` no `manifest.yml`:

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

O `$VAR` A expressão lê o valor de uma variável de ambiente chamada `VAR`.

Durante o desenvolvimento, o valor pode ser definido no arquivo ENV local como `aio` O lê automaticamente as variáveis de ambiente dos arquivos ENV, além das variáveis definidas no shell de chamada. Neste exemplo, o arquivo ENV tem a seguinte aparência:

```CONF
#...
SECRET_KEY=secret-value
```

Para implantação de produção, é possível definir as variáveis de ambiente no sistema CI, por exemplo, usando segredos em Ações do GitHub. Por fim, acesse os parâmetros padrão dentro do aplicativo como tal:

```javascript
const key = params.secretKey;
```

## Dimensionamento de aplicativos {#sizing-workers}

Um aplicativo é executado em um contêiner em [!DNL Adobe I/O] Tempo de execução com [limites](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) que pode ser configurado por meio do `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Devido ao processamento mais extenso normalmente feito por aplicativos Asset compute, é mais provável que seja necessário ajustar esses limites para desempenho ideal (grande o suficiente para lidar com ativos binários) e eficiência (não desperdiçar recursos devido à memória de contêiner não usada).

O tempo limite padrão para ações em Tempo de execução é de um minuto, mas pode ser aumentado ao configurar a variável `timeout` limite (em milissegundos). Se você espera processar arquivos maiores, aumente esse tempo. Considere o tempo total necessário para baixar a fonte, processar o arquivo e fazer upload da representação. Se uma ação expirar, ou seja, não retornar a ativação antes do tempo limite especificado, o Tempo de execução descarta o contêiner e não o reutilizará.

Por natureza, os aplicativos Asset compute tendem a ser de entrada ou saída de rede e disco. O arquivo de origem deve ser baixado primeiro, o processamento geralmente exige muitos recursos e, em seguida, as renderizações resultantes são carregadas novamente.

A memória disponível para um contêiner de ação é especificada por `memorySize` em MB. Atualmente, isso também define quanto acesso à CPU o contêiner recebe e, o mais importante, é um elemento essencial do custo de usar o Runtime (contêineres maiores custam mais). Use um valor maior aqui quando o processamento exigir mais memória ou CPU, mas tenha cuidado para não desperdiçar recursos, pois quanto maior for o contêiner, menor será a taxa de transferência geral.

Além disso, é possível controlar a simultaneidade de ações em um contêiner usando a variável `concurrency` configuração. Esse é o número de ativações simultâneas que um único contêiner (da mesma ação) recebe. Neste modelo, o contêiner de ação é como um servidor Node.js que recebe várias solicitações simultâneas, até esse limite. Se não estiver definido, o padrão em Tempo de execução é 200, o que é ótimo para ações menores do App Builder, mas geralmente é muito grande para aplicativos Asset compute, dado o processamento local mais intenso e a atividade do disco. Alguns aplicativos, dependendo de sua implementação, também podem não funcionar bem com atividades simultâneas. O SDK do Asset compute garante que as ativações sejam separadas, gravando arquivos em pastas exclusivas diferentes.

Teste os aplicativos para encontrar os números ideais para `concurrency` e `memorySize`. Contêineres maiores = limite de memória mais alto pode permitir mais simultaneidade, mas também pode ser um desperdício para tráfego menor.
