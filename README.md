# vaultwarden-mirror

Espelho das imagens oficiais do [Vaultwarden](https://github.com/dani-garcia/vaultwarden) no GitHub Container Registry desta conta. O cofre em produção (`https://0xid.r16.com.br`, no Railway) puxa a imagem daqui, e não do Docker Hub, para continuar podendo fazer redeploy mesmo que o projeto oficial deixe de existir.

## O que é espelhado

| Imagem de origem (Docker Hub) | Espelho (GHCR) | Para quê |
|---|---|---|
| `vaultwarden/server:<x.y.z>` | `ghcr.io/r16rodolfo/vaultwarden:<x.y.z>` | Imagem que roda no Railway (servidor + interface web embutida) |
| `vaultwarden/web-vault:<vYYYY.M.P>` | `ghcr.io/r16rodolfo/vaultwarden-web-vault:<vYYYY.M.P>` | Interface web isolada, necessária para compilar o servidor a partir do fork [`r16rodolfo/vaultwarden`](https://github.com/r16rodolfo/vaultwarden) |

São copiadas todas as arquiteturas (amd64, arm64, arm/v7, arm/v6), sem alteração: o digest é o mesmo do upstream.

## Como funciona

1. O workflow [`mirror.yml`](.github/workflows/mirror.yml) roda todo dia às 05:17 UTC (ou manualmente em **Actions → Espelhar imagens Vaultwarden → Run workflow**).
2. Ele lê no Docker Hub a versão mais recente de cada imagem e copia para o GHCR o que ainda não existe aqui.
3. Cada versão nova é registrada em [`VERSIONS.md`](VERSIONS.md). Para o servidor, também é publicada uma **Release** neste repositório, com link para as notas oficiais.
4. O Railway está configurado com **Auto Updates** nativo: ele detecta a nova tag semver no GHCR e redeploya na janela de manutenção, depois de fazer backup do volume.

Para ser avisado de cada versão nova: **Watch → Custom → Releases** neste repositório. O Railway também notifica os admins do workspace quando aplica a atualização.

## Se o projeto oficial sumir

- O workflow passa a falhar (não acha as tags no Docker Hub). As imagens já espelhadas continuam aqui e o Railway segue redeployando a última versão.
- Para alterar o código: compilar o fork `r16rodolfo/vaultwarden` trocando, em `docker/DockerSettings.yaml` / `Dockerfile`, a imagem `docker.io/vaultwarden/web-vault@sha256:...` por `ghcr.io/r16rodolfo/vaultwarden-web-vault:<versão>`.
- O fork só tem o código até a última sincronização: use **Sync fork** nele periodicamente.

## Manutenção

- O GitHub desliga workflows agendados depois de 60 dias sem atividade no repositório. O workflow grava `.heartbeat` quando passa 40 dias sem versão nova, para evitar isso.
- Os pacotes no GHCR precisam ficar **públicos**: o Railway puxa sem credencial, e o Auto Updates do Railway não usa credenciais de registry.
