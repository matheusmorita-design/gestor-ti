# Sistema de Gestão de T.I. (Flask + SQLAlchemy)

## 🚀 Apresentação

Este é um painel web (dashboard) interno construído em Python (Flask) para a gestão completa dos serviços e ativos de T.I.

O sistema evoluiu de um simples leitor de chamados do Google Sheets para uma aplicação de ITSM (IT Service Management) robusta, agora com foco em **IT Asset Management (ITAM)** e **Software Asset Management (SAM)**. Ele utiliza uma arquitetura híbrida:

* **Google Forms/Sheets:** Usado como "caixa de entrada" para novos chamados abertos por utilizadores.
* **Base de Dados SQL (SQLite + SQLAlchemy):** Usada como o "cérebro" do sistema, gerindo todos os chamados, ativos, suprimentos, fornecedores, licenças, contratos e artigos da base de conhecimento.

O sistema inclui autenticação segura via Google Workspace (limitado ao seu domínio) e uma interface **responsiva (mobile-first)** que agora utiliza um **Menu Hambúrguer Fixo** para otimizar a navegação em todas as telas, dada a expansão de módulos.

## ✨ Funcionalidades Principais

Este painel centraliza diversas operações críticas da equipa de T.I.:

### 1. Gestão de Chamados (Service Desk)

* **Sincronização Centralizada:** Utiliza um **Mapeamento Centralizado** para sincronizar novos chamados da planilha Google Sheets para a base de dados SQL, lidando com inconsistências de nomes de colunas e garantindo a integridade dos dados.
* **Fila de Trabalho (Chamados Ativos):** A página principal exibe apenas chamados que exigem ação (ex: "Aberto", "Em Andamento").
* **Livro Mestre (Histórico):** Arquiva todos os chamados "Finalizados" para análise e consulta futura (coluna de data de finalização corrigida para consistência).
* **Interface "Carrinho de Compras":** Permite vincular **múltiplos ativos** e **múltiplos suprimentos** (com quantidade) a um único chamado.

### 2. Gestão de Ativos e Infraestrutura (ITAM/SAM)

O inventário foi expandido em cinco categorias principais:

* **Ativos (Hardware/ITAM):** Gestão (CRUD) de itens únicos e rastreáveis (notebooks, desktops, etc.). O modelo foi aprimorado para incluir rastreamento de **Garantia**, **Custo**, **Data de Aquisição** e **Configurações de Hardware**.
* **Suprimentos:** Gestão (CRUD) de itens quantificáveis (toners, ratos, cabos) com controlo de stock (Quantidade Atual e Quantidade Mínima).
* **[NOVO] Fornecedores:** Módulo centralizado (CRUD) para registar e gerir contatos, e-mails e notas. É a **fonte primária de dados** para os módulos de Ativos, Licenças e Contratos, garantindo a integridade da chave estrangeira.
* **[NOVO] Licenças (SAM):** Módulo para registar e controlar software, chaves de acesso, datas de início/fim e custos. Inclui **Alertas de Renovação** por e-mail para licenças que se aproximam da data de término.
* **[NOVO] Contratos:** Módulo para gerir contratos de serviços (suporte, telecom, cloud), rastreando ID, datas de início/fim, fornecedor e link para a documentação. Inclui **Alertas de Encerramento** por e-mail.

### 3. Automação de Processos e Auditoria

* **Movimentação Inteligente:** Ao finalizar um chamado com um ativo do "Estoque", o sistema **automaticamente** altera o status desse ativo para "Em uso" e atribui-o ao solicitante do chamado.
* **Baixa Automática de Estoque:** Ao finalizar um chamado, o sistema **automaticamente** deduz a quantidade de suprimentos do stock principal.
* **Trilha de Auditoria:** Todas as movimentações automáticas são **registadas e anexadas** ao campo "Observação" do chamado finalizado.

### 4. Base de Conhecimento (Knowledge Base)

* Módulo completo para criar, editar e apagar artigos de procedimentos internos.
* Os artigos são organizados por Categoria e Autor.
* Controlo de Visibilidade ("TI Interno" ou "Público").

### 5. Experiência do Utilizador (UX)

* **Navegação Fixa:** Implementação do **Menu Offcanvas Fixo** (Menu Hambúrguer) em todas as larguras de tela para simplificar a navegação nos **8 módulos** existentes.
* **Autenticação Segura:** Login via Google OAuth, restrito a utilizadores do seu domínio.
* **Busca Inteligente:** Utiliza `Tom Select` (no carrinho de compras) e `filter.js` (filtros em tempo real em dashboards) para pesquisa rápida e dinâmica.
* **Modo Escuro/Claro:** Trocador de tema com persistência no `localStorage`.

---

## 🛠️ Tecnologias Utilizadas

* **Backend:** Python 3, Flask, SQLAlchemy (ORM)
* **Base de Dados:** SQLite (pode ser facilmente migrado para PostgreSQL ou MySQL)
* **Frontend:** Bootstrap 5, JavaScript (Puro / Vanilla), Tom Select, **CSS Customizado** (para Menu Fixo e Estilos de Alerta)
* **APIs & Auth:** Google Sheets API, Google Drive API, Google OAuth 2.0 (Flask-Dance)
* **Bibliotecas Python:** `gspread`, `Flask-Login`, `python-dotenv`

---

## 📂 Estrutura do Projeto

/gestor-ti/ 
├── instance/ 
│ └── sistema_ti.db <-- (A sua base de dados SQL) 
├── static/ 
│ ├── css/ 
│ │ └── style.css <-- (Folha de estilo global, **AGORA COM MENU FIXO**) 
│ └── js/ 
│ ├── main.js 
│ ├── page.assets.js 
│ ├── page.contracts.js <-- (Novo script de Contratos)
│ ├── page.licenses.js <-- (Novo script de Licenças)
│ ├── page.suppliers.js <-- (Novo script de Fornecedores)
│ └── filter.js 
├── templates/ 
│ ├── assets.html 
│ ├── contracts.html 
│ ├── index.html 
│ ├── kb.html 
│ ├── licenses.html 
│ ├── logbook.html 
│ ├── suppliers_mgmt.html <-- (Novo template Fornecedores)
│ ├── kb_form.html 
│ └── supplies.html 
├── .env <-- (Ficheiro de configuração. IMPORTANTE) 
├── app.py <-- (O "cérebro" da aplicação: **Com Mapeamento Centralizado e Alertas**) 
├── models.py <-- (A "planta" da base de dados: **Com 8 modelos centrais e chaves estrangeiras**) 
├── requirements.txt 
└── README.md <-- (Este ficheiro)

---

## ⚙️ Guia de Instalação e Execução

### 1. Pré-requisitos (Ficheiros JSON)

Antes de executar, você **DEVE** gerar dois ficheiros de credenciais no [Google Cloud Console](https://console.cloud.google.com/):

1.  **`credentials.json` (Conta de Serviço):**
    * Crie uma **Conta de Serviço**.
    * Ative as APIs **Google Sheets** e **Google Drive**.
    * Gere uma chave JSON e renomeie-a para `credentials.json`.
    * **Importante:** Partilhe a sua Planilha Google Sheets com o `client_email` encontrado dentro deste JSON (dando permissão de **Editor**).

2.  **`oauth_client_secret.json` (ID do Cliente OAuth):**
    * Configure a **"Tela de permissão OAuth"** (tipo "Interno" para Google Workspace).
    * **[IMPORTANTE]** Na secção "Escopos", clique em "Adicionar ou Remover Escopos" e adicione manualmente os três seguintes: `.../auth/userinfo.email`, `.../auth/userinfo.profile`, `openid`.
    * Crie um **"ID do cliente OAuth 2.0"** do tipo "Aplicativo da Web".
    * Adicione as **URIs de redirecionamento autorizados** (ex: `http://127.0.0.1:5001/auth/google/callback` e `http://localhost:5001/auth/google/callback`).
    * Baixe o JSON e renomeie-o para `oauth_client_secret.json`.

### 2. Configuração do Ambiente

1.  **Crie e ative um ambiente virtual:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # (Linux/macOS)
    .\venv\Scripts\activate   # (Windows)
    ```

2.  **Instale as dependências:**
    ```bash
    pip install -r requirements.txt
    ```

3.  **Configure o `.env`:**
    * Copie o conteúdo de `.env.example` (se existir) para um novo ficheiro chamado `.env`.
    * Preencha **TODAS** as variáveis (especialmente `ALLOWED_DOMAIN`, `SHEET_NAME`, etc.).

### 3. Executando o Sistema

1.  **Primeira Execução (ou após alterar `models.py`):**
    * Se a pasta `instance/` já existir e contiver um ficheiro `sistema_ti.db`, **apague-o** para permitir que o sistema crie a nova estrutura com as chaves estrangeiras e novos módulos.
    * `WARNING:` Isto apagará todos os dados existentes!

2.  **Inicie o servidor Flask:**
    ```bash
    python app.py
    ```
    * Ao iniciar, o `app.py` irá automaticamente criar o ficheiro `instance/sistema_ti.db` e sincronizar os chamados do Google Sheets.

3.  **Acesse o sistema:**
    * Abra o seu navegador e acesse: `http://localhost:5001`
    * Faça login com a sua conta Google do domínio autorizado.
