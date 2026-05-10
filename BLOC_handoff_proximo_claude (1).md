# BLOC — Documento de handoff para o próximo Claude

Gerado em 10/05/2026. Passar este documento ao próximo Claude antes de qualquer implementação.

---

## Contexto do projeto

BLOC é um monólito vanilla JS (~8.200 linhas, um único `index.html`) com arquitetura em camadas rígidas:

```
TEMPLATE (blocks) → SESSÃO DO DIA (todaySession) → HISTÓRICO (history)
```

**Regras críticas que nunca podem ser violadas:**
- `renderHoje()` lê apenas `todaySession`, nunca `blocks[todayKey]`
- `saveLog()` lê apenas `todaySession`, nunca `blocks`
- Mutações em `blocks` nunca vêm de `todaySession` ou `history`
- Toda mutação em `todaySession` → `saveTodayCache()`
- Toda mutação em `blocks` → `saveStore()` + `resetTodaySession()`
- Tudo feito na guia Hoje fica apenas na guia Hoje, a menos que o usuário clique explicitamente em "Adicionar à Semana"

---

## ✅ Implementado nesta sessão

| Item | Descrição |
|------|-----------|
| Bug 🔴 | `setsRemaining` não era preservado ao reabrir o app — corrigido via `wmResume()` e proteção do botão "Iniciar" |
| Segunda tela | `wm-root` fixo no DOM (`position:fixed` no `body`), overlay minimiza sem encerrar, pip flutuante, boot restaura overlay automaticamente |
| Timer | Botão "Confirmar série" ficava travado após pular descanso — corrigido (faltava `_wmNeedsRender=true` em `_timerCancel`) |
| Descanso | Limite estendido para 5 minutos — `REST_STEPS = [15,30,45,60,90,120,180,240,300]` |
| Encerrar | `confirm()` bloqueado silenciosamente em PWA → substituído por confirmação inline de dois cliques com timeout de 4s |
| Layout | `wm-root` com `inset:0` conflitante com `left:50%` corrigido — overlay volta a cobrir a tela corretamente |

---

## Estado atual do sistema

### Flags de UI da segunda tela (§3 STATE)
```javascript
let _wmOverlayVisible = false; // true = overlay visível; false = em background
let _wmNeedsRender    = false; // true = forçar re-render da overlay no próximo render()
let _wmAbortPending   = false; // true = aguardando 2º clique para confirmar encerramento
```
**Importante:** esses três flags são de UI pura — **não entram no `workoutMode` salvo em cache**.

### Fluxo da segunda tela
- `wmStart()` → seta `_wmOverlayVisible=true`, `_wmNeedsRender=true`, chama `render()`
- `wmHideOverlay()` → seta `_wmOverlayVisible=false`, mostra pip flutuante (`#wm-pip`)
- `wmShowOverlay()` / `wmResume()` → seta `_wmOverlayVisible=true`, remove pip
- `wmAbortConfirm()` → dois cliques: 1º seta `_wmAbortPending=true`; 2º chama `wmAbort()`
- `wmAbort()` → encerra tudo, limpa `wm-root`, chama `render()`
- `wmFinish()` → registra bloco, limpa `wm-root`, chama `render()` + `saveLog()`
- `wmClose()` → mantido por compatibilidade; chama `wmHideOverlay()` se treino ativo

### Botão "Iniciar / Retomar" na guia Hoje
Três estados mutuamente exclusivos:
- `!workoutMode.active` → "Iniciar treino guiado" → chama `wmStart(dk, bi)`
- `workoutMode.active && isWmActive` → "Retomar treino em andamento" → chama `wmResume()`
- `workoutMode.active && !isWmActive` → nenhum botão (outro bloco está em treino)

### _wmNeedsRender — quando deve ser setado true
Toda função que muda estado visível da overlay deve setar `_wmNeedsRender=true` antes de `render()`:
- `wmNext()`, `wmSkip()`, `wmSetRestTime()`, `wmReorderEx()`
- `_timerCancel()` (re-habilita o botão Confirmar)
- Timer ao expirar (linha do `if(workoutMode.active)`)
- `wmAbortConfirm()` (mostra estado "Confirmar?")
- `wmStart()` (treino novo)

### wm-root no DOM
- Criado UMA vez em `render()`, appendado ao `document.body` (não ao `#app`)
- `position:fixed; top:0; bottom:0; left:50%; transform:translateX(-50%); width:100%; max-width:430px`
- Visibilidade via `.wm-hidden` (`visibility:hidden`) — nunca destruído enquanto treino ativo
- Só tem `innerHTML` reescrito quando `_wmNeedsRender===true` ou `wmRoot.dataset.wmActive!=='true'`

---

## 🟡 Pendentes — definir com o usuário antes de implementar

### 1. Sistema de dados do histórico (dois tipos)
O usuário mencionou querer "dados certinhos" e "dados incertos" no histórico.

**Perguntar antes de implementar:**
> "Se você planejou 3×10 a 60kg mas durante o treino fez 3×12 a 65kg, o que quer ver no histórico — o planejado (3×10/60kg), o executado (3×12/65kg), os dois separados, ou uma nota de variação?"

Impacta diretamente: schema de `history[]`, `saveLog()`, `calcWeeklyVolume`, `calcMonthlyVolume`.

### 2. Revisão dos dois timers
O app tem dois timers distintos:
1. `timerState` + setInterval — timer de descanso dentro do workout mode
2. `#rest-timer-banner` / `_timerRenderOutside()` — timer avulso na guia Hoje fora do workout mode

**Perguntar antes de implementar:**
> "O timer avulso da guia Hoje (fora do treino guiado) — quer manter como está, remover, ou transformar em notificação também?"

### 3. Re-render a cada clique *(anotado, NÃO implementar sem alinhamento explícito)*
O usuário quer **discutir** a estratégia antes de qualquer mudança. Atualmente todo clique (abrir bloco, abrir exercício, trocar aba) chama `render()` completo que reconstrói o `#content` inteiro.

Opções a discutir:
- Toggle de classe CSS direto no DOM para `toggleEx` / `toggleBlk` (requer adicionar `id` nos elementos)
- Manter como está (funciona, só tem flash visual)
- Outra abordagem que o usuário prefira

---

## Sistemas que NÃO devem ser tocados

- `calcWeeklyVolume()`, `calcMonthlyVolume()`, `calcStreak()`
- `_buildHistoryIndex()` — `completedSets` não entra no índice de PR
- `_statsCache`
- `blocks` — nunca tocado pelas features do treino guiado
- `sessionExStatus` — mantido como está
- Drag engine (`_drag`, `_exDrag`)

---

## Referência de localização no arquivo (aproximada — usar grep -n para confirmar)

- §1 CONSTANTS: ~linha 3000
- §2 MIGRATIONS/STORE: ~linha 3200
- §3 STATE: ~linha 3500 (`workoutMode`, `timerState`, `_wmOverlayVisible`, `_wmNeedsRender`, `_wmAbortPending`)
- §4 SESSION: ~linha 3600 (`todaySession`, `saveTodayCache`, `loadTodayCache`)
- §5 ANALYTICS: ~linha 3700
- §8 CRUD/ACTIONS: ~linha 5100 (`saveLog`, `promoteBlockToWeek`)
- §9 RENDER ENGINE: ~linha 5200 (`renderHoje`, `renderBlockCard`, `renderExRow`)
- WORKOUT MODE: ~linha 6630 (`wmStart`, `wmNext`, `wmSkip`, `wmShowOverlay`, `wmHideOverlay`, `wmAbort`, `wmFinish`, `wmAbortConfirm`, `wmResume`, `renderWorkoutMode`)
- TIMER: ~linha 6460 (`_timerStart`, `_timerStop`, `_timerCancel`, `_timerRenderWm`, `_timerRenderOutside`)
- SW / Notificações: início do arquivo, ~linha 17
- Boot: final do arquivo, ~linha 8000

---

## Instrução para o próximo Claude

1. **Pedir o `index.html` atual** antes de qualquer implementação — as linhas acima são aproximadas
2. **Usar `grep -n`** para localizar funções exatas antes de editar
3. **Para cada item 🟡**, perguntar ao usuário antes de codar
4. **Para o item de re-render**, não tocar até o usuário confirmar a abordagem desejada
5. `wmClose()` existe só por compatibilidade — novos `onclick` devem usar `wmHideOverlay()` ou `wmAbortConfirm()`
