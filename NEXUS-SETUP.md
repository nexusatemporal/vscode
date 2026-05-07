# Nexus Console — Setup na VPS Linux

Como instalar e rodar o Nexus Console em uma VPS Linux (Debian/Ubuntu).

## Pré-requisitos do sistema

```bash
# Ferramentas de build (sem essas, native modules quebram)
sudo apt update
sudo apt install -y build-essential python3 python3-pip git curl libsecret-1-dev libx11-dev libxkbfile-dev fakeroot rpm libkrb5-dev

# Git LFS (alguns assets binários usam)
sudo apt install -y git-lfs
git lfs install
```

## Node.js — versão exata

⚠️ **TEM que ser Node 22.22.1+** (o fork checa no `build/npm/preinstall.ts`). Versões diferentes saem com erro explícito.

```bash
# Instalar nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc

# Instalar Node 22.22.1
nvm install 22.22.1
nvm use 22.22.1
nvm alias default 22.22.1

# Verificar
node --version    # v22.22.1
npm --version     # 10.9+
```

## Clone e install

```bash
# Em /root ou /opt, escolha o local
cd /opt
git clone https://github.com/nexusatemporal/vscode.git nexus-console
cd nexus-console

# Install — vai demorar 5-15 min na primeira vez
# Linux geralmente é mais tranquilo que Windows (sem Spectre lib drama)
npm install --no-audit --no-fund

# Verificar tamanho
du -sh node_modules     # esperado ~1.4 GB
ls node_modules | wc -l # esperado ~944+
```

### Se `npm install` falhar:
- Erro `gyp ERR! find Python` → `sudo apt install python3` e `npm config set python python3`
- Erro de `keytar` ou `libsecret` → `sudo apt install libsecret-1-dev`
- Erro de permissão → não rode com `sudo`, use o usuário normal

## Compilar

```bash
# Compilação completa do TypeScript (~5 min)
npm run compile

# Ao final espera ver:
# [HH:MM:SS] Finished 'compile' after X.XX min
# 0 errors

# Verificar
ls out/                 # 6977+ arquivos
du -sh out/             # ~170 MB
```

### Se quiser rodar a versão WEB também:

```bash
# Compila as extensões pra browser (resolve os 3 warnings de extensão)
npm run gulp compile-web-extensions-build

# Esbuild pro web (otimizado/minified)
npm run gulp esbuild-vscode-web-min
```

## Modos de execução

### Modo 1 — Web dev (test-web)

Pra desenvolvimento, igual a S1. **Não use em produção.**

```bash
./scripts/code-web.sh --port 8088 --host 0.0.0.0
# Acesse: http://VPS_IP:8088/
```

### Modo 2 — Server real (S2)

Esse é o pra produção. Tem auth, fs real, terminal:

```bash
./scripts/code-server.sh \
  --port 8088 \
  --host 0.0.0.0 \
  --connection-token nexus-secret-token \
  --user-data-dir /data/nexus-console/user-data \
  /data/nexus-console/workspaces

# Acesse: http://VPS_IP:8088/?tkn=nexus-secret-token
```

⚠️ Ver [`NEXUS-SPRINTS.md`](./NEXUS-SPRINTS.md) Sprint 2 pra config completa (multi-user, HTTPS via Traefik, etc).

## Firewall / portas

```bash
# Liberar porta 8088 (ou trocar pra outra)
sudo ufw allow 8088/tcp

# Recomendado: NÃO expor 8088 direto pra internet
# Use Traefik/Nginx fazendo reverse proxy com HTTPS na porta 443
```

## Branding overrides (web)

O arquivo `product.overrides.json` na raiz já tá no repo e versionado. Quando você rodar `code-web.sh` ou `code-server.sh`, ele é lido automaticamente e aplica:

- nameLong: "Nexus Console"
- urlProtocol: "nexus"
- Tema default: Nexus Dark (tem limitação no test-web — em prod com server real funciona)

## Comandos úteis

```bash
# Rebuild parcial (depois de mudar TypeScript)
npm run watch                # watch mode, compila on save

# Limpar e recompilar
rm -rf out/
npm run compile

# Rodar testes
./scripts/test.sh

# Fazer pacote final pro Linux (S2.5)
npm run gulp vscode-linux-x64-min
```

## Recursos esperados

| Item | Mínimo | Recomendado |
|------|--------|-------------|
| RAM (build) | 4 GB | 8 GB+ |
| RAM (runtime, 1 user) | 1 GB | 2 GB |
| RAM (runtime, multi-user) | 2-4 GB | 8 GB+ |
| Disco | 5 GB livres | 20 GB |
| CPU | 2 cores | 4+ cores |

## Troubleshooting

| Sintoma | Causa | Solução |
|---------|-------|---------|
| `Please use Node.js v22.22.1` | Versão errada | `nvm use 22.22.1` |
| `gyp ERR! find Python` | Sem Python | `sudo apt install python3` |
| `Error: libsecret` | Falta lib Linux | `sudo apt install libsecret-1-dev` |
| `EACCES` em `node_modules/` | Rodou install com sudo antes | `sudo chown -R $USER node_modules` e reinstall |
| Build verde mas web não abre | Porta bloqueada | `sudo ufw allow 8088/tcp` |
| Página em branco no browser | Erro JS | Abre DevTools, ver Console |
