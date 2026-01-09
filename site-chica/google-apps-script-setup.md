# Configuração do Google Apps Script para o Formulário

## Passo 1: Criar a Planilha no Google Sheets

1. Acesse https://sheets.google.com
2. Crie uma nova planilha
3. Renomeie para "Contatos Rafael Chicaroni"
4. Na primeira linha, adicione os cabeçalhos:
   - **A1**: Data/Hora
   - **B1**: Nome
   - **C1**: E-mail
   - **D1**: Telefone
   - **E1**: Demanda

## Passo 2: Criar o Google Apps Script

1. Na planilha, clique em **Extensões** > **Apps Script**
2. Delete o código padrão
3. Cole o código abaixo:

```javascript
function doPost(e) {
  try {
    // Parse os dados recebidos
    const data = JSON.parse(e.postData.contents);

    // Pega a planilha ativa
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // Formata a data
    const timestamp = new Date(data.timestamp);
    const formattedDate = Utilities.formatDate(timestamp, "America/Sao_Paulo", "dd/MM/yyyy HH:mm:ss");

    // Adiciona os dados na planilha
    sheet.appendRow([
      formattedDate,
      data.name,
      data.email,
      data.phone,
      data.demand
    ]);

    // Envia e-mail de notificação
    const emailBody = `
Nova mensagem recebida do site!

Nome: ${data.name}
E-mail: ${data.email}
Telefone: ${data.phone}

Demanda:
${data.demand}

---
Data/Hora: ${formattedDate}
    `.trim();

    MailApp.sendEmail({
      to: 'rafaelchicaroni.fotografo@gmail.com',
      subject: `🔔 Novo Contato no Site: ${data.name}`,
      body: emailBody
    });

    return ContentService.createTextOutput(JSON.stringify({
      status: 'success',
      message: 'Dados salvos com sucesso'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}
```

## Passo 3: Fazer Deploy do Script

1. Clique no botão **Implantar** (Deploy) > **Nova implantação** (New deployment)
2. Clique no ícone de engrenagem ⚙️ > Selecione **Aplicativo da Web** (Web app)
3. Configure:
   - **Descrição**: "Formulário de Contato Site Rafael"
   - **Executar como**: Eu (sua conta)
   - **Quem tem acesso**: Qualquer pessoa
4. Clique em **Implantar** (Deploy)
5. **IMPORTANTE**: Copie a URL do Web App que aparece (algo como `https://script.google.com/macros/s/xxxxx/exec`)
6. Clique em **Concluído**

## Passo 4: Autorizar o Script

1. Você verá uma mensagem pedindo autorização
2. Clique em **Revisar permissões**
3. Escolha sua conta Google
4. Clique em **Avançado** > **Ir para [nome do projeto] (não seguro)**
5. Clique em **Permitir**

## Passo 5: Adicionar a URL no Site

1. Copie a URL do Web App que você recebeu no Passo 3
2. Abra o arquivo `index.html`
3. Procure pela linha:
   ```javascript
   const SCRIPT_URL = 'COLE_AQUI_A_URL_DO_SEU_GOOGLE_APPS_SCRIPT';
   ```
4. Substitua por:
   ```javascript
   const SCRIPT_URL = 'https://script.google.com/macros/s/xxxxx/exec';
   ```
   (Use a URL que você copiou)

## Passo 6: Testar

1. Faça commit e push das alterações
2. Aguarde o deploy no Vercel
3. Acesse o site e preencha o formulário
4. Verifique se:
   - Os dados apareceram na planilha
   - Você recebeu o e-mail em rafaelchicaroni.fotografo@gmail.com

## Solução de Problemas

### E-mail não chegou?
- Verifique a pasta de spam
- Confirme que autorizou o script a enviar e-mails

### Dados não aparecem na planilha?
- Abra o Apps Script e vá em **Execuções** para ver os logs de erro
- Verifique se a URL do script está correta no index.html

### Erro CORS no navegador?
- Isso é normal! Usamos `mode: 'no-cors'` justamente por isso
- O formulário vai funcionar mesmo com esse aviso no console

---

✅ **Pronto!** Seu formulário está configurado e funcionando.
