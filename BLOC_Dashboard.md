# BLOC — Dashboard Estratégica
## Documento de Projeto Completo

---

## 1. Contexto e Problema

O BLOC é um app de treino single-file HTML (~5.000 linhas). O objetivo da dashboard não é ser uma tela genérica cheia de gráfico aleatório — é **fechar o gap entre o que o usuário acha que está fazendo e o que realmente está fazendo**.

**Critério de corte para qualquer métrica:** ela muda o comportamento do usuário? Se não muda, corta.

O app vivia com:
- Histórico sem timestamp de hora (só data)
- Sem duração de bloco
- Sem mapeamento de grupo muscular por tipo de bloco
- Aba "metas" existente mas desconectada do fluxo principal

---

## 2. Arquitetura Decidida

### Arquivo único
O projeto permanece como single-file HTML. Separar em múltiplos `.js` agora quebraria o fluxo sem ganho real — a modularidade é mental, não física.

### Camadas conceituais da dashboard (não telas separadas)
Tudo em scroll único dentro da aba **Stats**, sem sub-navegação:

| Camada | Pergunta que responde |
|---|---|
| **Pulso** | Como estou indo agora? |
| **Insights automáticos** | O que meu comportamento real está dizendo? |
| **Padrões** | O que se repete ao longo do tempo? |
| **Evolução** | Estou ficando melhor? |
| **Metas** | O que estou perseguindo? |

### Navegação
A aba **"metas"** foi substituída por **"stats"** no nav principal. As metas foram absorvidas como seção dentro de stats — menos abas, mais coesão.

---

## 3. Roadmap — 4 Fases + Extras

### Fase 0 — Fundação de dados ✓
*Fazer antes de qualquer UI. Dados históricos são imutáveis — cada campo ausente hoje é uma análise impossível para sempre.*

| Item | Implementado |
|---|---|
| `timestamp` ISO 8601 em `saveLog()` | ✓ |
| `duration` em segundos por bloco | ✓ |
| `sessionStart` gravado no `sessionStorage` ao abrir bloco | ✓ |
| Dicionário `MUSCLE_GROUPS` por tipo de bloco | ✓ |
| `DATA_VERSION` 1 → 2 com migração retroativa | ✓ |

**Migração:** registros antigos recebem `timestamp: null` e `duration: null`. Sem dado = sem análise, nunca erro.

### Fase 1 — Analytics core ✓
*Funções puras, sem tocar no DOM. Testáveis no console a qualquer momento.*

| Função | O que faz |
|---|---|
| `calcStreak()` | Dias consecutivos com ao menos 1 registro. Retorna `{current, best}` |
| `calcWeekRate()` | Taxa de conclusão da semana atual (%) |
| `calcWeeklyVolume(offset)` | Volume total em kg de qualquer semana passada |
| `getSkippedExercises()` | Agrega `status:'skipped'` por exercício, retorna ranqueado |
| `renderHeatmap()` | SVG puro, grade Seg→Dom, 12 semanas |

### Fase 2 — Dashboard mínima ✓
*Aba stats funcional com dados reais.*

| Seção | Descrição |
|---|---|
| Cards de pulso | Streak + taxa da semana + volume (grid 3 colunas) |
| Heatmap de frequência | 12 semanas, labels de meses, intensidade por volume semanal |
| Consistência por dia | Donut SVG + barras horizontais por dia da semana |
| Evolução de carga | Gráfico de linha SVG com área preenchida por exercício |
| Volume semanal | Barras SVG das últimas 8 semanas |
| PRs | Grid de cards 2 colunas com maior peso por exercício |

Funções adicionadas:
- `getTopExercisesEvolution(limit)` — exercícios com mais histórico de carga
- `calcDayOfWeekStats()` — taxa de conclusão por dia da semana
- `getAllPRs()` — maior peso já registrado por exercício
- `renderSparkline(vals, useWeight)` — sparkline SVG 80×28px

### Fase 3 — Comportamento e padrões ✓

| Seção | Descrição |
|---|---|
| Frequência muscular | Barras horizontais por grupo, normalizado pelo mais frequente |
| Aderência ao plano | 5 piores blocos: planejado vs real, barra colorida semântica |
| Resumo semana passada | Dias + blocos + volume + PRs + mais pulados |

Funções adicionadas:
- `calcMuscleFrequency()` — usa `MUSCLE_GROUPS`, retorna por sessões decrescente
- `calcBlockAdherence()` — calcula semanas do período histórico e compara com registros reais
- `calcWeeklySummary(weekOffset)` — dados completos de qualquer semana passada
- `isPR_before(exName, weight, beforeDate)` — helper para PRs dentro de período

### Fase 4 — Insights automáticos ✓
*Cards gerados automaticamente a partir do comportamento real. Aparecem no topo de stats, logo abaixo do pulso.*

| Insight | Condição de disparo |
|---|---|
| 🔥 Streak em alta | 7+ dias consecutivos |
| ⚠️ Sequência zerada | Streak atual = 0 após recorde ≥ 5 dias |
| 📉 Volume caindo | Volume caindo 2 semanas seguidas |
| 📈 Semana mais intensa | Volume +20% vs semana anterior |
| 👻 Blocos nunca registrados | Blocos no plano que nunca tiveram um registro |
| 🚫 Exercício mais evitado | Taxa de skip ≥ 50% |
| 💤 Grupo muscular parado | Grupo muscular sem registro há 14+ dias |
| 🏆 PR recente | Personal record nos últimos 7 dias |
| 📅 Dia fraco | Dia da semana com consistência < 40% e mínimo 2 planejados |
| ✅ Semana perfeita | Taxa de conclusão da semana = 100% |

Design dos cards: borda esquerda colorida semântica (verde = positivo, amarelo = atenção, cinza = neutro).

Funções adicionadas:
- `generateInsights()` — analisa histórico e retorna array `{type, icon, text, sub}`
- `doneSet_insight(b, dk, doneBlocks)` — helper interno

### Extras da sessão ✓

| Item | Descrição |
|---|---|
| Metas restauradas | Seção § 11 dentro de stats com botão "+ nova", barra de progresso, controles ±1% ±5% |
| Heatmap corrigido | Bug de labels à direita → movido para esquerda; grid começava no domingo → corrigido para segunda-feira |
| Todos os 7 dias no heatmap | Antes mostrava só seg/qua/sex; agora S T Q Q S S D todos visíveis |
| Dashboard gráfica | Substituiu listas planas por gráficos SVG reais (linha, barras, donut) |

---

## 4. Estrutura de Dados

### Snapshot gravado por `saveLog()`
```js
{
  date: "10/05/2026",           // data pt-BR (existia)
  timestamp: "2026-05-10T14:32:00.000Z", // ISO 8601 — NOVO (Fase 0)
  duration: 2340,               // segundos — NOVO (Fase 0)
  dayKey: "SEG",                // dk do bloco (existia)
  blockLabel: "Academia",       // label do bloco (existia)
  blockType: "academia",        // tipo do bloco (existia)
  exercises: [                  // array de exercícios (existia)
    {
      name: "Supino",
      sets: 4,
      reps: 10,
      weight: 80,
      status: "done" | "skipped"  // por exercício (existia)
    }
  ]
}
```

### Dicionário MUSCLE_GROUPS
```js
const MUSCLE_GROUPS = {
  academia:    ['peito','costas','ombro','bíceps','tríceps','perna'],
  mobilidade:  ['full_body'],
  cardio:      ['cardio'],
  // ... por tipo de bloco
}
```

---

## 5. Decisões de UX

**Scroll único vs tabs internas:** escolhido scroll único. O histórico já funciona assim, usuário está acostumado. Tabs entram quando tiver conteúdo demais para uma tela — ainda não chegou lá.

**Métricas com impacto psicológico real:**
- Streak funciona — mas só se o usuário achar que pode quebrar
- Progresso relativo bate progresso absoluto ("40% mais que semana passada" > "6 treinos")
- Nomeação do padrão negativo tem efeito forte — "você pulou posterior de coxa 4 semanas seguidas" é específico e pessoal
- Menos é mais — máximo 3-4 números em destaque, o resto abaixo da dobra

**O que foi descartado ou adiado:**
| Item | Decisão | Motivo |
|---|---|---|
| Análise de horários de treino | ❌ Bloqueado | Depende de timestamp — fundação foi criada na Fase 0, análise possível no futuro |
| Timeline de evolução visual | ⏳ Adiado | Cognitivamente pesado, usuário raramente lê timelines longas |
| Sistema de conquistas | ⏳ Adiado | Gamificação vazia sem comportamento real para reconhecer |
| Refatorar em múltiplos arquivos | ❌ Não agora | Quebra o fluxo sem ganho real |
| sessionRPE | ⏳ Futuro | Percepção subjetiva de esforço — custo baixo, valor alto, mas não urgente |

---

## 6. Análises Comportamentais Implementadas

O diferencial real do BLOC frente a outros apps: falar sobre o comportamento do usuário de forma **específica e pessoal**.

Não "você treinou 5 vezes essa semana".  
Mas "você nunca completa blocos de Recovery — estão no plano há 8 semanas e foram registrados 0 vezes".

Análises ativas:
- Exercícios frequentemente ignorados (taxa de skip por exercício)
- Dias da semana com maior taxa de abandono
- Grupos musculares negligenciados (14+ dias sem registro)
- Comparação plano vs realidade por bloco
- Consistência por período (streak atual vs recorde)
- Aderência por tipo de treino
- Volume total por semana com comparação percentual
- Taxa de conclusão por bloco (planejado vs registrado)
- PRs automáticos por exercício
- Evolução de carga temporal com gráfico de linha

---

## 7. O Que Falta

### Pendente do roadmap original
- **Insights automáticos — expansão:** dezenas de cards novos além dos 10 existentes (comportamento mais granular, conquistas específicas e contextuais)
- **Sistema de conquistas:** só vale implementar quando houver dados suficientes para que as conquistas sejam inesperadas e significativas

### Limpeza técnica
- Nenhuma pendência crítica. As funções órfãs de metas (`updGoal`, `delGoal`, `openModal({t:'addGoal'})`) foram reativadas quando metas voltou para dentro de stats.

### Possíveis próximos passos
- Análise de horários de treino (fundação já existe com timestamp da Fase 0)
- Duração média por bloco (fundação existe com duration da Fase 0)
- Resumos semanais automáticos push (requer PWA/Service Worker)
- Comparação entre períodos (este mês vs mês anterior)
- Insights mais granulares: combinações de padrões (ex: "você treina academia mas pula mobilidade no mesmo dia consistentemente")

---

## 8. Estado do Código

| Métrica | Valor |
|---|---|
| Linhas totais | ~5.871 |
| Arquitetura | Single-file HTML |
| Funções analytics | 15 funções puras |
| Seções na aba stats | 11 seções |
| Insights automáticos | 10 tipos |
| DATA_VERSION | 2 |
| Migrações | 2 (v0→v1, v1→v2) |
