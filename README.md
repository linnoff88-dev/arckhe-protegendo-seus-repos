# ðŸ›¡ï¸ Protegendo seus repos

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


[![Protegendo seus repos](guia/assets/banner.jpg)](https://inematds.github.io/protegendo-seus-repos/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

## O que Ã©

Este Ã© um guia para quem guarda cÃ³digo no GitHub (ou em qualquer serviÃ§o parecido) e quer estar pronto para o dia em que a conta for bloqueada ou invadida. Ele explica por que copiar tudo para outra conta do mesmo serviÃ§o protege pouco e mostra um desenho em trÃªs camadas: uma cÃ³pia em outra empresa, fotos datadas num disco seu e, se quiser, um servidor prÃ³prio. Traz os comandos do git para cada passo, os limites dos planos grÃ¡tis e o que mudaria se vocÃª saÃ­sse do GitHub de vez. NÃ£o precisa instalar nada para ler: Ã© a visÃ£o e o mÃ©todo, prontos para vocÃª aplicar.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/protegendo-seus-repos/guia/**

A visÃ£o inteira, em texto corrido: [docs/VISAO.md](docs/VISAO.md).

## A ideia em 30 segundos

- **Outra conta no GitHub protege pouco:** Ã© a mesma empresa, os tokens costumam estar na mesma mÃ¡quina e um espelho automÃ¡tico copia o estrago.
- **Regra 3-2-1:** 3 cÃ³pias, 2 lugares, 1 fora de casa â€” e pelo menos uma cÃ³pia que a sua mÃ¡quina do dia a dia nÃ£o consegue apagar.
- **TrÃªs camadas:** GitLab numa conta sem vÃ­nculo (token que envia mas nÃ£o apaga) + fotos datadas num disco seu + Forgejo numa VPS (opcional).
- **Teste a volta** com 3 repos antes de precisar dela.
- **Migrar de vez** custa caro (links, Pages, CI, deploy). O que tira a dependÃªncia de verdade Ã© um **domÃ­nio prÃ³prio** na frente das pÃ¡ginas.

## O processo, do inÃ­cio ao fim

| # | Passo | Comando principal | Quem faz |
|---|---|---|---|
| 1 | PrevenÃ§Ã£o: 2FA, faxina de tokens e chaves, proteÃ§Ã£o de branch | painel da conta | ðŸ§  vocÃª |
| 2 | Criar a conta de backup sem vÃ­nculo (GitLab, e-mail e telefone novos) | gitlab.com â†’ Register | ðŸ§  vocÃª |
| 3 | Gerar o token do GitLab sÃ³ de envio (`read_repository` + `write_repository`) e o do GitHub sÃ³ de leitura, guardados em `tokens.env` (chmod 600) | GitLab: avatar â†’ Edit profile â†’ Access tokens Â· GitHub: Settings â†’ Developer settings â†’ Fine-grained tokens | ðŸ§  vocÃª |
| 4 | Espelho local de cada repo | `git clone --mirror <url> /backup/espelho/<repo>.git` | âš™ï¸ script |
| 5 | Atualizar o espelho todo dia | `git -C <espelho> remote update --prune` | âš™ï¸ timer |
| 6 | Foto do dia | `mkdir -p <data>` + `git -C <espelho> bundle create <data>/<repo>.bundle --all` | âš™ï¸ timer |
| 7 | Enviar para o GitLab (cria o projeto privado no 1Âº envio) | `git -C <espelho> push <url-gitlab> --all` + `--tags` | âš™ï¸ timer |
| 8 | Antes da cÃ³pia completa: testar a volta com 3 repos e ver os maiores (GitLab grÃ¡tis: 10 GiB por projeto, 5 GiB por envio) | `gh repo list <conta> --json name,diskUsage` + os comandos do passo 11 | ðŸ§  vocÃª |
| 9 | Primeira cÃ³pia completa Ã  mÃ£o, depois a rotina no relÃ³gio todo dia | `/backup/rotina-diaria.sh` + cron `0 3 * * *` | âš™ï¸ timer |
| 10 | CÃ³pia semanal fora de casa: a VPS puxa as fotos (ou Forgejo) | na VPS: `rsync -a usuario@sua-maquina:/backup/fotos/ /backup/fotos/` | âš™ï¸ timer |
| 11 | Todo mÃªs: testar a restauraÃ§Ã£o com 3 repos sorteados | `ls /backup/espelho \| shuf -n 3` + `git clone --mirror <bundle>` + `git push <url> --all` + `--tags` | ðŸ§  vocÃª |

## LicenÃ§a

Texto sob [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Feito pelo [INEMA.CLUB](https://inema.club) â€” conteÃºdo aberto e gratuito.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>