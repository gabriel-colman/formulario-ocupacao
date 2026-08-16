# Levantamento de Ocupação Máxima — SENAI/MS

Sistema web para cadastro de ambientes físicos, armazenamento automático de imagens no Google Drive e registro estruturado no Google Sheets.

---

## Acesso ao Sistema

### Aplicação publicada (GitHub Pages)

🔗 https://gabriel-colman.github.io/formulario-ocupacao/formulario-ocupacao.html

---

## Tecnologias Utilizadas

- HTML5
- CSS3
- JavaScript
- GitHub Pages
- Google Apps Script
- Google Drive API (via Apps Script)
- Google Sheets API (via Apps Script)

---

## Arquitetura do Projeto

```mermaid
graph LR

A[Usuário]
--> B[Frontend HTML / GitHub Pages]

B --> C[Apps Script Web App]

C --> D[Google Drive]
C --> E[Google Sheets]
```

---

## Fluxo de Funcionamento

### 1 — Usuário preenche o formulário

Informações coletadas:

- Unidade SENAI
- Responsável
- Cargo
- Data
- Nome do espaço
- Tipo do espaço
- Capacidade
- Área
- Equipamentos
- Acessibilidade
- Observações
- Upload de imagens

---

### 2 — Frontend envia os dados

O HTML utiliza JavaScript (`fetch`) para enviar um payload JSON para o backend Apps Script.

Exemplo simplificado:

```javascript
fetch(SCRIPT_URL,{
    method:'POST',
    body: JSON.stringify(payload)
})
```

---

### 3 — Backend Apps Script processa

O Apps Script:

- recebe o JSON;
- cria pasta de protocolo no Drive;
- cria subpastas por espaço;
- salva imagens;
- gera links compartilháveis;
- registra os dados na planilha.

---

### 4 — Google Drive

Estrutura criada automaticamente:

```text
Ocupacao/
└── FOTOS/
    └── OC-20260601-103815 — Unidade
        └── 1 - Sala de Aula
            ├── foto1.jpg
            ├── foto2.jpg
```

---

### 5 — Google Sheets

Os registros são armazenados automaticamente.

Estrutura atual:

| Coluna | Campo |
|--------|--------|
| A | Unidade |
| B | Responsável |
| C | Cargo |
| D | DataLevantamento |
| E | NomeEspaco |
| F | TipoEspaco |
| G | BlocoPavimento |
| H | Capacidade |
| I | Area |
| J | Equipamentos |
| K | Acessibilidade |
| L | Status |
| M | Observacoes |
| N | Tipo de Item |
| O | Caminho |

---

## Backend — Google Apps Script

Backend desenvolvido em Google Apps Script.

Responsabilidades:

✔ Receber payload JSON

✔ Processar upload Base64

✔ Criar pastas automaticamente

✔ Salvar imagens no Drive

✔ Registrar dados na planilha

✔ Gerar links dos arquivos

---

## Configuração do Apps Script

Publicação:

```text
Implantar
→ Nova implantação
→ Aplicativo da Web
→ Executar como: Eu
→ Quem possui acesso: Qualquer pessoa
```

---

## Painel Administrativo

🔗 https://gabriel-colman.github.io/formulario-ocupacao/admin.html

Página separada (`admin.html`) para corrigir registros já enviados: editar
campos, e adicionar/substituir/remover fotos (útil para os casos de
registros sem foto ou com fotos duplicadas). Acesso restrito por login
Google — o backend verifica o token e confere o e-mail contra uma lista de
administradores autorizados antes de aceitar qualquer alteração.

### Configuração (passo único, feito fora do repositório)

1. **Criar um OAuth Client ID** no [Google Cloud Console](https://console.cloud.google.com/apis/credentials):
   - Tipo: *Web application*.
   - *Authorized JavaScript origins*: `https://gabriel-colman.github.io`.
   - Copiar o Client ID gerado (formato `algo.apps.googleusercontent.com`).
2. Colar esse Client ID na constante `GOOGLE_CLIENT_ID` no topo do
   `admin.html`.
3. No editor do Apps Script → **Configurações do projeto** → **Propriedades
   do script**, adicionar:
   - `GOOGLE_CLIENT_ID` → o mesmo Client ID do passo 1.
   - `ADMIN_EMAILS` → e-mails autorizados, separados por vírgula.
4. **Implantar → Gerenciar implantações → editar (ícone de lápis) → Nova
   versão** — atualiza o código sem trocar a `SCRIPT_URL` já usada pelo
   formulário público.
5. Rodar a função `backfillIdsEPastas()` **uma vez**, manualmente, pelo
   editor do Apps Script (selecionar a função → Executar). Ela gera um ID
   único e resolve a pasta do Drive de cada um dos registros já existentes,
   necessário para o painel saber o que editar. É idempotente — pode rodar
   de novo sem duplicar nada.

### Como funciona

- O `doPost` do Apps Script agora aceita um campo `action` no payload.
  Sem `action` (ou `action: "submeter"`) segue exatamente o fluxo público
  de sempre, sem autenticação — isso não mudou.
- Ações administrativas (`listarRegistros`, `atualizarRegistro`,
  `adicionarFoto`, `substituirFoto`, `removerFoto`) exigem um `idToken`
  válido (JWT do Google Sign-In), verificado no servidor contra o endpoint
  `oauth2.googleapis.com/tokeninfo` e contra a lista `ADMIN_EMAILS`.
- Toda alteração feita pelo painel é registrada na aba `LogAdmin` da
  planilha (quem, quando, o quê).
- Fotos removidas ou substituídas vão para a lixeira do Drive (recuperáveis
  por 30 dias), nunca são apagadas na hora.

---

## Problemas Encontrados

### CORS — GitHub Pages × Apps Script

Problema identificado:

Frontend hospedado no GitHub Pages bloqueava a resposta do Apps Script.

Solução aplicada:

```javascript
headers:{
    'Content-Type':'text/plain;charset=utf-8'
}
```

e ajuste no tratamento do `fetch()`.

---

## Estrutura do Repositório

```text
formulario-ocupacao/
│
├── formulario-ocupacao.html
├── admin.html
├── relatorio_ocupacao.html
├── README.md
```

---

## Autor

**Gabriel Colman Rodrigues**

GitHub:

https://github.com/gabriel-colman

---

## Observações

Projeto desenvolvido para gerenciamento de levantamento de ocupação máxima de ambientes educacionais do SENAI/MS.