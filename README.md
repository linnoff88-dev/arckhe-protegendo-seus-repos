ï»¿# ê°Å¸âºÂ¡ê¯Â¸Â Protegendo seus repos

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


[![Protegendo seus repos](guia/assets/banner.jpg)](https://linnoff88-dev.github.io/arckhe-/guia/)

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

## O que êÂ©

Este êÂ© um guia para quem guarda cêÂ³digo no GitHub (ou em qualquer serviêÂ§o parecido) e quer estar pronto para o dia em que a conta for bloqueada ou invadida. Ele explica por que copiar tudo para outra conta do mesmo serviêÂ§o protege pouco e mostra um desenho em trêÂªs camadas: uma cêÂ³pia em outra empresa, fotos datadas num disco seu e, se quiser, um servidor prêÂ³prio. Traz os comandos do git para cada passo, os limites dos planos grêÂ¡tis e o que mudaria se vocêÂª saêÂ­sse do GitHub de vez. NêÂ£o precisa instalar nada para ler: êÂ© a visêÂ£o e o mêÂ©todo, prontos para vocêÂª aplicar.

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://linnoff88-dev.github.io/arckhe-/guia/**

A visêÂ£o inteira, em texto corrido: [docs/VISAO.md](docs/VISAO.md).

## A ideia em 30 segundos

- **Outra conta no GitHub protege pouco:** êÂ© a mesma empresa, os tokens costumam estar na mesma mêÂ¡quina e um espelho automêÂ¡tico copia o estrago.
- **Regra 3-2-1:** 3 cêÂ³pias, 2 lugares, 1 fora de casa ê¢â¬â e pelo menos uma cêÂ³pia que a sua mêÂ¡quina do dia a dia nêÂ£o consegue apagar.
- **TrêÂªs camadas:** GitLab numa conta sem vêÂ­nculo (token que envia mas nêÂ£o apaga) + fotos datadas num disco seu + Forgejo numa VPS (opcional).
- **Teste a volta** com 3 repos antes de precisar dela.
- **Migrar de vez** custa caro (links, Pages, CI, deploy). O que tira a dependêÂªncia de verdade êÂ© um **domêÂ­nio prêÂ³prio** na frente das pêÂ¡ginas.

## O processo, do inêÂ­cio ao fim

| # | Passo | Comando principal | Quem faz |
|---|---|---|---|
| 1 | PrevenêÂ§êÂ£o: 2FA, faxina de tokens e chaves, proteêÂ§êÂ£o de branch | painel da conta | ê°Å¸Â§Â  vocêÂª |
| 2 | Criar a conta de backup sem vêÂ­nculo (GitLab, e-mail e telefone novos) | gitlab.com ê¢â â Register | ê°Å¸Â§Â  vocêÂª |
| 3 | Gerar o token do GitLab sêÂ³ de envio (`read_repository` + `write_repository`) e o do GitHub sêÂ³ de leitura, guardados em `tokens.env` (chmod 600) | GitLab: avatar ê¢â â Edit profile ê¢â â Access tokens êÂ· GitHub: Settings ê¢â â Developer settings ê¢â â Fine-grained tokens | ê°Å¸Â§Â  vocêÂª |
| 4 | Espelho local de cada repo | `git clone --mirror <url> /backup/espelho/<repo>.git` | ê¢Å¡â¢ê¯Â¸Â script |
| 5 | Atualizar o espelho todo dia | `git -C <espelho> remote update --prune` | ê¢Å¡â¢ê¯Â¸Â timer |
| 6 | Foto do dia | `mkdir -p <data>` + `git -C <espelho> bundle create <data>/<repo>.bundle --all` | ê¢Å¡â¢ê¯Â¸Â timer |
| 7 | Enviar para o GitLab (cria o projeto privado no 1êÂº envio) | `git -C <espelho> push <url-gitlab> --all` + `--tags` | ê¢Å¡â¢ê¯Â¸Â timer |
| 8 | Antes da cêÂ³pia completa: testar a volta com 3 repos e ver os maiores (GitLab grêÂ¡tis: 10 GiB por projeto, 5 GiB por envio) | `gh repo list <conta> --json name,diskUsage` + os comandos do passo 11 | ê°Å¸Â§Â  vocêÂª |
| 9 | Primeira cêÂ³pia completa êÂ  mêÂ£o, depois a rotina no relêÂ³gio todo dia | `/backup/rotina-diaria.sh` + cron `0 3 * * *` | ê¢Å¡â¢ê¯Â¸Â timer |
| 10 | CêÂ³pia semanal fora de casa: a VPS puxa as fotos (ou Forgejo) | na VPS: `rsync -a usuario@sua-maquina:/backup/fotos/ /backup/fotos/` | ê¢Å¡â¢ê¯Â¸Â timer |
| 11 | Todo mêÂªs: testar a restauraêÂ§êÂ£o com 3 repos sorteados | `ls /backup/espelho \| shuf -n 3` + `git clone --mirror <bundle>` + `git push <url> --all` + `--tags` | ê°Å¸Â§Â  vocêÂª |

## LicenêÂ§a

Texto sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Feito pelo [ARCKHÉ AI](https://ARCKHÉ AI) ê¢â¬â conteêÂºdo aberto e gratuito.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>