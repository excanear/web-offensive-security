# Web Offensive Security — Claude Code Skill

Uma [skill do Claude Code](https://docs.claude.com/en/docs/claude-code/skills) que transforma o agente em um **auditor sênior de AppSec / Red Team** para encontrar o máximo de vulnerabilidades **reais e exploráveis** em aplicações web — com análise estática, validação dinâmica por PoC segura, correlação de cadeias de ataque e relatório profissional.

> ⚠️ **Uso autorizado apenas.** Esta skill foi desenhada para auditar aplicações dentro de um escopo explicitamente autorizado (seu próprio projeto, ambientes locais, engajamentos de pentest com permissão). Ela **não** ataca sistemas de terceiros, produção não autorizada ou alvos externos, e suas PoCs são não-destrutivas por design.

## O que ela cobre

Rastreio **source → sink** em ~20 classes de vulnerabilidade:

- **Injection:** SQL / NoSQL, Command Injection / RCE, SSTI, XXE
- **Client-side:** XSS (refletido / armazenado / DOM), CSRF, CORS, clickjacking, headers de segurança
- **Acesso:** Autenticação, sessão/cookies, Autorização (IDOR / BOLA / escalonamento de privilégio)
- **Identidade:** JWT (`alg:none`, confusão de algoritmo), OAuth / OIDC / SSO
- **Servidor:** SSRF, Path Traversal / LFI, Upload de arquivos, Desserialização insegura
- **Exposição:** Secrets, dados sensíveis, source maps, endpoints de debug
- **Negócio:** Race conditions, replay, bypass de workflow, validação client-side
- **Cadeia/infra:** Dependências com CVE, configuração, IaC/containers, GraphQL

## Fluxo em 5 fases

1. **Recon & escopo** — confirma autorização, mapeia a stack e constrói a *attack surface* (rotas, entradas, fronteiras de confiança).
2. **Análise estática** — percorre o catálogo de vulnerabilidades categoria por categoria, rastreando o fluxo de dados até os sinks perigosos.
3. **Validação dinâmica** (apenas em ambiente local autorizado) — confirma candidatos com **PoCs seguras e não-destrutivas**.
4. **Correlação** — combina achados em cadeias de ataque ponta a ponta (ex.: *open redirect → vazamento de token OAuth → account takeover*).
5. **Relatório** — gera documento executivo + técnico e artefato estruturado.

## Saídas

| Arquivo | Conteúdo |
|---------|----------|
| `security-audit/REPORT.md` | Executive Summary, Attack Surface, findings (evidência/PoC, arquivo·função·rota·linha, severidade, CVSS 3.1, impacto, correção), cadeias de ataque, prioridade de remediação, **Security Score** e checklist de validação. |
| `security-audit/findings.json` | Findings estruturados segundo [`references/findings-schema.json`](references/findings-schema.json), prontos para uma skill defensiva / de remediação consumir diretamente. |

## Regras de engajamento (não-negociáveis)

- **Somente escopo autorizado** — nunca sistemas externos, de terceiros ou produção não autorizada.
- **Não-destrutivo** — sem destruição de dados, persistência, exfiltração real ou impacto em disponibilidade. PoCs read-only ou benignas (`id`, `SELECT 1`, marcador canário local).
- **Teste dinâmico é opt-in** — só roda contra alvo quando o usuário confirma ambiente autorizado.
- **Honestidade de evidência** — findings estáticos não confirmados ficam marcados `"verified": false`; a aplicação nunca é declarada segura sem evidência suficiente.

## Instalação

Copie a pasta para o diretório de skills do Claude Code:

```bash
# Skill de usuário (global)
cp -r web-offensive-security ~/.claude/skills/

# ou skill de projeto
cp -r web-offensive-security .claude/skills/
```

No Windows (PowerShell):

```powershell
Copy-Item -Recurse web-offensive-security "$env:USERPROFILE\.claude\skills\"
```

## Uso

```
/web-offensive-security
```

ou simplesmente peça: *"audite a segurança desta aplicação"* / *"faça um pentest deste projeto"*. O agente confirmará o escopo autorizado antes de qualquer teste dinâmico.

## Estrutura

```
web-offensive-security/
├── SKILL.md                      # instruções principais + regras de engajamento
└── references/
    ├── methodology.md            # enumeração de surface + rastreio source→sink
    ├── vulnerability-catalog.md  # o que procurar, sinks e bypasses por classe
    ├── safe-poc-patterns.md      # receitas de PoC não-destrutiva por classe
    ├── report-template.md        # estrutura do relatório + fórmula do Security Score
    └── findings-schema.json      # JSON Schema do output estruturado
```

## Licença

[MIT](LICENSE)
