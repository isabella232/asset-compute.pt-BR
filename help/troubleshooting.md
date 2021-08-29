---
title: Solução de problemas [!DNL Asset Compute Service]
description: Solucione problemas e depure aplicativos personalizados usando [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '285'
ht-degree: 1%

---

# Solução de problemas {#troubleshoot}

Algumas dicas genéricas de solução de problemas que podem ajudá-lo a solucionar problemas com o Asset compute Service são:

* Certifique-se de que o aplicativo JavaScript não trave na inicialização. Normalmente, essas falhas estão relacionadas a uma biblioteca ausente ou a uma dependência.
* Certifique-se de que todas as dependências a serem instaladas estejam referenciadas no arquivo `package.json` do aplicativo.
* Certifique-se de que todos os erros que possam vir da limpeza na falha não gerem seus próprios erros que ocultam o problema original.

* Ao iniciar a ferramenta do desenvolvedor pela primeira vez com uma nova integração [!DNL Asset Compute Service], ela pode falhar na primeira solicitação de processamento se o Diário de eventos do Asset compute não estiver totalmente configurado. Aguarde algum tempo para que o diário seja configurado antes de enviar outra solicitação.
* Se tiver erros ao enviar solicitações Asset compute `/register` ou `/process`, verifique se todas as APIs necessárias foram adicionadas ao [!DNL Adobe I/O] Projeto e Espaço de trabalho, ou seja, Asset compute, [!DNL Adobe I/O] Eventos, [!DNL Adobe I/O] Gerenciamento de eventos e [!DNL Adobe I/O] Tempo de execução.

## Fazer logon em problemas por meio da CLI [!DNL Adobe I/O] {#login-via-aio-cli}

Se tiver problemas ao fazer logon no [!DNL Adobe Developer Console] [por meio do  [!DNL Adobe I/O] CLI](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli), adicione manualmente as credenciais necessárias para desenvolver, testar e implantar seu aplicativo personalizado:

1. Navegue até o projeto e o espaço de trabalho do Firefly no [Adobe Developer Console](https://console.adobe.io/) e pressione **[!UICONTROL Download]** no canto superior direito. Abra este arquivo e salve este JSON em um local seguro no computador.

1. Navegue até o arquivo ENV em seu aplicativo Firefly.

1. Adicione as [!DNL Adobe I/O] Credenciais de Tempo de Execução. Obtenha as [!DNL Adobe I/O] credenciais de tempo de execução do JSON baixado. As credenciais estão em `project.workspace.services.runtime`. Adicione as credenciais de [!DNL Adobe I/O] Tempo de execução nas variáveis `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Adicione o caminho absoluto ao JSON baixado na Etapa 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure o restante das [credenciais necessárias](develop-custom-application.md) necessárias para a ferramenta do desenvolvedor.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
