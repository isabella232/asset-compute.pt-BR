---
title: Solução de problemas [!DNL Asset Compute Service].
description: Solucione problemas e depure aplicativos personalizados usando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---


# Solução de problemas {#troubleshoot}

Algumas dicas genéricas de solução de problemas que podem ajudá-lo a solucionar problemas com o Serviço de Asset compute são:

* Certifique-se de que o aplicativo JavaScript não trave na inicialização. Tais falhas normalmente estão relacionadas a uma biblioteca ausente ou a uma dependência.
* Verifique se todas as dependências a serem instaladas são referenciadas no arquivo `package.json` do aplicativo.
* Certifique-se de que todos os erros que possam vir da limpeza na falha não gerem seus próprios erros que ocultam o problema original.

* Ao iniciar a ferramenta do desenvolvedor pela primeira vez com uma nova integração [!DNL Asset Compute Service], ela pode falhar na primeira solicitação de processamento porque o Journal de Eventos do Asset compute pode não estar completamente configurado. Aguarde até que o journal seja configurado antes de enviar outra solicitação.
* Se você encontrar erros ao enviar solicitações de Asset compute `/register` ou `/process`, verifique se todas as APIs necessárias foram adicionadas ao [!DNL Adobe I/O] Projeto e Espaço de trabalho, ou seja, Asset compute, Eventos de E/S, Gerenciamento de Eventos de E/S e Tempo de execução.

## Fazer logon em problemas via [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Se você tiver problemas ao fazer logon em [!DNL Adobe Developer Console] [pelo [!DNL Adobe I/O] CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli), adicione manualmente as credenciais necessárias para desenvolver, testar e implantar seu aplicativo personalizado:

1. Navegue até o projeto e o espaço de trabalho do Firefly no [Console do desenvolvedor do Adobe](https://console.adobe.io/) e pressione **[!UICONTROL Download]** no canto superior direito. Abra este arquivo e salve este JSON em um local seguro na sua máquina.

1. Navegue até o arquivo ENV em seu aplicativo Firefly.

1. Adicione as credenciais [!DNL Adobe I/O] do tempo de execução. Obtenha as credenciais [!DNL Adobe I/O] de um tempo de execução do JSON baixado. As credenciais estão em `project.workspace.services.runtime`. Adicione as credenciais [!DNL Adobe I/O] do tempo de execução nas variáveis `AIO_runtime_XXX`:

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
