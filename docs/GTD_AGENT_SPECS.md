# Documento de Especificação: Sistema GTD/PARA via NanoClaw

## 1. Visão Geral da Arquitetura
O sistema será implementado como uma extensão/customização sobre a base de código do **NanoClaw** (`qwibitai/nanoclaw`). O agente utilizará a interface de mensageria nativa do NanoClaw (ex: Telegram) para receber os inputs do usuário, processará a intenção usando as capacidades nativas do Claude Code e fará o roteamento local de arquivos (Obsidian) e chamadas de API externas (Todoist).

**Objetivo:** Transformar a instância do NanoClaw em um "Chefe de Gabinete" focado no método GTD e framework de arquivos PARA.

---

## 2. Requisitos de Infraestrutura e Isolamento (Crucial)
Devido ao isolamento de sistema de arquivos (sandbox) do NanoClaw:
1. **Obsidian (Direct File System):** O diretório raiz do Vault do Obsidian do usuário deve ser mapeado como um volume explicitly montado no contêiner do NanoClaw (ex: `/host_mnt/obsidian_vault`). O agente deve ler e escrever arquivos `.md` diretamente usando a biblioteca `fs` do Node/TypeScript, dispensando o uso de plugins de API no Obsidian.
2. **Variáveis de Ambiente (`.env`):** O agente deve garantir que o `.env` contenha o `TODOIST_API_TOKEN` para as chamadas REST externas.
3. **Mensageria:** O agente deve utilizar a skill de Telegram nativa do NanoClaw (ou configurar a integração equivalente) para o I/O com o usuário.

---

## 3. Especificação do Obsidian (Cérebro Digital Local)

### 3.1. Árvore de Diretórios (Framework PARA)
O agente deve garantir que a seguinte estrutura exista no Vault montado e rotear os arquivos estritamente para essas pastas:

```text
📦 Vault
 ┣ 📂 00_Inbox       # Purgatório: itens ambíguos ou falhas de roteamento
 ┣ 📂 01_Projetos    # Material de suporte para metas com prazo (ativas)
 ┣ 📂 02_Areas       # Responsabilidades contínuas (ex: Saúde, Finanças)
 ┣ 📂 03_Recursos    # Interesses, Read-it-later, Zettelkasten, Links
 ┣ 📂 04_Arquivo     # Inativos/concluídos (Ação de manutenção)
 ┗ 📂 05_Diario      # Logs, reuniões, dumps de áudio processados com data
```

### 3.2. Estrutura de Metadados (YAML Frontmatter)
Toda nota criada pelo NanoClaw DEVE conter o bloco de propriedades YAML abaixo, formatado para facilitar a recuperação de contexto via RAG posteriormente:

```yaml
---
id: "UUID-ou-Timestamp"
tipo: "MEETING_LOG | LINK | INSIGHT | PROJECT_SUPPORT"
data_criacao: "YYYY-MM-DDTHH:mm:ss"
projeto: "[[Nome do Projeto]]" # Se houver. Usar wikilink padrão do Obsidian
tags: ["tag1", "tag2"]
pessoas: ["[[Nome da Pessoa]]"] # Opcional
status: "processado" # ou "revisar" (se for para a 00_Inbox)
---
# Título da Nota
[Conteúdo sintetizado pelo Claude]
```

---

## 4. Especificação do Todoist (Motor de Ação)
O script de handler de mensagens do NanoClaw deve fazer chamadas REST para a API do Todoist v2.
* **Projetos:** As strings de projeto identificadas pelo LLM devem dar *match* (ou serem criadas) com os IDs do Todoist.
* **Labels Mapeadas:** * `@aguardando` (Ações delegadas a terceiros)
    * `@baixa_energia` / `@alta_energia` (Inferido pelo LLM na captura)
* **Regra de Ouro:** Nenhuma nota de referência deve ir para o Todoist. Apenas **Next Actions** (verbos acionáveis).

---

## 5. Lógica de Roteamento (Modificação do Message Handler)

O agente deve interceptar as mensagens recebidas via NanoClaw e instruir o LLM interno a gerar um output estruturado (JSON) antes de executar as ações. O *System Prompt* do handler deve conter a seguinte lógica de árvore de decisão:

> **Instruções de Roteamento (Internal Prompt):**
> Você é o NanoClaw, atuando como roteador GTD. Analise a mensagem do usuário e extraia (1) Tarefas acionáveis e (2) Notas de referência.
> 
> **Decisão de Diretórios para Notas (Obsidian):**
> - Se não tiver contexto claro -> `folder: 00_Inbox`
> - Se for log/reunião/data específica -> `folder: 05_Diario`
> - Se for apoio a um projeto ativo -> `folder: 01_Projetos`
> - Se for manutenção de rotina (saúde, finanças) -> `folder: 02_Areas`
> - Se for consumo, links, artigos, ideias genéricas -> `folder: 03_Recursos`

O handler do NanoClaw deve então fazer o *parse* dessa decisão, disparar os POSTs via `fetch` para a API do Todoist, e usar `fs.writeFileSync` para criar os `.md` no volume do Obsidian.

---

## 6. Funcionalidades (Skills) a serem Implementadas no NanoClaw

Como o NanoClaw suporta customizações modulares, o agente deve implementar as seguintes capacidades:

### Skill 1: `process_capture` (O Funil Base)
* Modificar o loop principal de resposta do NanoClaw para escutar mensagens. Quando uma mensagem chega, aplicar a "Lógica de Roteamento" (Sec. 5).
* Retornar uma resposta silenciosa ou curta ao usuário via Telegram: *"✅ Salvo: 1 tarefa no Todoist, 1 link no Obsidian."*

### Skill 2: `context_briefing` (Recuperação RAG)
* Criar um comando natural. Se o usuário mandar *"Me atualize sobre [Projeto X]"*:
* O NanoClaw deve ler recursivamente as pastas do Obsidian via `fs`, encontrar menções ao Projeto X através do YAML (ou grep de texto), fazer requisição ao Todoist pelas tarefas abertas desse projeto, sintetizar um briefing e responder no Telegram.

### Skill 3: `weekly_review` (Cron Job Nativo)
* Usar o sistema de *scheduled jobs* do NanoClaw para rodar toda sexta-feira às 16h.
* **Ação do Job:** O script lê o volume do Obsidian (`00_Inbox`), consulta as tarefas vencidas da API do Todoist e envia uma mensagem proativa para o Telegram do usuário: *"Hora da Revisão. Você tem X itens na Inbox e Y tarefas atrasadas. O que faremos com a Tarefa Z?"* conduzindo o fluxo de limpeza em formato de chat.
