---
title: Solução de problemas [!DNL Asset Compute Service]
description: Solucionar problemas e depurar aplicativos personalizados usando [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Solução de problemas {#troubleshoot}

Algumas dicas genéricas de solução de problemas que podem ajudá-lo a solucionar problemas com o Asset compute Service são:

* Certifique-se de que o aplicativo JavaScript não trave na inicialização. Normalmente, essas falhas estão relacionadas a uma biblioteca ausente ou a uma dependência.
* Certifique-se de que todas as dependências a serem instaladas estejam referenciadas no `package.json` arquivo.
* Certifique-se de que todos os erros que possam vir da limpeza na falha não gerem seus próprios erros que ocultam o problema original.

* Ao iniciar a ferramenta do desenvolvedor pela primeira vez com um novo [!DNL Asset Compute Service] integração, pode ocorrer uma falha na primeira solicitação de processamento se o Asset compute Events Journal não estiver completamente configurado. Aguarde algum tempo para que o diário seja configurado antes de enviar outra solicitação.
* Se você tiver erros ao enviar o Asset compute `/register` ou `/process` , verifique se todas as APIs necessárias foram adicionadas ao [!DNL Adobe I/O] Projeto e espaço de trabalho, ou seja, Asset compute, [!DNL Adobe I/O] Eventos, [!DNL Adobe I/O] Gerenciamento de eventos e [!DNL Adobe I/O] Tempo de execução.

## Problemas de logon via [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Se tiver problemas ao fazer logon na [!DNL Adobe Developer Console] [através da [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), em seguida, adicione manualmente as credenciais necessárias para desenvolver, testar e implantar seu aplicativo personalizado:

1. Navegue até o projeto e a área de trabalho do Adobe Developer App Builder no [Console do Adobe Developer](https://console.adobe.io/) e pressione **[!UICONTROL Baixar]** no canto superior direito. Abra este arquivo e salve este JSON em um local seguro no computador.

1. Navegue até o arquivo ENV no aplicativo Adobe Developer App Builder.

1. Adicione o [!DNL Adobe I/O] Credenciais de Tempo de Execução. Obtenha o [!DNL Adobe I/O] As credenciais do Runtime do JSON baixado. As credenciais estão em `project.workspace.services.runtime`. Adicione o [!DNL Adobe I/O] Credenciais de tempo de execução na `AIO_runtime_XXX` variáveis:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Adicione o caminho absoluto ao JSON baixado na Etapa 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure o restante da variável [credenciais necessárias](develop-custom-application.md) necessário para a ferramenta do desenvolvedor.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
