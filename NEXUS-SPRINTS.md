# Nexus Console — Plano de Sprints

Histórico das sprints feitas + plano detalhado das pendentes. Use isso como **roadmap mestre**.

---

## ✅ Sprint 0 — Setup & Branding (CONCLUÍDA)

**Quando:** 2026-05-07 (ambiente Windows)  
**Commit:** `f6c10d0c513` (`feat(branding): rebrand para Nexus Console`)

### O que foi entregue

- ✅ `product.json` rebrandead pra "Nexus Console"
  - nameShort: "Nexus" / nameLong: "Nexus Console"
  - applicationName: `nexus-console`
  - urlProtocol: `nexus://`
  - GUIDs Windows novos (instalador limpo)
  - darwinBundleIdentifier: `com.nexusatemporal.nexusconsole`
  - reportIssueUrl → fork Nexus
  - enableTelemetry: false

- ✅ Tema **Nexus Dark** criado em `extensions/theme-defaults/themes/nexus-dark.json`
  - Cores swap: teal `#3994BC` → laranja Nexus `#FF7300`
  - Registrado em `package.json` da extensão
  - Adicionado em `package.nls.json`
  - É a 1ª opção em onboardingThemes

- ✅ Ícones gerados a partir do logo Nexus (símbolo isolado):
  - `resources/win32/code.ico` (multi-res 16/24/32/48/64/128/256)
  - `resources/win32/code_70x70.png`, `code_150x150.png`
  - `resources/linux/code.png` (512×512)
  - `resources/server/code-192.png`, `code-512.png`, `favicon.ico`
  - `resources/darwin/code.png` (1024×1024 — `.icns` gerado no build)

- ✅ Compilação com **0 erros** validada (5 min, 6977 arquivos out/)

### Arquivos de marca (referência)

Ver [`NEXUS-BRANDING.md`](./NEXUS-BRANDING.md) pra paleta completa, lista de logos, etc.

---

## ✅ Sprint 1 — Web Build Target (CONCLUÍDA)

**Quando:** 2026-05-07  
**Commit:** `58deb7293c8` (`feat(web): product.overrides.json com branding Nexus Console`)

### O que foi entregue

- ✅ Investigado: `@vscode/test-web` é a tool dev pra rodar IDE no browser
- ✅ Servidor web rodando em `localhost:8088`
- ✅ `product.overrides.json` na raiz com branding aplicado em runtime web
- ✅ Title bar e Welcome page mostram "Nexus Console" em vez de "Code - OSS"

### Pendências documentadas

⚠️ **Tema default Nexus Dark não pega no test-web**
- O `configurationDefaults.workbench.colorTheme` é ignorado pelo `@vscode/test-web`
- **Como resolver na S2:** quando usarmos o server real (`scripts/code-server.sh`), forçar via settings.json built-in
- **Workaround imediato:** o user pode trocar manualmente via Cmd+K Cmd+T → "Nexus Dark"

⚠️ **3 extensões web não compiladas:** `merge-conflict`, `git-base`, `emmet`
- Cosmético, não bloqueia
- **Como resolver:** rodar `npm run gulp compile-web-extensions-build` antes de servir

### Como rodar (referência)

```bash
./scripts/code-web.sh --port 8088 --host 0.0.0.0
```

---

## 🔜 Sprint 2 — Code-Server Layer (PRÓXIMA)

**Onde será feita:** VPS Linux (não dá pra continuar no Windows do dev)

### Objetivo

Pegar o IDE branded e colocar dentro de um **servidor real** com:
- **Auth** (só user logado no Nexus acessa)
- **Workspace persistente por user**
- **HTTPS** (cert válido)
- **Embed em iframe** no sistema Nexus existente
- **Docker pra deploy** padronizado

### Definition of Done

- [ ] `scripts/code-server.sh` sobe sem erros
- [ ] Login obrigatório (token mínimo, OAuth ideal)
- [ ] Cada user tem `/data/nexus-console/workspaces/<userId>/` isolado
- [ ] Reverse proxy (Traefik) com HTTPS
- [ ] IDE carrega dentro de `<iframe>` do sistema Nexus
- [ ] Imagem Docker pronta
- [ ] Deploy documentado
- [ ] Branch `feature/nexus-server` pushada

### Fases detalhadas

#### S2.1 — Investigação dos 2 caminhos
**Decisão crítica:** vamos pelo **server embutido no fork** (`scripts/code-server.sh`), NÃO pelo `coder/code-server`.  
**Razão:** o coder/code-server usa VS Code OSS upstream — não pega nosso fork branded.

#### S2.2 — Subir server local
```bash
./scripts/code-server.sh --port 8088 --connection-token nexus-test
# Validar: editor abre, salva arquivo real, terminal funciona
```

#### S2.3 — Auth via token
```bash
./scripts/code-server.sh --connection-token <token-único-por-user>
```
- Request sem token → 401
- Token correto → entra
- Posteriormente: integrar com JWT do sistema Nexus

#### S2.4 — Workspace por user
Layout esperado:
```
/data/nexus-console/
├── workspaces/<userId>/   ← arquivos do user
└── user-data/<userId>/    ← settings, themes, extensions
```
Comando:
```bash
./scripts/code-server.sh \
  --user-data-dir /data/nexus-console/user-data/<userId> \
  --extensions-dir /data/nexus-console/user-data/<userId>/extensions \
  /data/nexus-console/workspaces/<userId>
```

#### S2.5 — Dockerfile multi-stage
```dockerfile
FROM node:22.22.1-bookworm AS builder
WORKDIR /build
COPY . .
RUN npm install --no-audit --no-fund && npm run compile

FROM node:22.22.1-bookworm-slim AS runtime
WORKDIR /app
COPY --from=builder /build/out ./out
COPY --from=builder /build/extensions ./extensions
COPY --from=builder /build/resources ./resources
COPY --from=builder /build/scripts ./scripts
COPY --from=builder /build/product.json ./
COPY --from=builder /build/product.overrides.json ./
COPY --from=builder /build/package.json ./
COPY --from=builder /build/node_modules ./node_modules
EXPOSE 8088
CMD ["./scripts/code-server.sh", "--port", "8088", "--host", "0.0.0.0"]
```

#### S2.6 — Reverse proxy + HTTPS (Traefik)
```yaml
services:
  nexus-console:
    image: nexus-console:latest
    labels:
      - traefik.enable=true
      - traefik.http.routers.nexus-console.rule=Host(`console.nexusatemporal.com.br`)
      - traefik.http.routers.nexus-console.tls.certresolver=letsencrypt
      - traefik.http.services.nexus-console.loadbalancer.server.port=8088
    volumes:
      - /data/nexus-console:/data/nexus-console
    environment:
      - CONNECTION_TOKEN=${NEXUS_CONNECTION_TOKEN}
```

#### S2.7 — Iframe embed
- Configurar `Content-Security-Policy: frame-ancestors https://one.nexusatemporal.com.br`
- Remover `X-Frame-Options: DENY` do server
- Validar postMessage handshake parent (Nexus) ↔ child (IDE)

```html
<!-- No sistema Nexus, página do IDE -->
<iframe src="https://console.nexusatemporal.com.br/?tkn=USER_TOKEN" 
        width="100%" height="100%" 
        sandbox="allow-scripts allow-same-origin allow-forms"></iframe>
```

#### S2.8 — Push
```bash
git checkout -b feature/nexus-server
git add .
git commit -m "feat(s2): server real + auth + workspace + docker"
git push -u origin feature/nexus-server
```

### Riscos S2

| Risco | Mitigação |
|-------|-----------|
| `scripts/code-server.sh` não compila no fork | Investigar `src/vs/server/`, possível ajuste; alternativa: rodar com flags do test-web mas habilitar fs real |
| Docker build pesado (>2GB) | Otimizar multi-stage, copiar só o `out/` + node_modules de runtime |
| CORS/iframe quebrado | `frame-ancestors` no CSP; sandbox attr correto |
| Performance ruim multi-user | Spawnar instância por user (1 process por user); ou usar conexões compartilhadas |

---

## ⏳ Sprint 3 — Extensão Nexus AI (Ollama)

### Objetivo

Built-in extension que adiciona painel de chat lateral conectado a **Ollama** (local ou Cloud), com settings pra configurar URL/API key.

### Spec

- Extensão em `extensions/nexus-ai/`
- Painel lateral (Activity Bar) com ícone Nexus
- Settings:
  - `nexus.ollama.url` (default `http://localhost:11434`)
  - `nexus.ollama.apiKey` (pra Ollama Cloud)
  - `nexus.ollama.model` (default `qwen3:14b` ou similar)
- Comandos:
  - `Nexus: Ask` (Ctrl+Shift+L)
  - `Nexus: Edit Selection`
  - `Nexus: Explain`
- Integração com editor: chat sobre código selecionado, edição inline

### Riscos

- Ollama Cloud API pode ter rate limits
- Streaming pode quebrar com WebSockets em iframe

---

## ⏳ Sprint 4 — Integração OpenCode

### Objetivo

Substituir/integrar o agent loop da extensão Nexus AI com o **OpenCode** (`anomalyco/opencode` ou `sst/opencode`).

### Investigação prévia (TODO)

- Ler https://opencode.ai/docs
- Ver licença (precisa ser MIT/Apache pra integrar)
- Avaliar se o agent loop é extraível como lib ou se é monolítico TUI

### Spec inicial

- OpenCode roda como **backend headless** (sem TUI)
- Extensão Nexus AI consome a API
- File edits, tool use, multi-step orchestration via OpenCode
- Mantém Ollama como provider LLM

---

## ⏳ Sprint 5 — Embed & Deploy final

### Objetivo

- Deploy oficial em `console.nexusatemporal.com.br`
- Iframe no sistema Nexus production
- postMessage API pra comunicação parent ↔ child
- Login flow integrado (SSO Nexus)
- Monitoring + logs centralizados
- Documentação de admin

### Definition of Done

- Cliente Nexus loga no sistema → vê tab "IDE" → IDE abre embutido com workspace dele
- Performance OK (load < 5s, uso normal smooth)
- Multi-user funciona sem vazamento entre tenants
- Backup automático dos workspaces
