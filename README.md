# Dashboard Produção Cirúrgica
## Resumo do Projeto para Memória de Contexto

---

## O QUE FOI CONSTRUÍDO
Dashboard interativo completo de produção cirúrgica doe um hospital em São Paulo, desenvolvido como arquivo HTML standalone (~1MB), sem frameworks, sem servidor, sem Power BI.

**Arquivo entregue:** `dashboard_producao_cirurgica_HRR.html`

---

## FONTE DE DADOS
- **Arquivo:** `Produção_Cirurgica_HRR.xlsx`
- **Aba principal:** `Registro cirúrgico` — colunas A até AS, linhas 1 a 2666
- **Total de registros:** 2.665 cirurgias
- **Período:** Janeiro 2025 a Abril 2026
- **Observação importante:** há inconsistências nos campos `MÊS` e `ANO` para registros do dia 1 de cada mês (digitados com o mês anterior). Todos os agrupamentos temporais usam o campo `DATA_STR` (derivado de `DATA`) como fonte de verdade, ignorando `MÊS`/`ANO`.

---

## IDENTIDADE VISUAL
- **Empresa:** EXIMIO — ecossistema de gestão hospitalar
- **Cores:** Amarelo `#FDD306` (accent principal), Cinza `#6f7378`, Cinza escuro `#231f20` (background)
- **Fonte:** Roboto Condensed (títulos/labels) + Roboto Mono (valores/código)
- **Estilo:** dark theme, bordas angulares, sem linhas de grade

---

## FILTROS GLOBAIS (barra superior)
Todos os filtros são encadeados e afetam todos os gráficos e KPIs simultaneamente.

| Filtro | Tipo | Campo |
|--------|------|-------|
| Ano | Multi-select checkbox dropdown | `ANO` (derivado de DATA_STR) |
| Mês | Multi-select checkbox dropdown | `MÊS` (derivado de DATA_STR) |
| Data | Date range picker (2 calendários) | `DATA_STR` |
| Sala | Select simples | `SALA` |
| Especialidade | Select simples | `CARDIACA` |
| Prioridade | Select simples | `PRIORIDADE` |
| 1ª Cirurgia | Select simples | `1a cirurgia` |
| Final de Semana | Select simples | `Final de semana?` |

**Filtros padrão ao abrir:** 1ª Cirurgia = Somente 1ª / Final de Semana = Não / Data = 01/08/2025–31/03/2026

**Botão "↺ Limpar filtros"** reseta tudo para valores em branco.

---

## CROSS-FILTERING INTERATIVO
- Clicar em **qualquer elemento** de qualquer gráfico filtra todos os outros
- Suporta **múltipla seleção** (ex: dois cirurgiões ao mesmo tempo) — usa `Set` por dimensão
- Chips visuais aparecem abaixo dos filtros mostrando filtros ativos com `×` individual
- Cards com filtro ativo ganham borda amarela brilhante
- Clicar no mesmo elemento novamente **remove** o filtro (toggle)

---

## GRÁFICOS

### KPI Cards (5 cards)
Total Cirurgias / Eletivas / Urgência+Emergência / Cirurgias Grandes / Final de Semana

### Produção Cirúrgica Mensal
- Barras agrupadas por ano, mês no eixo X
- Opção: valor absoluto OU empilhado por prioridade
- Labels derivados de `DATA_STR` (imune às inconsistências de `MÊS`/`ANO`)
- Mobile: scrollável horizontalmente (min 58px/coluna)

### Porte da Cirurgia
- Donut chart — PEQUENA / MEDIA / GRANDE

### Prioridade
- Donut chart — ELETIVA / URGÊNCIA / EMERGÊNCIA

### Nível ASA (Treemap squarified)
- Algoritmo squarified implementado do zero
- Dropdown: **Consolidado** (ASA I, II, III, IV, V) ou **Detalhado** (inclui variantes "E" de emergência como ASA I E, ASA II E...)
- Normaliza automaticamente variantes de digitação (i, ii, iii, IE, IIE...)
- Cores: amarelo→cinza→âmbar→vermelho→roxo

### Distribuição de Horário de Início por Período
- Barras 100% empilhadas, percentual por faixa
- Faixas: Até as 08h / Entre 08h e 10h / Entre 10h e 12h / Entre 12h e 14h / Depois das 14h
- **Nota:** os labels "Até as 10h", "Até as 12h", "Até as 14h" no Excel foram renomeados para "Entre Xh e Yh" apenas na exibição — o dado bruto continua com os nomes originais
- Dropdown: agrupamento por Ano/Mês, Especialidade ou Porte
- Mobile: scrollável + labels abreviados (<08h, 08-10h...)

### Produção por Especialidade
- Barras horizontais, ordenadas por volume
- Cross-filter por especialidade

### Top Cirurgiões
- Barras horizontais
- Dropdown: Top 5 / Top 10 / Top 15 / Top 20 / Todos

### Top Anestesistas
- Barras horizontais (idêntico ao de cirurgiões)
- Dropdown: Top 5 / Top 10 / Top 15 / Top 20 / Todos
- Campo: `ANESTESISTA` (coluna L do Excel)

### Distribuição por Sexo
- Donut chart

### Faixa Etária
- Barras verticais com faixas customizadas derivadas da coluna `IDADE`:
  - 0 a 1 ano / 2 a 4 anos / 5 a 8 anos / 9 a 12 anos / 13 a 20 anos / 21 a 30 anos / 31 a 40 anos / 41 a 60 anos / 61 a 80 anos / 81+ anos

### Tipo de Anestesia (Treemap squarified)
- Dropdown: **Detalhado** (normaliza ~60 variantes de digitação em ~10 labels limpos) ou **Por Técnica** (6 grupos clínicos)
- Grupos clínicos: Geral / Raqui + Sedação / Raqui / Sedação Combinada / Sedação / Local-Bloqueio
- Normaliza automaticamente typos: RAQUI+SEDAÇAÕ, GERALL, RQUI, GER, etc.
- Campo: `ANESTESIA`

---

## ARQUITETURA TÉCNICA

### Stack
- **Chart.js 4.4.1** (via cdnjs) — todos os gráficos canvas
- **Treemaps** — algoritmo squarified implementado do zero em JS vanilla
- **Date range picker** — implementado do zero (sem biblioteca)
- **Multi-select dropdowns** — implementados do zero
- Sem React, sem D3, sem jQuery, sem backend

### Dados
- Embutidos inline no HTML como `const RAW_DATA = [...]`
- Gerados via script Python com `pandas` a partir do Excel
- Campo `DATA_STR` no formato `YYYY-MM-DD` é a fonte de verdade temporal

### Funções JS principais
- `applyGlobalFilters()` — aplica todos os filtros globais, reseta cross-filters
- `getActiveData()` — aplica cross-filters sobre filteredData
- `setCrossFilter(dim, val)` — toggle de valor em Set por dimensão
- `getDP(r)` — extrai ano/mês/abbr/key de DATA_STR (ignora MÊS/ANO do Excel)
- `normASA(v, mode)` — normaliza valor ASA para Consolidado ou Detalhado
- `normAnest(v, mode)` — normaliza tipo de anestesia para Detalhado ou Por Técnica
- `getFaixa(age)` — classifica IDADE numérica nas faixas etárias customizadas
- `renderASA()` / `renderAnestesia()` — treemaps com squarify próprio
- `squarify(items, row, x, y, w, h)` — algoritmo de treemap
- Plugin `hrr_datalabels` registrado no Chart.js — labels dentro das barras (mobile-friendly)

### Mobile (responsivo)
- Breakpoints: 900px (filtros colapsáveis), 768px (layout coluna única), 480px
- Filtros em painel colapsável com badge de contagem de filtros ativos
- Gráficos de barras horizontais: `afterFit` fixa 80px para labels, trunca nomes >10 chars
- Produção Mensal e Hora de Início: scroll horizontal com min-width dinâmico
- Data labels: 8px no mobile, desenhados dentro das barras (nunca transbordam)
- Zero uso de `?.` (optional chaining) ou `{...spread}` — compatível com Android WebView antigo

---

## PRÓXIMOS PASSOS PLANEJADOS
1. **Conectar à planilha online** — aguardando informação se será Google Sheets ou Excel Online/SharePoint
   - Google Sheets pública: só o link basta
   - Google Sheets privada: precisa de API key do Google Cloud
   - SharePoint/OneDrive: precisa de token Microsoft Graph
2. **Publicar na web** — opções: Netlify Drop (imediato) ou GitHub Pages (permanente + portfólio)
3. **README para GitHub** — a preparar quando o projeto for publicado

---

## NOTAS DE MANUTENÇÃO
- Para atualizar os dados: rodar o script Python com o novo Excel e substituir o bloco `const RAW_DATA = [...]` no HTML
- O campo `SALA` contém valores: 1, 2, 3, 4, 5, HEMO (sala de hemodinâmica)
- O campo `ASA` tem muitas variantes de digitação — a função `normASA()` trata todas
- O campo `ANESTESIA` tem ~60 variantes — a função `normAnest()` trata todas
- Registros com DATA_STR fora do range de filtro de data são corretamente excluídos mesmo que MÊS/ANO indiquem outro período
