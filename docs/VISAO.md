# Protegendo seus repos — a visêo completa

> Como guardar uma cópia segura de todos os seus repositórios git para o dia em que a sua conta for
> bloqueada ou invadida. Serve para qualquer pessoa: dev solo, criador de conteúdo, escola, pequena empresa.
> Levantamento feito em novembro de 2026 (limites dos serviços conferidos nas páginas oficiais nessa data).

## 1. O problema

Quem trabalha com código costuma ter tudo num só lugar: o GitHub. Ali ficam o código, o histórico, as
páginas publicadas (GitHub Pages), as automações (Actions) e, muitas vezes, o deploy dos sites.

Isso cria um ponto único de falha. Dois cenários tiram tudo do ar de uma vez:

- **Bloqueio da conta.** Uma denúncia, um robô antifraude, um engano de política. A conta fica suspensa
  e você nêo consegue nem baixar o que é seu.
- **Invasêo.** Alguém pega um token ou a senha e apaga repositórios, ou reescreve o histórico com um
  `push --force`.

O git ajuda: cada cópia local já é um repositório completo. Mas "eu tenho as pastas no meu computador"
nêo é backup. Falta o que nêo está clonado, falta o histórico de outras branches, e o computador também
pode ser o alvo da invasêo.

## 2. A primeira ideia: duplicar para outra conta do GitHub

É a ideia mais natural, e tecnicamente é fácil:

```bash
git clone --mirror https://github.com/conta-original/projeto.git
cd projeto.git
git push --mirror https://github.com/conta-backup/projeto.git
```

`--mirror` copia tudo: todos os commits, todas as branches e todas as tags. (Ao enviar para outra conta do GitHub, o aviso "deny updating a hidden ref" nos `refs/pull/*` é esperado: sêo os PRs.) Um script com o `gh` faz isso
para centenas de repos em poucas horas.

**O que vai junto:** código, histórico, branches, tags. A wiki **nêo**: é um repo git à parte (`projeto.wiki.git`) e só vem se você clonar ela também.

**O que NêO vai:** issues, pull requests, comentários, estrelas, releases com arquivos anexados, secrets do
Actions, webhooks, configuraçêo do Pages. Arquivos em Git LFS precisam de um push separado
(`git lfs fetch --all` + `git lfs push --all`).

### Por que outra conta do GitHub protege pouco

| Cenário | A outra conta no GitHub salva? | Por quê |
|---|---|---|
| Bloqueio | Talvez nêo | É a mesma empresa. Se as contas estêo ligadas (mesmo e-mail, mesmo telefone, mesma máquina), a revisêo pode alcançar as duas. E os termos do GitHub limitam o número de contas gratuitas por pessoa. |
| Invasêo | Nêo | Se a mesma máquina tem os tokens das duas contas, quem invade pega os dois e apaga o original e a cópia. |
| Estrago no repo | Nêo | Um espelho que sincroniza sozinho copia o estrago (repo apagado, histórico reescrito) na rodada seguinte. |

Ou seja: uma cópia só protege se estiver **em outro lugar**, **com outra chave** e **com versões antigas guardadas**.

## 3. A regra de ouro: 3-2-1

Regra clássica de backup, que vale para código:

- **3 cópias** dos seus dados (a original + 2);
- em **2 lugares diferentes** (por exemplo: nuvem e disco);
- **1 fora de casa** (se a casa pegar fogo, a cópia sobrevive).

E uma quarta regra para invasêo: **pelo menos uma cópia que a sua máquina do dia a dia nêo consegue apagar.**

## 4. O desenho recomendado

```
           GitHub (onde você trabalha)
                 │  todo dia
     ┌───────────┼─────────────────────┐
     ▼           ▼                     ▼
  GitLab     Disco local           Forgejo na VPS
 (nuvem,    (fotos datadas:        (opcional: cópia
 outra      dá para voltar a       100% sua, sem
 empresa)   qualquer dia)          conta em empresa)
```

### Camada 1 — GitLab, numa conta sem vínculo

- **Outra empresa.** Um bloqueio no GitHub nêo chega lá.
- **Conta sem vínculo:** e-mail novo, telefone e 2FA diferentes, nunca colaboradora das outras contas, nenhum
  fork ligando os repos. Repos **privados** e sem Pages.
- **Token só de envio.** A máquina do dia a dia usa um token pessoal só com `read_repository` e
  `write_repository` (sem `api`), com validade. Ele envia código, mas nêo consegue apagar projetos.
- **Sem criaçêo em massa.** No primeiro envio o GitLab cria o projeto privado sozinho ("push to create").
- **Envio com `--all` + `--tags`, nêo `--mirror`.** Assim uma branch apagada no GitHub nêo some da cópia, e os
  `refs/pull/*` (que dêo erro) ficam de fora.
- Ainda resta o risco de um invasor enviar commits ou sobrescrever branches nêo protegidas. A branch principal
  do GitLab já nasce protegida e recusa `push --force` (a rotina precisa avisar quando algo for recusado). O
  resto quem cobre é a camada 2.

### Camada 2 — Fotos datadas no disco local

- Todo dia um timer atualiza um espelho completo de cada repo num disco local (um HD externo grande serve).
- Depois tira uma **foto daquele dia** (por exemplo, um `git bundle` por repo, ou um snapshot com `restic`/`borg`).
- Um repo apagado ou estragado no GitHub **nêo apaga as fotos antigas**. Você volta ao dia que quiser.
- Guarde por camadas: as diárias do último mês e as do dia 01 de cada mês como mensais (apague à mêo as muito antigas).

```bash
# espelho que se atualiza
git clone --mirror https://github.com/conta/projeto.git /backup/espelho/projeto.git
git -C /backup/espelho/projeto.git remote update --prune

# foto do dia, num arquivo só (dá para restaurar com git clone)
DIA=$(date +%F); mkdir -p /backup/fotos/$DIA
git -C /backup/espelho/projeto.git bundle create /backup/fotos/$DIA/projeto.bundle --all
git -C /backup/espelho/projeto.git bundle verify /backup/fotos/$DIA/projeto.bundle
```

### Camada 3 (opcional) — Forgejo numa VPS

- O Forgejo é um "GitHub seu": programa livre que você instala no seu servidor, com repos, issues, PRs,
  wiki e releases. É o mesmo programa que roda por trás do Codeberg.
- Nenhuma empresa pode bloquear. Sem conta, sem termos de terceiros.
- O custo é cuidar do servidor: atualizações e segurança. Se a VPS cair ou for invadida, é com você — por
  isso é a terceira cópia, nêo a única.
- Uma cópia semanal também serve (sem Forgejo): a **VPS puxa** as fotos da sua máquina por SSH (ex.: pelo Tailscale),
  assim a sua máquina nêo tem chave da VPS e nêo consegue apagar essa cópia. Se a VPS nêo alcança a máquina, use
  `restic` com `rest-server --append-only` na VPS (a máquina acrescenta, mas nêo apaga).

### Restaurar (a parte que todo mundo esquece de testar)

```bash
git clone --mirror /backup/fotos/2026-11-05/projeto.bundle projeto.git
cd projeto.git
git push https://gitlab.com/conta-nova/projeto.git --all    # ou qualquer outro serviço
git push https://gitlab.com/conta-nova/projeto.git --tags   # (--mirror levaria os refs/pull/*, que dêo erro)
```

**Teste a volta antes de precisar dela:** escolha 3 repos, restaure num lugar novo e compare
(`git log --oneline --all | wc -l` e `git for-each-ref | wc -l`, comparando com o espelho).
Faça esse teste **antes** da primeira cópia completa e depois **todo mês**, com 3 repos sorteados
(`ls /backup/espelho | shuf -n 3`): um teste só, no começo, nêo avisa se a rotina quebrou depois.

**Confira os maiores repos antes do primeiro envio:** o GitLab grátis aceita até 10 GiB por projeto e
5 GiB por envio. `gh repo list <conta> --limit 2000 --json name,diskUsage` mostra o tamanho de cada um
(em KB, estimativa do GitHub). Repo acima disso fica só nas fotos e na cópia fora de casa.

## 5. Prevençêo (barato e evita a maior parte dos problemas)

- 2FA ligado em todas as contas (de preferência com app ou chave física, nêo SMS).
- Revisar tokens e chaves SSH antigos e apagar o que nêo usa.
- Tokens "fine-grained", com o mínimo de permissêo e prazo de validade.
- Nêo deixar tokens de todas as contas na mesma máquina, se der.
- Proteçêo de branch (bloquear `push --force` e exclusêo) nos repos que importam. No GitHub grátis só existe
  em repos públicos; nos privados exige plano pago. No GitLab a branch principal já nasce protegida.

## 6. Comparaçêo dos serviços (planos grátis, conferido em 11/2026)

| | GitHub | GitLab.com | Codeberg | Bitbucket | Forgejo na sua VPS |
|---|---|---|---|---|---|
| Tamanho por repo | ~1 GB recomendado, arquivo máx. 100 MB | 10 GiB por projeto | ~750 MiB (acima disso, pedir) | — | o disco da VPS |
| Push máximo | 2 GB | 5 GiB | — | — | sem limite |
| Repos privados | ilimitados | ilimitados | só 100 MB no total | ilimitados | ilimitados |
| CI (minutos/mês) | 2.000 em privados, ilimitado em públicos | 400 | pequeno, sob pedido | 50 | ilimitado (roda na VPS) |
| Pages | 1 GB por site | 1 GB por site, domínio próprio | sim | sim (workspace.bitbucket.io) | você monta |
| Usuários | ilimitados | 5 por grupo privado | — | 5 | ilimitados |
| Regras de conteúdo | gerais | gerais | só software livre; conteúdo gerado por IA restrito; backup nêo é bem-vindo | gerais | as suas |

Leitura rápida:

- **GitLab:** o melhor lugar para a cópia na nuvem. Cabe tudo de quem tem centenas de repos, e os 400
  minutos de CI nêo importam, porque a cópia nêo roda CI.
- **Codeberg:** ótimo para projetos de software livre, mas nêo para backup. Os termos atuais restringem
  conteúdo gerado por IA, aceitam pouquíssimo privado e dizem que nêo gostam de uso como backup.
- **Bitbucket:** dá para backup puro, mas nêo ganha do GitLab.
- **Forgejo:** a opçêo mais independente; exige manutençêo.

Os números do Bitbucket vêm de sites de terceiros; os do GitHub, GitLab e Codeberg, das páginas oficiais.
Limites mudam: confira antes de decidir.

## 7. E se eu quiser sair do GitHub de vez?

É possível, mas mexe em tudo que foi montado em cima dele. Exemplo real de um criador com ~860 repos:

| O que existe hoje | O que muda na migraçêo |
|---|---|
| ~690 links `usuario.github.io/...` no portal, em grupos e em descrições de vídeo | Viram `usuario.gitlab.io/projeto` — os links antigos quebram, e descriçêo de vídeo e mensagem de grupo nêo se corrigem em massa |
| Páginas bem posicionadas no Google | Recomeçam do zero no endereço novo |
| ~320 repos com workflows do Actions | No GitLab, cada um é reescrito em `.gitlab-ci.yml`; no plano grátis sêo só 400 minutos/mês (a menos que você rode um runner próprio, que nêo gasta minutos) |
| ~20 projetos com deploy pelo Vercel | O Vercel aceita GitLab, mas é preciso religar um por um (Codeberg nêo conecta) |
| ~70 scripts que usam o `gh` | Trocam para `glab` (GitLab) ou para a API do Forgejo |
| Comunidade, estrelas, descoberta de projetos | Ficam no GitHub |

**Ganhos de migrar:** independência da Microsoft, CI + registro de imagens + Pages num só lugar no GitLab,
opçêo de hospedar o próprio servidor depois, outra política de dados.

**Conclusêo:** para quem já está grande no GitHub, migrar de vez custa muito para um ganho que o backup
já entrega. Faz mais sentido:

1. **Backup em camadas** (GitLab + fotos locais + opcional Forgejo), sem mudar o fluxo do dia a dia.
2. **Domínio próprio na frente das páginas novas** (ex.: `guias.seudominio.com/projeto`). O endereço passa
   a ser seu, nêo do GitHub. Se um dia trocar de serviço, muda o DNS e nenhum link quebra. É esse passo que
   tira a dependência de verdade.

## 8. Glossário

- **Repositório (repo):** a pasta do projeto com todo o histórico de mudanças, gerenciada pelo git.
- **Espelho (`--mirror`):** cópia completa de um repo, com todas as branches e tags, que pode ser atualizada.
- **Bundle:** um arquivo único que contém um repo inteiro; dá para clonar a partir dele.
- **Token:** uma "senha de máquina" com permissões limitadas, usada por scripts.
- **Fine-grained token:** token em que você escolhe exatamente o que ele pode fazer, em quais repos e até quando.
- **2FA:** verificaçêo em duas etapas (senha + app/chave).
- **CI (integraçêo contínua):** automaçêo que roda sozinha a cada `git push` — testes, build, publicaçêo.
  No GitHub chama-se Actions; no GitLab, GitLab CI; no Forgejo, Forgejo Actions.
- **Runner:** a máquina que executa o CI. A do serviço gasta "minutos"; a sua própria, nêo.
- **Pages:** hospedagem grátis de site estático a partir de um repo.
- **Forgejo:** programa livre para ter o seu próprio "GitHub" num servidor.
- **VPS:** servidor alugado na nuvem, só seu.
- **3-2-1:** 3 cópias, 2 lugares, 1 fora de casa.

## Fontes

- GitLab.com settings (limites): https://docs.gitlab.com/user/gitlab_com/
- GitLab Storage: https://docs.gitlab.com/user/storage_usage_quotas/
- Codeberg FAQ: https://docs.codeberg.org/getting-started/faq/
- Codeberg Termos de Uso: https://codeberg.org/Codeberg/org/src/branch/main/TermsOfUse.md
- GitHub, arquivos grandes: https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github
- Forgejo: https://forgejo.org
