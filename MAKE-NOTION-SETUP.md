# ⚡ GUIA DE INTEGRAÇÃO — MAKE + NOTION + VILA DAS PEDRAS

## VISÃO GERAL DO FLUXO

```
NOTION (banco de dados)
  ↓
MAKE (automação — roda a cada 1h ou por gatilho)
  ↓
Gera: data/notion-cache.json
  ↓
GitHub Pages serve o arquivo
  ↓
Site lê o JSON e atualiza os cards automaticamente
```

---

## PASSO 1 — CRIAR INTEGRAÇÃO NO NOTION

1. Acesse: https://www.notion.so/my-integrations
2. Clique em **"New integration"**
3. Nome: `Vila das Pedras — Make`
4. Selecione o workspace correto
5. Copie o **Integration Token** (começa com `secret_...`)
6. Nas páginas do Notion que quer conectar:
   - Clique nos **...** da página
   - Clique em **"Add connections"**
   - Selecione `Vila das Pedras — Make`

---

## PASSO 2 — OBTER IDs DOS DATABASES DO NOTION

Para cada database que quer sincronizar:
1. Abra o database no Notion
2. Copie a URL — ex: `https://www.notion.so/SEU-WORKSPACE/DATABASE-ID?v=VIEW-ID`
3. O ID é a string entre a última `/` e o `?`

Databases a configurar:
- 🌐 Pesquisas da Ametista
- 🧠 Hooks - BAÚ DE TEMAS
- 🎥 Reels e Vídeos
- 📸 Referência Visual
- 🎬 Conteúdos em Produção
- 📊 Calendário de Conteúdo / Métricas

---

## PASSO 3 — CRIAR CENÁRIO NO MAKE

### Cenário: "Notion → GitHub Cache"

**Módulos em ordem:**

```
[1] SCHEDULE (Agendamento)
    → Roda a cada 1 hora (ou manualmente)

[2] NOTION — Search Objects
    → Database ID: (seu database de Hooks)
    → Filter: Status = "Para criar" OR "Para gravar"
    → Limit: 20

[3] NOTION — Search Objects (segundo)
    → Database ID: (seu database de Reels)
    → Filter: Status = "Para gravar"
    → Limit: 10

[4] JSON — Create JSON
    → Montar o JSON no formato do notion-cache.json
    → Estrutura:
      {
        "_meta": { "gerado_em": "{{now}}", "fonte": "Notion via Make" },
        "stats": { "reels": {{count(reels)}}, "temas": {{count(hooks)}} },
        "hooks": [ ... array de hooks do Notion ... ],
        "reels": [ ... array de reels do Notion ... ]
      }

[5] GITHUB — Create or Update File
    → Repository: naypedraugc-bot/vila-das-pedras
    → File path: data/notion-cache.json
    → Content: (JSON gerado no passo 4)
    → Commit message: "Auto: atualiza cache Notion"
    → Branch: main
```

---

## PASSO 4 — CONFIGURAR O SITE

No arquivo `index.html`, dentro de `VILA_CONFIG`, preencha:

```javascript
var VILA_CONFIG = {
  notionToken: 'secret_XXXXXXXX...',   // seu token do Notion
  databases: {
    hooks: 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX',
    reels: 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX',
    // ... outros IDs
  },
  makeWebhook: 'https://hook.eu1.make.com/XXXXXXXX',
  cacheFile: 'data/notion-cache.json',
};
```

---

## PASSO 5 — WEBHOOK DE ENTRADA (opcional avançado)

Para disparar automações a partir do site (ex: quando a Guardiã salva algo):

1. No Make, crie um **novo cenário** com módulo inicial: `Webhooks — Custom Webhook`
2. Copie a URL gerada
3. Cole em `VILA_CONFIG.makeWebhook`
4. No Make, conecte o webhook a ações no Notion (criar página, atualizar status, etc.)

**Eventos que o site pode disparar:**
```javascript
// Exemplos de uso no console do site:
vilaWebhook('nova_referencia', { titulo: 'Meu Reel', formato: 'Reel' });
vilaWebhook('conteudo_aprovado', { id: 'xxx', status: 'Aprovado' });
vilaWebhook('ametista_pesquisou', { tema: 'IA marketing', insights: '...' });
```

---

## ESTRUTURA DE ARQUIVOS NO REPOSITÓRIO

```
vila-das-pedras/
├── index.html                    ← site principal
├── data/
│   └── notion-cache.json         ← gerado pelo Make automaticamente
└── assets/
    ├── banner-vila-das-pedras.png
    ├── conselho.png
    ├── PROJETOS.png
    ├── ametista.png
    ├── sodalita.png
    ├── aguamarinha.png
    ├── quartzorosa.png
    ├── citrino.png
    ├── cornalina.png
    └── onix.png
```

---

## FLUXO FUTURO — PIPELINE MULTIAGENTE

```
Referência salva (manual ou Save to Notion)
  ↓
Make detecta nova entrada no Notion
  ↓
Make chama Claude API (Ametista) com o conteúdo
  ↓
Ametista gera relatório estratégico
  ↓
Make salva relatório em "Pesquisas da Ametista" no Notion
  ↓
Make atualiza data/notion-cache.json no GitHub
  ↓
Site exibe automaticamente na aba Baú de Temas / Reels
  ↓
Sodalita recebe contexto atualizado na próxima sessão
```

---

## CHECKLIST DE ATIVAÇÃO

- [ ] Token do Notion criado e copiado
- [ ] Integração adicionada aos databases no Notion
- [ ] IDs dos databases copiados
- [ ] Cenário Make criado e testado
- [ ] Conexão GitHub no Make configurada
- [ ] `VILA_CONFIG` no index.html preenchido
- [ ] Primeiro `data/notion-cache.json` gerado pelo Make
- [ ] Site testado em produção
- [ ] Webhook de entrada configurado (opcional)

---

## SUPORTE

Dúvidas sobre o Make: https://make.com/en/help
Dúvidas sobre a API do Notion: https://developers.notion.com
Repositório da Vila: https://github.com/naypedraugc-bot/vila-das-pedras
