# PROMPT BASE — DESENVOLVIMENTO DO BLOC

Você está trabalhando no código do BLOC.

Antes de qualquer alteração:
leia a arquitetura atual, respeite as seções existentes e entenda o fluxo de estado do sistema.

O BLOC NÃO é um arquivo aleatório.
Ele é um monólito estruturado em camadas.

Seu trabalho NÃO é “só fazer funcionar”.
Seu trabalho é:

* preservar coerência arquitetural;
* reduzir acoplamento;
* evitar regressões;
* manter previsibilidade;
* manter escalabilidade futura;
* manter o código legível;
* respeitar as fronteiras entre sistemas.

---

# CONTEXTO DO PROJETO

O BLOC passou por uma grande reorganização arquitetural (Fase 5).

O sistema foi dividido em seções estruturadas e documentadas.

A arquitetura principal validada é:

TEMPLATE → SESSÃO → HISTÓRICO

Fluxo:

* `blocks` = template persistente da semana;
* `todaySession` = snapshot mutável do treino atual;
* `history` = registros imutáveis já concluídos.

ESSA SEPARAÇÃO É CRÍTICA.

Nunca misture responsabilidades entre essas camadas.

---

# REGRA MAIS IMPORTANTE

NÃO implemente features “em um único ponto”.

Antes de alterar qualquer funcionalidade:
mapeie TODAS as camadas impactadas.

Quase toda feature no BLOC atravessa:

* estado;
* render;
* persistência;
* cache;
* índices;
* stats;
* histórico;
* UI;
* invalidação;
* drag;
* sessão;
* storage.

Se modificar apenas uma função:
você provavelmente criou inconsistência invisível.

---

# COMO VOCÊ DEVE TRABALHAR

Sempre siga este fluxo:

## 1. MAPEAR

Antes de escrever código:
liste:

* quais sistemas serão afetados;
* quais estados dependem disso;
* quais renders usam isso;
* quais caches precisam invalidar;
* quais side-effects existem;
* quais funções assumem o comportamento antigo.

---

## 2. IDENTIFICAR FRONTEIRAS

Pergunte:

* isso pertence ao TEMPLATE?
* pertence à SESSÃO?
* pertence ao HISTÓRICO?
* pertence ao RENDER?
* pertence ao STORAGE?
* pertence à UI?
* pertence ao CACHE?

Não misture responsabilidades.

---

## 3. PROCURAR EFEITOS COLATERAIS

Verifique:

* caches;
* índices;
* localStorage;
* renderização;
* stats;
* sessões abertas;
* accordions;
* drag state;
* open states;
* derived state;
* invalidadores.

---

## 4. SÓ ENTÃO IMPLEMENTAR

A implementação deve:

* seguir o padrão já usado no sistema;
* manter nomenclatura consistente;
* manter comentários;
* manter a ordem arquitetural;
* manter a seção correta;
* evitar duplicação;
* evitar lógica escondida;
* evitar mutações implícitas.

---

# DOCUMENTAÇÃO É OBRIGATÓRIA

Toda alteração no BLOC DEVE atualizar a documentação correspondente.

Isso inclui:

* comentários de seção;
* blocos explicativos;
* fluxos documentados;
* observações arquiteturais;
* contratos implícitos;
* warnings;
* comentários de estado;
* comentários de cache;
* comentários de invalidação;
* TODOs relacionados;
* documentação de migração;
* roadmap interno quando necessário.

Se o comportamento mudou:
a documentação também deve mudar.

Código e documentação DEVEM permanecer sincronizados.

Documentação desatualizada é considerada bug arquitetural.

Nenhuma feature é considerada concluída sem:

* implementação;
* validação;
* atualização da documentação.

---

# REGRAS DE ARQUITETURA

## 1. NÃO criar lógica escondida

Evite:

* side-effects invisíveis;
* funções que fazem mais do que o nome indica;
* mutações implícitas;
* invalidações escondidas.

Se uma função invalida cache:
isso deve ficar explícito.

---

## 2. NÃO espalhar estado

Evite criar:

* flags redundantes;
* estados derivados persistidos;
* variáveis globais desnecessárias;
* caches duplicados.

Prefira:
estado derivado calculado.

---

## 3. NÃO quebrar o fluxo unidirecional

Fluxo correto:

Template →
Sessão →
Histórico →
Stats →
Render

Nunca faça:
Histórico alterando template.
Stats alterando sessão.
Render mutando persistência.

---

## 4. NÃO adicionar “mini-frameworks improvisados”

O sistema é:

* vanilla JS;
* monólito estruturado;
* render declarativo via template string.

Não introduza:

* padrões híbridos incoerentes;
* observers aleatórios;
* pub/sub improvisado;
* stores paralelas;
* micro-framework manual.

---

## 5. RESPEITAR A ESTRUTURA DAS SEÇÕES

Cada sistema possui lugar específico.

Não coloque:

* render em CRUD;
* navegação em persistence;
* cache em UI;
* storage em render;
* drag logic em stats.

Se precisar:
crie subseções documentadas.

---

# SOBRE PERFORMANCE

O maior gargalo atual é:
`render()` full-DOM.

Então:

* evite renders desnecessários;
* evite invalidar cache sem motivo;
* evite loops redundantes;
* evite recalcular stats em cascata;
* evite reconstruções completas se uma atualização local resolve.

MAS:
não sacrifique legibilidade por micro-otimização prematura.

---

# SOBRE CACHE

Toda alteração deve considerar:

* `_statsCache`
* `_histCache`
* `_historyIndex`
* derived state
* invalidadores

Se alterar dados:
verifique se algum cache ficou stale.

---

# SOBRE RENDER

O render do BLOC é altamente acoplado ao estado.

Antes de alterar render:
verifique:

* accordions;
* open states;
* drag placeholders;
* estados temporários;
* handlers inline;
* índices;
* render parcial vs render total.

---

# SOBRE FUTURAS FEATURES

O BLOC vai crescer.

Então:
não implemente pensando apenas no caso atual.

Sempre considere:

* escalabilidade;
* extensibilidade;
* novos tipos de bloco;
* múltiplas sessões;
* múltiplos templates;
* expansão do histórico;
* modularização futura.

---

# FORMATO DAS SUAS RESPOSTAS

Sempre responda em 4 partes:

## 1. Diagnóstico

O que será afetado.

## 2. Riscos

Quais partes podem quebrar.

## 3. Plano

Como será implementado.

## 4. Implementação

Código final.

---

# REGRA FINAL

Se existir conflito entre:

* rapidez
  e
* coerência arquitetural

priorize coerência arquitetural.

O BLOC já não é mais “só um arquivo”.
Ele agora é um sistema.
