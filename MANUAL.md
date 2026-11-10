# Manual da Fábrica Agêntica Solo (UAH - Universal Agent Harness)

Repositório: https://github.com/Hugoleosilva/agent-harness-dev-template

## 1. Intenção e benefícios

Um **harness** (esteira de governança) modular que transforma um desenvolvedor solo em Diretor de Engenharia: a IA (Claude Code, Cursor, Copilot) executa, o humano aprova.

**Problema que ataca:** viés de confirmação e perda de contexto. A IA tende a inventar funções, esquecer regras no meio do caminho e complicar o código (overengineering).

**Solução:** separar a *governança* (como a IA se comporta) da *tecnologia* (a linguagem usada), com barreiras comportamentais e verificações reais.

**Benefícios:**
- **Velocidade:** reduz bastante o tempo de entrega, mas o ganho depende do projeto e da sua revisão.
- **Custo baixo de infraestrutura:** modelos pagos para execução local e modelos gratuitos para planejamento/auditoria externa.
- **Longevidade:** se surgir nova linguagem ou modelo, o processo continua; basta atualizar `.agentconfig.json`.

## 2. Arquitetura e boas práticas

### Protocolo Multi-Agente Interno (Criar, Validar, Reparar)

1. **Builder:** implementa as regras de negócio de forma mínima e escreve os testes.
2. **Validator:** adversarial. **Roda os comandos reais** (lint, tipos, testes) e cita o resultado; revisa segurança, simplicidade e escopo.
3. **Fixer:** corrige só o ponto quebrado e devolve ao Validator. **Máximo de 3 ciclos**; depois disso para e pergunta ao humano. Nunca enfraquece um teste para passar.

Os três papéis rodam no mesmo contexto, então o risco de viés de confirmação diminui, mas não some. Para auth, pagamentos, migrations e criptografia, use também `.agent-harness/REVIEW_PROMPT.md` em uma **sessão nova**.

O humano revisa o diff final e controla o deploy (Human-in-the-Loop). Ações destrutivas, `git push`, migrations e deploys exigem aprovação.

### Boas práticas aplicadas
- **KISS:** código enxuto; menos linhas, menos bugs.
- **Clean Architecture / desacoplamento:** regra de negócio separada de UI e banco.
- **Graphify (mapeamento de erros):** em bugs, a IA escreve uma árvore textual da causa-raiz antes de corrigir.
- **Segurança:** `.gitignore` e `.env.example` evitam vazar chaves; o agente nunca grava segredos no código.

## 3. Repositório: Fábrica de Desenvolvimento

**Foco:** software determinístico (sites, apps mobile, plataformas web, sistemas corporativos, Web3).

```text
development-factory/
├── CLAUDE.md                  # Carrega o AGENTS_CORE no Claude Code
├── AGENTS.md                  # Idem para Cursor, Copilot, Codex
├── LICENSE
├── .agent-harness/
│   ├── AGENTS_CORE.md         # Cérebro comportamental (inglês + XML)
│   ├── SETUP_PROMPT.md        # Assistente de inicialização
│   ├── REVIEW_PROMPT.md       # Revisão independente (alto risco)
│   ├── presets/               # Next.js, FastAPI, Kotlin/Android
│   ├── changelog/updates.md   # Histórico datado
│   ├── mcp/                   # mcp-config.json (+ exemplo Postgres)
│   ├── memory/ctx_memory.md   # Memória de decisões (manter < ~150 linhas)
│   └── logs/{mapping,archive}/
├── .agentconfig.json          # Stack e comandos de validação
├── .env.example
└── .gitignore
```

- **AGENTS_CORE.md:** diretivas em inglês com tags XML: KISS, desacoplamento, segurança de recursos, protocolo multi-agente e regras de log.
- **.agentconfig.json:** stack ativa e comandos que a IA deve rodar (linter, compilador de tipos, testes).

## 4. Uso passo a passo

**Passo 1 - Novo projeto:** copie o harness para a raiz, abra no VS Code, inicie a IA (ex.: Claude Code). O `CLAUDE.md`/`AGENTS.md` já carrega as regras. Envie o conteúdo de `.agent-harness/SETUP_PROMPT.md`.

**Passo 2 - Entrevista:** a IA pergunta uma coisa por vez (ideia, stack, regras de negócio, arquitetura, comandos de validação) ou oferece um preset pronto. Se não souber a tecnologia, responda: *"Não sei, o que você me recomenda para esse cenário?"* A IA explica prós e contras em português. Ao final, ela resume o plano, espera seu OK e só então preenche `.agentconfig.json`.

Depois disso, dê ordens em português ou inglês. A IA aplica KISS, roda o ciclo de validação, registra o changelog com a data e entrega o resultado para o seu aval.

## 5. Idioma e economia de tokens

**Idioma híbrido:** arquivos de sistema (`AGENTS_CORE.md`, `SETUP_PROMPT.md`) em inglês; você interage no idioma que preferir.

**Por quê:** os modelos são treinados majoritariamente em inglês e o português costuma gastar **mais tokens por palavra** (acentuação e palavras longas). Como esses arquivos entram no contexto a cada sessão, mantê-los em inglês e enxutos reduz o custo de entrada. A economia varia por modelo e texto; meça no seu caso em vez de assumir um percentual fixo. Prompt caching dos provedores reduz ainda mais esse custo.

### Três níveis de interação

**Nível 1 - Português puro:** tarefas simples e consultoria.
> "Crie uma tela de login simples usando Tailwind e me diga se recomenda componentes separados para e-mail e senha."

**Nível 2 - Tags XML + português:** quando há regras de negócio que a IA não deve ignorar.
```xml
<instrucao>Crie a rota de API para salvar um novo gasto do usuário.</instrucao>
<regras_de_negocio>
- O valor do gasto nunca pode ser negativo ou zero.
- Se a categoria não for informada, salve como "Outros".
- Aplique KISS: não crie funções genéricas extras.
</regras_de_negocio>
```

**Nível 3 - Tags XML + inglês:** em geral o mais preciso e econômico, mesmo que o código final use termos em português.
```xml
<instruction>Create a new TypeScript service to handle user check-out logic.</instruction>
<business_rules>
- Total price must never be negative.
- If payment_method is "PIX", apply a 5% discount.
- Enforce KISS: no unnecessary abstractions.
</business_rules>
<current_context>
const precoTotal = 150.00; const metodoPagamento = "PIX";
</current_context>
```

**Benefícios:** limites da tarefa mais claros, menos mistura entre código antigo e novas ordens, menor custo de tokens e boa independência de idioma entre regras (inglês) e código (português).

## 6. Limites honestos

Tags XML e inglês **melhoram** a obediência e reduzem alucinações e custo, mas **não as eliminam** nem garantem 100% de conformidade. A segurança real vem de: comandos de validação executados de verdade, limite de iterações, revisão independente em trabalho de alto risco e a sua revisão final.
