# Registro de Verdade — PWA

Esta versão transforma a interface recebida em um Progressive Web App (PWA), instalável na tela inicial do iPhone e do Android. A interface é publicada como site estático; a sincronização é feita diretamente pela Google Sheets API após autorização OAuth no próprio aparelho. O aplicativo não usa mais `google.script.run` e não depende de abrir a página HTML do Apps Script no celular.

## O que foi preservado

- Controle semanal com nove perguntas principais, observações, pergunta sobre perceber o impulso de esconder/distorcer/omitir e análise opcional de causa raiz por dia.
- Registros rápidos com análise guiada: situação, verdade completa, tentativa de fuga, emoções, pensamento automático, o que tentava proteger, ganho imediato, custo posterior, hipótese de causa, cinco porquês e reparação.
- Relatório semanal com padrões de fuga, emoções e fatores que a pessoa tentou proteger.
- Histórico, exclusão local e backup JSON com imagens.
- Armazenamento local IndexedDB em cada dispositivo.
- Sincronização de respostas, observações e registros rápidos com a aba `Registros`.
- Mesma estrutura da planilha antiga: `key`, `type`, `date`, `updatedAt`, `payload`.
- Fotos continuam locais e também entram nos backups exportados; não são enviadas à planilha.
- O relatório automático por e-mail continua no Apps Script, usando o arquivo separado `EmailSemanal-causa-raiz.gs`. Não publique esse arquivo no GitHub: ele contém os destinatários do relatório.

## Antes de migrar: proteja os registros locais

1. No aplicativo antigo que ainda abre no computador, entre em **Histórico e backup → Exportar backup**.
2. Guarde o arquivo JSON em local seguro. Essa etapa é importante porque o IndexedDB do endereço antigo não é compartilhado com o novo domínio.
3. Não exclua nem altere o projeto Apps Script `registro v` nem a planilha existente.

Quando a PWA estiver publicada, conecte o Google e selecione a planilha **Registro de Verdade - Dados Privados**, depois importe o backup antigo em **Histórico e backup** e clique em **Sincronizar agora**. A sincronização compara `updatedAt` para evitar substituir um registro remoto mais novo por uma cópia local mais antiga.

## Publicar no GitHub Pages

1. Crie um repositório no GitHub, por exemplo `controle-de-verdade-pwa`.
2. Envie somente os arquivos desta pasta para a raiz do repositório: `index.html`, `manifest.webmanifest`, `service-worker.js`, `LEIA-ME.md` e a pasta `icons`. O arquivo `EmailSemanal-atualizado.gs` é separado e deve ser copiado diretamente para o Apps Script, nunca para o repositório público.
3. No repositório, entre em **Settings → Pages**.
4. Em **Build and deployment**, escolha **Deploy from a branch**, branch `main`, pasta `/ (root)` e salve.
5. Aguarde a publicação e copie o endereço HTTPS exibido pelo GitHub Pages. Ele será semelhante a `https://SEU-USUARIO.github.io/controle-de-verdade-pwa/`.

Documentação GitHub Pages: https://docs.github.com/en/pages

## Configurar acesso seguro ao Google (uma vez)

O aplicativo usa `drive.file`, escopo recomendado pelo Google para permitir acesso apenas aos arquivos que o usuário escolheu ou abriu com o aplicativo. O Google Picker seleciona a planilha específica. Não é necessário publicar a planilha na internet.

1. Abra https://console.cloud.google.com/ e crie um projeto Google Cloud.
2. Em **APIs & Services → Library**, habilite **Google Sheets API** e **Google Picker API**.
3. Em **Google Auth Platform** (ou **APIs & Services → OAuth consent screen**, conforme a interface), configure a tela de consentimento. Durante os testes, escolha público externo e adicione a conta Google proprietária da planilha como **Test user**.
4. Em **Credentials**, crie um **OAuth client ID** do tipo **Web application**.
5. Em **Authorized JavaScript origins**, adicione somente a origem do GitHub Pages, por exemplo `https://SEU-USUARIO.github.io` (sem `/controle-de-verdade-pwa/`).
6. Crie uma **API key**. Restrinja a chave à **Google Picker API** e configure a restrição HTTP referrer para o endereço do site, por exemplo `https://SEU-USUARIO.github.io/controle-de-verdade-pwa/*`.
7. Copie o **Project number** (número do projeto, não o nome e não o ID do projeto), o **OAuth Client ID** e a **API key**.
8. Abra o PWA, entre na aba **Conta Google e nuvem**, preencha os três valores, salve e clique em **Conectar conta Google**.
9. Clique em **Escolher planilha** e selecione a planilha já existente **Registro de Verdade - Dados Privados**. O aplicativo valida o cabeçalho da aba `Registros` antes de gravar qualquer coisa.

Referências oficiais:
- OAuth para aplicações web: https://developers.google.com/identity/oauth2/web/guides/use-token-model
- Escopos do Sheets API: https://developers.google.com/workspace/sheets/api/scopes
- Google Picker: https://developers.google.com/drive/picker/guides/overview
- Sheets API — leitura e escrita: https://developers.google.com/workspace/sheets/api/guides/values

## Instalar no iPhone

1. Abra o endereço HTTPS do GitHub Pages no **Safari**.
2. Toque em **Compartilhar**.
3. Escolha **Adicionar à Tela de Início** e confirme.
4. Abra pelo novo ícone **Verdade**.
5. Entre em **Conta Google e nuvem**, conecte a conta e escolha a mesma planilha.

No computador, use o mesmo endereço. Em cada aparelho, a autorização Google é feita naquele navegador/aparelho; os dados ficam na mesma planilha.

## Relatório automático de segunda-feira

A PWA grava diretamente na mesma aba `Registros`. O projeto Apps Script continua necessário apenas para o e-mail semanal. No arquivo `EmailSemanal.gs`, o acionador atualizado está configurado para a janela das **10h às 11h** (horário `America/Sao_Paulo`). No projeto `registro v`, substitua o conteúdo de `EmailSemanal.gs` pelo arquivo separado `EmailSemanal-atualizado.gs`. Depois execute **uma vez** `setupWeeklyReportTrigger()` para apagar o acionador antigo e criar o novo. O horário `atHour(10)` não garante disparo exatamente às 10h em ponto.

## Segurança e limitações

- Não coloque senha, `APP_ACCESS_KEY`, segredo OAuth ou chave privada neste repositório.
- O OAuth Client ID e a API key são identificadores públicos; restrinja a API key por API e HTTP referrer.
- O acesso ao conteúdo da planilha depende do login Google e da seleção explícita do arquivo no Picker.
- O escopo `drive.file` é limitado aos arquivos escolhidos/abertos com este app; não usa o escopo amplo `spreadsheets`.
- A autorização fornece um token temporário mantido em memória. Ao reabrir a PWA, toque em **Conectar conta Google** novamente se a sessão não estiver ativa.
- Os dados locais e as imagens continuam no aparelho. Para trocar de aparelho, importe um backup e sincronize; textos e respostas também vêm da planilha.
- A exclusão na tela de histórico remove a cópia local. Para manter a consistência entre dispositivos, revise o histórico sincronizado antes de excluir; a versão atual não propaga exclusões para a planilha.
