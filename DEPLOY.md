# Deploy no Railway

Registro de como o cofre `https://0xid.r16.com.br` está implantado. Nenhum segredo fica neste arquivo.

## Recursos

| Item | Valor |
|---|---|
| Workspace | R16 Bits (plano Hobby) |
| Projeto | `vaultwarden` (`46bc8844-562f-4c11-9531-d92ae1086a53`) |
| Ambiente | `production` (`108bf809-9241-456f-898f-a3a7ebb46219`) |
| Serviço | `vaultwarden` (`fa2880a5-2f04-4aa3-a890-368ce560a776`) |
| Origem | imagem `ghcr.io/r16rodolfo/vaultwarden:<x.y.z>` (espelho deste repo) |
| Volume | `vaultwarden-volume`, montado em `/data`, 5 GB, SQLite |
| Domínio | `0xid.r16.com.br` → porta 8080 |
| Healthcheck | `GET /alive`, timeout 120 s, restart on failure (10 tentativas) |

## Variáveis

| Variável | Valor | Motivo |
|---|---|---|
| `DOMAIN` | `https://0xid.r16.com.br` | URLs geradas, WebAuthn/passkeys, anexos |
| `SIGNUPS_ALLOWED` | `false` | Cadastro só por convite |
| `INVITATIONS_ALLOWED` | `true` | Convites pelo painel `/admin` |
| `SHOW_PASSWORD_HINT` | `false` | Não expor dica de senha na tela de login |
| `ADMIN_TOKEN` | hash argon2id (OWASP: m=19456, t=2, p=1) | Acesso ao `/admin`. O token em texto fica só com o dono. |
| `ROCKET_PORT` / `PORT` | `8080` | Porta interna do container |
| `DATA_FOLDER` | `/data` | Banco, chaves e anexos no volume |
| `TZ` | `America/Sao_Paulo` | Horário dos logs |

Sem SMTP: o convidado não recebe e-mail. Ele se cadastra em `https://0xid.r16.com.br` usando o e-mail convidado, e o aviso é feito por fora.

## DNS (Cloudflare, zona `r16.com.br`)

| Tipo | Nome | Conteúdo | Proxy |
|---|---|---|---|
| CNAME | `0xid` | `dgt0x8xa.up.railway.app` | DNS only (nuvem cinza) |
| TXT | `_railway-verify.0xid` | `railway-verify=6c924da61ffeafe721d18a5606c71296f63b06d8dc58d608f1a2d8dc37ba04f2` | — |

## Atualizações

1. O workflow [`mirror.yml`](.github/workflows/mirror.yml) copia toda versão nova do Docker Hub para o GHCR, diariamente.
2. Auto Updates nativo do Railway, configurado no painel (Service → Settings → Source → Configure Auto Updates): **minor updates and patches**, janela **Night (02:00–06:00 UTC)**.
3. Aviso: Release neste repositório + notificação do Railway aos admins do workspace.

## Questões em aberto

- **Backup do volume.** No plano Hobby o Railway não faz backup do volume, nem agendado nem antes do auto update (os dois são exclusivos do Pro). Hoje a única cópia fora do servidor é o cache local de cada cliente Bitwarden e as exportações manuais. Opções: assinar o Pro e ligar backups DAILY/WEEKLY/MONTHLY, ou montar um job próprio que copie `/data/db.sqlite3` para armazenamento externo.
