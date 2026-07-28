# Portal de Curadoria — MIND.Funga

Páginas estáticas (GitHub Pages) onde uma pessoa **completa** a solicitação para participar
da curadoria do MIND.Funga: informa um telefone de contato e aceita os termos de
responsabilidade.

**No ar em:** <https://mindfunga.github.io/portalCuradoria/>

> 🔴 **Enviar o formulário NÃO torna ninguém curador.** Ele consome o token do e-mail, grava
> telefone + aceite e move a solicitação para **"aguardando análise"**. Quem promove é uma
> pessoa, depois, no painel administrativo. Esta página é um **filtro** — ela faz o clicador
> impulsivo desacelerar antes que um humano gaste atenção com o pedido dele — **não um portão**.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | O formulário. Lê o `?token=` da URL, valida telefone + aceite e chama a API. |
| `termos-curadoria.html` | Termos de uso da curadoria — **responsabilidades de quem valida**. |
| `privacidade-curadoria.html` | O *delta* de privacidade deste fluxo (só o telefone). Complementa a política geral. |
| `logo.svg` | O logo do app. **Mesmo arquivo** nos repositórios de páginas. |

## O fluxo inteiro

```
app: botão "Solicitação de curador"        (ticket 22, mindFungaApp)
  → POST /api/usuario/curadoria/solicitar  (ticket 21, backend)
  → e-mail com token de 40 hex, válido 48h, uso único
  → ESTA PÁGINA: telefone + aceite
  → POST /api/usuario/curadoria/confirmar  (ticket 21, backend)
  → "aguardando análise"
  → administrador aprova/recusa no painel  (ticket 24, admin Flutter)
```

## ⚠️ O endereço é contrato com o backend

O e-mail é montado em `backend/src/controllers/CuradoriaController.ts` a partir da variável de
ambiente **`PORTAL_CURADORIA_URL`** (`https://mindfunga.github.io/portalCuradoria`, **sem barra
no fim**), e o link fica `${PORTAL_CURADORIA_URL}/?token=…` — ou seja, **o token chega na raiz,
no `index.html`**.

**Nunca renomeie** o repositório, a pasta ou qualquer um destes arquivos: quebra os e-mails já
enviados (tokens valem 48h), os próximos, e qualquer citação externa. Endereços públicos deste
projeto já acabaram referenciados fora dele — as páginas de exclusão de conta hoje constam na
ficha da Play Store.

## 🔴 A armadilha da API

**O backend responde HTTP 200 mesmo em erro.** O `index.html` ramifica pelo campo **`error` do
JSON**, nunca por `response.ok` nem pelo status. Se olhar o status, a página diz "pronto!" para
uma solicitação que falhou — e a pessoa nunca mais volta.

## 🔴 Nada de CPF

A seção de termos nasceu quando o plano previa coletar **CPF**. O CPF foi **cortado em
2026-07-27** (LGPD): ficou **só o telefone**, gravado em `usuario.telefone` (coluna que já
existia, nullable). Não acrescente campo nem menção a CPF em lugar nenhum.

## Mexendo nas páginas

Visual do `app_design.md` §6, herdado do `recuperacao/` (é o mesmo padrão "e-mail com token →
página estática → POST no backend", já em produção — ver `PASSWORD_RESET_FLOW.md`). HTML + CSS
num arquivo só, **sem nenhuma requisição externa** (nada de Google Fonts).

> ⚠️ **A armadilha da escala.** As páginas dimensionam tudo em `rem` a partir de
> `font-size: clamp(14.4px, calc(100vw / 390 * 16), 18.4px)` no `:root`. **Nunca** troque por
> `--escala: clamp(0.9, calc(100vw / 390), 1.15)`: `100vw/390` devolve um *comprimento*, o
> `clamp` mistura tipos, a declaração é descartada e **todo `calc()` que dependia dela cai
> junto** — bordas, alturas e espaçamentos somem de uma vez. Falha **silenciosa e total**.

Por isso, **renderize depois de editar**:

```bash
chromium --headless --no-sandbox --disable-gpu --hide-scrollbars \
  --force-device-scale-factor=2 --window-size=430,1800 \
  --screenshot=/tmp/check.png "file://$PWD/index.html?token=teste"
```

*(sem `?token=…` o `index.html` mostra, corretamente, a tela de "link inválido".)*

## Publicação

GitHub Pages publica **no push** para `main`, sem build.

⚠️ **Confirme nas configurações do repositório no GitHub** que o Pages está ligado
(*Settings → Pages → Deploy from a branch → `main` / `/ (root)`*) — este repositório nasceu
com um `index.html` vazio e pode nunca ter tido o Pages habilitado. Depois de publicar, abra a
URL de verdade: o `file://` não pega problema de caminho relativo (`logo.svg`).

**Identidade git deste repositório é local**, nunca global:

```bash
git config --local user.name  MINDFunga
git config --local user.email mind.funga.app@gmail.com
```
