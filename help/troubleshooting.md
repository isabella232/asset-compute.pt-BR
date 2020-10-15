---
title: Solução de problemas [!DNL Asset Compute Service].
description: Solucione problemas e depure aplicativos personalizados usando [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# Solução de problemas {#troubleshoot}

Algumas dicas genéricas de solução de problemas que podem ajudá-lo a solucionar problemas com o Asset Compute Service são:

* Certifique-se de que o aplicativo JavaScript não trave na inicialização. Tais falhas normalmente estão relacionadas a uma biblioteca ausente ou a uma dependência.
* Certifique-se de que todas as dependências a serem instaladas sejam referenciadas no `package.json` arquivo do aplicativo.
* Certifique-se de que todos os erros que possam vir da limpeza na falha não gerem seus próprios erros que ocultam o problema original.

* Ao iniciar a ferramenta para desenvolvedores pela primeira vez com uma nova [!DNL Asset Compute Service] integração, ela pode falhar na primeira solicitação de processamento porque o Journal Eventos Asset Compute pode não estar completamente configurado. Aguarde até que o journal seja configurado antes de enviar outra solicitação.
* Se você encontrar erros ao enviar a Computação de ativos `/register` ou `/process` solicitações, certifique-se de que todas as APIs necessárias foram adicionadas ao Projeto de E/S do Adobe e à Área de trabalho, ou seja, Computação de ativos, Eventos de E/S, Gerenciamento de Eventos de E/S e Tempo de execução.

## Problemas de login por meio da CLI de E/S do Adobe {#login-via-aio-cli}

Se tiver problemas ao fazer logon [!DNL Adobe Developer Console] pela CLI [](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)de E/S do Adobe, adicione manualmente as credenciais necessárias para desenvolver, testar e implantar seu aplicativo personalizado:

1. Navegue até o projeto e o espaço de trabalho do Firefly no [Adobe Developer Console](https://console.adobe.io/) e pressione **[!UICONTROL Download]** no canto superior direito. Abra este arquivo e salve este JSON em um local seguro na sua máquina.

1. Navegue até o arquivo ENV em seu aplicativo Firefly.

1. Adicione as credenciais do Adobe I/O Runtime. Obtenha as credenciais do Adobe I/O Runtime do JSON baixado. As credenciais estão embaixo `project.workspace.services.runtime`. Adicione as credenciais de I/O Runtime nas `AIO_runtime_XXX` variáveis:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Adicione o caminho absoluto ao JSON baixado na Etapa 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure o restante das credenciais [](develop-custom-application.md) necessárias para a ferramenta de desenvolvedor.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
