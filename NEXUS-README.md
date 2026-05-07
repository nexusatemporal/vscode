# Nexus Console — README de Continuação

> Este documento é o ponto de entrada pra qualquer Claude Code (ou dev humano) que pegar esse repo a partir de agora.

## O que é isso

**Nexus Console** é um IDE customizado da **Nexus Atemporal**, fork do VSCode (Antigravity edition) com branding completo Nexus, pensado pra rodar **embutido como iframe** dentro do sistema Nexus principal, hospedado na VPS, com acesso via browser.

A ideia: cada cliente Nexus, quando logado no sistema, vai ter um **IDE no browser** com workspace persistente próprio, alimentado por **Ollama** (local ou cloud) e **OpenCode** como agent backend.

## Stack final pretendida

```
┌─────────────────────────────────────────┐
│  Sistema Nexus (frontend, na VPS)       │
│   └─ <iframe src="https://console..."/> │
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│  Nexus Console (VSCode fork branded)    │
│   ├─ Server VSCode embutido             │
│   ├─ Auth via token do sistema Nexus    │
│   ├─ Workspace por user (/data/...)     │
│   ├─ Extensão "Nexus AI" embutida       │
│   │   ├─ Conecta Ollama (Cloud/local)   │
│   │   └─ Agent loop OpenCode            │
│   └─ Branding: cores + ícone Nexus      │
└─────────────────────────────────────────┘
```

## Onde estamos agora

✅ **S0 — Setup & Branding** — concluída em ambiente Windows  
✅ **S1 — Web Build Target** — concluída, IDE roda no browser branded  
🔜 **S2 — Code-Server Layer** — **PRÓXIMA, será feita na VPS Linux**  
⏳ S3 — Extensão Nexus AI (Ollama)  
⏳ S4 — Integração OpenCode  
⏳ S5 — Embed e Deploy  

**Detalhes completos de cada sprint:** veja [`NEXUS-SPRINTS.md`](./NEXUS-SPRINTS.md).

## Documentos pra ler antes de mexer

| Doc | Pra quê |
|-----|---------|
| [`NEXUS-README.md`](./NEXUS-README.md) | Este aqui — overview |
| [`NEXUS-SETUP.md`](./NEXUS-SETUP.md) | Como buildar/rodar na VPS Linux |
| [`NEXUS-SPRINTS.md`](./NEXUS-SPRINTS.md) | Histórico do que foi feito + plano S2-S5 detalhado |
| [`NEXUS-BRANDING.md`](./NEXUS-BRANDING.md) | Decisões de marca: cores, nomes, ícones, paleta |

## Branches relevantes

- **`main`** — atualizada com tudo da S0 + S1 mergeado
- `feature/nexus-branding` — commits da S0 (rebranding)
- `feature/nexus-web-build` — commits da S1 (web build target)

## Identidade do projeto

- **Nome do produto:** Nexus Console
- **applicationName:** `nexus-console`
- **URL protocol:** `nexus://`
- **Cores principais:** laranja `#FF7300` + cinza `#4B4B4D`
- **Tipografia oficial:** Intelo (primária) + Montserrat (secundária)
- **Empresa:** Nexus Atemporal (CEO Magdiel Pompeu)
- **Email:** contato@nexusatemporal.com.br

## Quick start na VPS

```bash
# 1. Clone
git clone https://github.com/nexusatemporal/vscode.git nexus-console
cd nexus-console

# 2. Setup deps (ver NEXUS-SETUP.md pra detalhes)
nvm install 22.22.1 && nvm use 22.22.1
sudo apt install build-essential python3

# 3. Install
npm install --no-audit --no-fund

# 4. Compile
npm run compile

# 5. Run web (modo dev)
./scripts/code-web.sh --port 8088 --host 0.0.0.0

# 6. Acesse: http://VPS_IP:8088/
```

Pra rodar como server real (auth + persistência), siga **S2** em [`NEXUS-SPRINTS.md`](./NEXUS-SPRINTS.md).

## Contato técnico

Repo: https://github.com/nexusatemporal/vscode
