# Galeria — WR Barão do Rio Branco

Galeria de imagens/plantas/vídeo para injeção via script no 3DVista, hospedada no AWS S3.

**Empreendimento:** Barão do Rio Branco — WR
**Tema:** cinza-chumbo + dourado (`#646461` / `#2B2A27`)

---

## URLs de produção

| Arquivo | URL |
|---------|-----|
| Galeria | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/wr/galeria-wr-barao-do-rio-branco/index.html` |
| Vídeo   | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/wr/galeria-wr-barao-do-rio-branco/video-gallery.html` |
| Script  | `https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/wr/galeria-wr-barao-do-rio-branco/inject.js` |

**S3 path:** `s3://skylineip/Tour Virtual/wr/galeria-wr-barao-do-rio-branco/`

---

## Estrutura de arquivos

```
galeria-wr-barao-do-rio-branco/
├── index.html              ← galeria de imagens (auto-suficiente)
├── video-gallery.html      ← player do vídeo de apresentação
├── inject.js               ← loader leve para injeção no 3DVista
├── deploy.ps1              ← deploy para o S3 (Windows; normaliza case + sync)
├── generate_thumbs.ps1     ← gerador de thumbnails (Windows/GDI+, sem deps)
└── assets/
    ├── fachada/            ← Fachada (1-5) e Foto Inserção (1-2)
    ├── tipo-cobertura/     ← Tipo (1-11) e Cobertura (1-2)
    ├── areas-comuns/       ← academia, garagem, gourmet, hall, lazer, pet place,
    │                          playground, salão de festas
    └── thumbs/             ← gerado automaticamente (espelha a árvore, .jpg)
```

> **Pastas em minúsculas e sem acento** (o S3 é case-sensitive) — nomes de arquivo com
> acento precisam estar em Unicode NFC (precomposto), não NFD.

---

## Categorias da galeria

| Categoria | Label | Sub-categorias | Pasta |
|-----------|-------|----------------|-------|
| `fachada` | Fachada | — | `assets/fachada/` |
| `apartamentos` | Apartamentos | Tipo · Cobertura | `assets/tipo-cobertura/` |
| `areas-comuns` | Áreas Comuns | — | `assets/areas-comuns/` |

O modo `plantas` do código permanece, mas não há plantas cadastradas neste projeto.

---

## Thumbnails

Todo `src` da grade usa um `thumb` leve (**900 px, JPEG q82**, achatado sobre branco).
Geração **sem dependências** no Windows via GDI+:

```powershell
./generate_thumbs.ps1          # gera apenas os que faltam / desatualizados
./generate_thumbs.ps1 -Force   # regenera todos
```

O script espelha `assets/` em `assets/thumbs/` (sempre `.jpg`, caminho minúsculo),
ignora `thumbs/`, `video/` e o PDF.

---

## Deploy AWS S3

> **Windows:** use o script pronto `./deploy.ps1`. Requer **AWS CLI** + `aws configure` (região `sa-east-1`).

```powershell
./deploy.ps1              # sync completo (sem o vídeo) + cache-control no HTML/JS
./deploy.ps1 -WithVideo   # inclui o vídeo de apresentação (~200 MB) — necessário na 1ª vez / ao trocá-lo
./deploy.ps1 -QuickHtml   # atualiza só index.html e inject.js (rápido)
```

O vídeo fica de fora do sync padrão (o `--exclude` também o protege do `--delete`),
então uma vez enviado com `-WithVideo` ele permanece no S3 nas próximas sincronizações.

> **`--cache-control "no-cache,no-store,must-revalidate"`** no HTML/JS — garante que o
> 3DVista nunca sirva uma versão cacheada da galeria ou do script.

---

## Integração 3DVista

### Passo 1 — Loader (JavaScript global do projeto)

```js
(function(){
  var s = document.createElement('script');
  s.src = 'https://skylineip.s3.sa-east-1.amazonaws.com/Tour+Virtual/wr/galeria-wr-barao-do-rio-branco/inject.js?v=' + Date.now();
  document.head.appendChild(s);
})();
```

> **`?v=` + `Date.now()`** — cache-busting: força o browser a baixar sempre a versão mais
> recente do script, evitando que o 3DVista sirva uma versão antiga em cache.

### Passo 2 — Acionar nos hotspots/botões

```js
GaleriaImagens(1);      // abre galeria de imagens   · GaleriaImagens(0) fecha
AbrirGaleriaVideos(1);  // abre o vídeo de apresentação · AbrirGaleriaVideos(0) fecha
```

> `video-gallery.html` aponta para `assets/video/apresentacao.mp4`, que ainda não foi
> enviado neste projeto — envie o arquivo do vídeo teaser nessa pasta antes de usar
> `AbrirGaleriaVideos` / `-WithVideo`.

---

## Cores e tipografia

Tema **cinza-chumbo + dourado** da WR Barão do Rio Branco:

| Token CSS | Valor | Papel |
|-----------|-------|-------|
| `--bg` (fundo) | `#646461` | Cinza-chumbo — fundo principal |
| `--surface` | `#706E6B` | Superfície de card (skeleton) |
| `--dark` (foreground) | `#F5F1EA` | Texto/ícones — quase-branco |
| `--accent` | `#2B2A27` | Chumbo quase-preto — destaque / estado ativo |
| Pílulas de filtro | `#F9D59D` | Dourado — fundo padrão dos botões de filtro |
| Lightbox | `#1c1c1a` | Quase-preto — palco da imagem/vídeo |
| Fonte títulos | Cormorant Garamond | — |
| Fonte UI | Inter | — |

> Cards de plantas mantêm **fundo branco** (legibilidade do desenho), com texto chumbo.
