# Nexus Console — Decisões de Marca

Tudo que foi decidido sobre identidade visual e textual do Nexus Console.

## Identidade textual

| Campo | Valor |
|-------|-------|
| Nome do produto | **Nexus Console** |
| Nome curto | Nexus |
| Empresa | Nexus Atemporal |
| Tagline | Onde tudo se conecta do jeito certo |
| Arquétipo da marca | Sábio (autoridade técnica + clareza) |
| URL protocol | `nexus://` |
| applicationName técnico | `nexus-console` |
| Domínio final (planejado) | `console.nexusatemporal.com.br` |

## Paleta oficial (do Brand Book)

### Cores primárias

| Nome | Hex | RGB | Uso |
|------|-----|-----|-----|
| 🟠 Laranja vibrante | `#FF7300` | 255, 115, 0 | Primary, botões, links, accents |
| 🟠 Laranja escuro | `#D93D00` | 217, 61, 0 | Hover, button background |
| ⚫ Cinza escuro | `#4B4B4D` | 75, 75, 77 | Borders, chrome neutro |
| ⚪ Cinza claro | `#C9CED6` | 201, 206, 214 | Texto secundário, dividers |

### Cores secundárias

| Nome | Hex | RGB | Uso |
|------|-----|-----|-----|
| 🌑 Azul-petróleo | `#1E2A38` | 30, 42, 56 | Background dark profundo |
| 🌃 Azul médio | `#2F4F6F` | 47, 79, 111 | Sidebar/panels alternativo |
| 🟡 Dourado | `#E8D591` | 232, 213, 145 | Warnings, highlights especiais |
| ⚪ Off-white | `#F5F6F7` | 245, 246, 247 | Texto principal no dark mode |

### Variações do laranja (geradas pra IDE)

| Nome | Hex | Uso |
|------|-----|-----|
| Laranja claro accent | `#FF8C2A` | Links/highlight no editor |
| Laranja escuro hover | `#D93D00` | Button hover/active |

### Cores no tema Nexus Dark (`extensions/theme-defaults/themes/nexus-dark.json`)

Substituições aplicadas em cima do `2026-dark.json`:
- `#3994BC` → `#FF7300` (accent primário)
- `#48A0C7` → `#FF8C2A` (links)
- `#2B7DA3` → `#D93D00` (button hover)
- `#297AA0` → `#D93D00` (button bg)
- `#53A5CA` → `#FF8C2A`
- `#1E3A47` → `#332014` (input info bg)

## Tipografia oficial

- **Primária:** **Intelo** (Light, Regular, Medium, Semi Bold, Bold, Extra Bold)
- **Secundária:** **Montserrat** (mesma escala de pesos)

⚠️ Não vem com a marca por default no IDE — pra usar nas docs/UI custom da extensão Nexus AI, baixar e embutir.

## Logo

### Arquivos disponíveis (no PC do dev)

`Desktop/Nexus Atemporal/Logo Nexus Atemporal/` tem 8 variações:
1. Horizontal full color (laranja + cinza)
2. Horizontal sobre fundo escuro
3. Horizontal monocromático
4. Variação extra
5. **Vertical full color** ← usado no ícone do app
6. Vertical com texto
7. Versão branca (transparente)
8. Versão preta

### Usado neste projeto

**Símbolo isolado** (sem texto), em `Desktop/Nexus Atemporal/one nexus atemporal/nexusatemporal/frontend/src/assets/images/logo-icon.png` — o melhor pra ícone de app porque o símbolo é forte mesmo em 16×16.

### Onde foi aplicado

Todos gerados via Python+Pillow a partir do `logo-icon.png`:

| Arquivo | Tamanho(s) |
|---------|-----------|
| `resources/win32/code.ico` | Multi-res 16, 24, 32, 48, 64, 128, 256 |
| `resources/win32/code_70x70.png` | 70×70 |
| `resources/win32/code_150x150.png` | 150×150 |
| `resources/linux/code.png` | 512×512 |
| `resources/server/code-192.png` | 192×192 (PWA) |
| `resources/server/code-512.png` | 512×512 (PWA) |
| `resources/server/favicon.ico` | Multi-res 16-256 |
| `resources/darwin/code.png` | 1024×1024 (placeholder; `.icns` gerado no build macOS) |

## Communication style (do Brand Book)

### Como a Nexus DEVE falar
- Clara e objetiva (sem jargão desnecessário)
- Segura e confiante (domínio do mercado)
- Didática e estratégica (benefícios, não features)
- Profissional e estruturada
- Inspiradora porém racional

### Como a Nexus NÃO DEVE falar
- ❌ Linguagem informal excessiva ou gírias
- ❌ Promessas exageradas
- ❌ Comunicação genérica
- ❌ Excesso de tecnicismo sem tradução prática

⚠️ **Aplicar isso em copy futura** — UI de extensão Nexus AI, mensagens de erro, welcome page customizada, docs.

## Elementos a não esquecer

- Telemetria sempre off (`enableTelemetry: false`)
- License: MIT (do upstream — não mexer)
- Copyright: respeitar atribuição Microsoft/Antigravity em arquivos originais; adicionar copyright Nexus em código novo

## Arquivos com branding aplicado (referência git)

```bash
git log --oneline | grep -i nexus
# Espera ver:
# 58deb7293c8 feat(web): product.overrides.json com branding Nexus Console
# 8ad9b73b243 chore: ignore local build helpers
# f6c10d0c513 feat(branding): rebrand para Nexus Console
```

```bash
git show f6c10d0c513 --stat
# product.json, theme, ícones, theme-defaults package.json/nls.json
```
