# agent-skills

Biblioteca única de skills compartilhada pelos agentes de código desta máquina.

`skills/` é a **fonte**. Cada ferramenta enxerga as mesmas skills por symlink de
diretório, então editar uma skill aqui vale para todas de imediato — não há cópia
para sincronizar.

```
~/.agents/skills/<nome>/
      ▲       ▲       ▲       ▲
      │       │       │       │
~/.claude  ~/.codex  ~/.cursor  ~/.grok
   /skills   /skills   /skills   /skills
```

| ferramenta | entradas | observação |
|---|---|---|
| Claude Code | 115 | todas por symlink |
| Codex | 119 | + `.system` e overrides locais |
| Cursor | 116 | + override local de `impeccable` |
| Grok | 116 | + overrides locais |

As contagens acima passam de 115 porque cada ferramenta pode ter uma pasta **real**
com o mesmo nome de uma skill da biblioteca. Nesse caso a pasta real vence: é um
override local, e é intencional.

## Procedência

`.skill-lock.json` registra de onde veio cada skill instalada a partir de um
repositório de terceiros — 95 das 115. As outras 20 existem só aqui.

| origem | skills |
|---|---|
| `danielvm-git/bigpowers` | 81 |
| `DietrichGebert/ponytail` | 6 |
| `anthropics/skills`, `mattpocock/skills`, `vercel-labs/agent-skills`, `shadcn/ui`, e outros | 1 cada |
| criadas localmente | 20 |

Cada skill de terceiros mantém a licença do projeto de origem. Este repositório é
privado justamente por conter cópia de código alheio.

## Sincronização automática

Três unidades systemd mantêm este repositório em dia sem intervenção:

| unidade | papel |
|---|---|
| `agent-skills-sync.path` | dispara ao criar/remover algo em `skills/` |
| `agent-skills-sync.timer` | varredura a cada 30 min |
| `agent-skills-sync.service` | roda `~/bin/agent-skills-sync` |

O script espera 30 s para a rajada de escritas assentar, commita com um resumo do
que mudou e empurra. Push que falha preserva o commit local e a execução seguinte
tenta de novo.

**O `.path` só vê o diretório de topo.** Editar `skills/<nome>/SKILL.md` não
dispara o evento — quem pega esse caso é o timer, com atraso de até 30 minutos. O
systemd não faz watch recursivo.

```bash
journalctl -u agent-skills-sync -f                                   # acompanhar
sudo systemctl disable --now agent-skills-sync.path agent-skills-sync.timer   # desligar
```

## Adicionar uma skill

Crie `skills/<nome>/SKILL.md` com frontmatter `name` e `description`. Ela aparece
nas quatro ferramentas na hora, e o sync a empurra sozinho.

Para expor a skill a uma ferramenta que ainda não a tem:

```bash
ln -s ../../.agents/skills/<nome> ~/.claude/skills/<nome>
```

## Restaurar noutra máquina

```bash
git clone git@github.com:marceloterra1983/agent-skills.git ~/.agents
for t in claude codex cursor grok; do
  mkdir -p ~/.$t/skills
  for s in ~/.agents/skills/*/; do
    n=$(basename "$s"); [ -e ~/.$t/skills/$n ] || ln -s "../../.agents/skills/$n" ~/.$t/skills/$n
  done
done
```
