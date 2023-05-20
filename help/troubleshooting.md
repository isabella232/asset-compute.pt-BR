---
title: Solução de problemas [!DNL Asset Compute Service]
description: Solucionar problemas e depurar aplicativos personalizados usando o [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Solução de problemas {#troubleshoot}

Algumas dicas de solução de problemas genéricas que podem ajudar você a solucionar problemas com o Asset compute Service são:

* Certifique-se de que o aplicativo JavaScript não falhe na inicialização. Essas falhas geralmente estão relacionadas a uma biblioteca ausente ou a uma dependência.
* Verifique se todas as dependências que serão instaladas estão referenciadas no `package.json` arquivo.
* Verifique se os erros que possam vir da limpeza na falha não geram seus próprios erros que ocultam o problema original.

* Ao iniciar a ferramenta de desenvolvedor pela primeira vez com uma nova [!DNL Asset Compute Service] integração, a primeira solicitação de processamento poderá falhar se o Diário de eventos do Asset compute não estiver totalmente configurado. Aguarde algum tempo para que o diário seja configurado antes de enviar outra solicitação.
* Se você encontrar erros ao enviar o Asset compute `/register` ou `/process` solicitações, verifique se todas as APIs necessárias foram adicionadas à [!DNL Adobe I/O] Project e Workspace — ou seja, Asset compute, [!DNL Adobe I/O] Eventos, [!DNL Adobe I/O] Gerenciamento de eventos e [!DNL Adobe I/O] Tempo de execução.

## Problemas de logon via [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Se tiver problemas ao fazer logon na [!DNL Adobe Developer Console] [por meio da [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), em seguida, adicione manualmente as credenciais necessárias para desenvolver, testar e implantar seu aplicativo personalizado:

1. Navegue até o projeto e o espaço de trabalho do Adobe Developer App Builder na [Console do Adobe Developer](https://console.adobe.io/) e pressione **[!UICONTROL Baixar]** no canto superior direito. Abra este arquivo e salve este JSON em um local seguro em sua máquina.

1. Navegue até o arquivo ENV no aplicativo Adobe Developer App Builder.

1. Adicione o [!DNL Adobe I/O] Credenciais de tempo de execução. Obtenha o [!DNL Adobe I/O] As credenciais de Tempo de execução do JSON baixado. As credenciais estão em `project.workspace.services.runtime`. Adicione o [!DNL Adobe I/O] Credenciais de tempo de execução na `AIO_runtime_XXX` variáveis:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Adicione o caminho absoluto ao JSON baixado na Etapa 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure o restante da [credenciais necessárias](develop-custom-application.md) necessário para a ferramenta de desenvolvedor.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
