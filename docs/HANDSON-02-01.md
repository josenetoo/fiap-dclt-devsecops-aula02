# 🎬 Vídeo 2.1 - Secret Scanning com Gitleaks

**Aula**: 2 - Gestão de Segredos  
**Vídeo**: 2.1  
**Temas**: Problema de secrets vazados; Gitleaks; Detecção automatizada no pipeline

---

## 🚀 Antes de Começar

### Pré-requisitos

| Requisito | Como verificar |
|-----------|----------------|
| Git instalado | `git --version` |
| Docker instalado | `docker --version` |
| Conta GitHub | Acesso ao github.com |

### Primeira vez?
→ Faça fork do repositório `aula02-secrets` para sua conta GitHub

---

## 📚 Parte 1: O Problema de Secrets Vazados

### Passo 1: Por que Secrets são Críticos?

```mermaid
graph TB
    A[Desenvolvedor] -->|commit com secret| B[Git Repository]
    B -->|push| C[GitHub Público]
    C -->|scan automático| D[🤖 Bots maliciosos]
    D -->|em segundos| E[💀 Exploração]
    
    E --> F[Crypto mining]
    E --> G[Data breach]
    E --> H[Conta AWS comprometida]
```

**Estatísticas reais:**
- Bots escaneiam GitHub em **tempo real**
- Tempo médio para explorar um secret vazado: **< 1 minuto**
- Custo médio de um vazamento: **$50.000+** em crypto mining

---

### Passo 2: O que é um Secret?

| Tipo | Exemplo | Risco |
|------|---------|-------|
| **API Keys** | `AKIAIOSFODNN7EXAMPLE` | Acesso a serviços AWS |
| **Senhas** | `password123` | Acesso a banco de dados |
| **Tokens** | `ghp_xxxxxxxxxxxx` | Acesso ao GitHub |
| **Chaves privadas** | `-----BEGIN RSA PRIVATE KEY-----` | Acesso SSH/SSL |
| **Connection strings** | `mongodb://user:pass@host` | Acesso a bancos |

> ⚠️ **Regra de ouro**: Se é sensível, NÃO vai no código!

---

### Passo 3: O que é Gitleaks?

**Gitleaks** = Ferramenta open source para detectar secrets em repositórios Git

**Como funciona:**
```mermaid
graph LR
    A[Código] --> B[Gitleaks]
    B --> C{Encontrou secret?}
    C -->|Sim| D[❌ Bloqueia]
    C -->|Não| E[✅ Continua]
```

**Características:**
- Analisa histórico completo do Git
- Detecta 100+ tipos de secrets (AWS, GitHub, Slack, etc.)
- Configurável via `.gitleaks.toml`
- Integração com CI/CD

---

## 🍴 Parte 2: Configurar o Repositório

### Passo 4: Fork do Repositório

1. Acesse: `https://github.com/fiap-devsecops/aula02-secrets`
2. Clique em **Fork**
3. Selecione sua conta

---

### Passo 5: Clonar seu Fork

**Linux/Mac:**
```bash
cd ~/projetos

# Clonar SEU fork
git clone https://github.com/SEU-USUARIO/aula02-secrets.git
cd aula02-secrets

# Verificar estrutura
ls -la
```

**Windows (PowerShell):**
```powershell
cd ~\projetos

# Clonar SEU fork
git clone https://github.com/SEU-USUARIO/aula02-secrets.git
cd aula02-secrets

# Verificar estrutura
Get-ChildItem
```

**Estrutura esperada:**
```
aula02-secrets/
├── app.py
├── requirements.txt
├── Dockerfile
├── .gitleaks.toml        ← Configuração do Gitleaks
├── .pre-commit-config.yaml
└── docs/
```

---

## 🔍 Parte 3: Testar Gitleaks Localmente

### Passo 6: Instalar Gitleaks

**Linux/Mac:**
```bash
# Via Homebrew (Mac)
brew install gitleaks

# Via script (Linux)
curl -sSfL https://github.com/gitleaks/gitleaks/releases/download/v8.18.0/gitleaks_8.18.0_linux_x64.tar.gz | tar -xz
sudo mv gitleaks /usr/local/bin/
```

**Windows (PowerShell):**
```powershell
# Via Chocolatey
choco install gitleaks

# Ou baixe de: https://github.com/gitleaks/gitleaks/releases
```

**Verificar instalação:**
```bash
gitleaks version
# Esperado: v8.18.x
```

---

### Passo 7: Executar Scan Local

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets

# Scan do diretório atual
gitleaks detect --source . -v

# Scan do histórico Git
gitleaks detect --source . --log-opts="--all" -v
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\aula02-secrets

# Scan do diretório atual
gitleaks detect --source . -v

# Scan do histórico Git
gitleaks detect --source . --log-opts="--all" -v
```

**Resultado esperado (sem secrets):**
```
○ No leaks found
```

---

### Passo 8: Testar Detecção (Criar secret falso)

Vamos criar um arquivo com secret FALSO para testar:

> ⚠️ **Importante**: Valores com "EXAMPLE" no final são ignorados pelo Gitleaks (allowlist interna). Use valores que pareçam reais.

**Linux/Mac:**
```bash
# Criar arquivo de teste com secret falso
cat > test-secret.txt << 'EOF'
# Este é um arquivo de TESTE
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYzzzzzzzz
EOF

# Rodar Gitleaks (--no-git para arquivos não commitados)
gitleaks detect --source . --no-git -v
```

**Windows (PowerShell):**
```powershell
# Criar arquivo de teste com secret falso
@"
# Este é um arquivo de TESTE
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYzzzzzzzz
"@ | Out-File -FilePath test-secret.txt -Encoding UTF8

# Rodar Gitleaks (--no-git para arquivos não commitados)
gitleaks detect --source . --no-git -v
```

**Resultado esperado (com secrets):**
```
Finding:     AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
Secret:      AKIAIOSFODNN7REALKEY
RuleID:      aws-access-key-id
File:        test-secret.txt
Line:        2

○ 2 leaks found
```

**Limpar arquivo de teste:**
```bash
rm test-secret.txt
```

---

## 🔄 Parte 4: Adicionar Gitleaks ao Pipeline

### Passo 9: Criar Workflow de Security

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets

# Criar diretório de workflows
mkdir -p .github/workflows

# Criar workflow de segurança
cat > .github/workflows/security.yml << 'EOF'
# ============================================
# WORKFLOW: Security Scanning
# Detecta secrets e vulnerabilidades
# ============================================
name: 🔒 Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # ============================================
  # JOB: Secret Scanning com Gitleaks
  # ============================================
  gitleaks:
    name: 🔍 Secret Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Necessário para escanear histórico
      
      - name: 🔍 Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\aula02-secrets

# Criar diretório de workflows
New-Item -ItemType Directory -Force -Path .github/workflows

# Criar workflow de segurança
@'
# ============================================
# WORKFLOW: Security Scanning
# Detecta secrets e vulnerabilidades
# ============================================
name: 🔒 Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # ============================================
  # JOB: Secret Scanning com Gitleaks
  # ============================================
  gitleaks:
    name: 🔍 Secret Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: 🔍 Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
'@ | Out-File -FilePath .github/workflows/security.yml -Encoding UTF8
```

---

### Passo 10: Entender o Workflow

```mermaid
graph LR
    A[Push/PR] --> B[Checkout com histórico]
    B --> C[Gitleaks Action]
    C --> D{Encontrou secret?}
    D -->|Sim| E[❌ Pipeline falha]
    D -->|Não| F[✅ Pipeline passa]
```

**Pontos importantes:**
- `fetch-depth: 0` → Baixa histórico completo para análise
- `GITHUB_TOKEN` → Token **automático** do GitHub (não precisa criar!)

> 💡 **Sobre o GITHUB_TOKEN**: O GitHub gera automaticamente este token para cada execução de workflow. Ele tem permissões para acessar o repositório. Você **NÃO precisa criar** este secret manualmente.

---

### Passo 11: Commit e Push

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets

# Adicionar workflow
git add .github/workflows/security.yml

# Commit
git commit -m "feat: adicionar Gitleaks ao pipeline"

# Push
git push origin main
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\aula02-secrets

# Adicionar workflow
git add .github/workflows/security.yml

# Commit
git commit -m "feat: adicionar Gitleaks ao pipeline"

# Push
git push origin main
```

---

### Passo 12: Verificar Execução

1. No GitHub, vá para **Actions**
2. Clique no workflow **Security Scan**
3. Verifique que passou

**Resultado esperado:**
```
✅ Secret Scan - Passed
```

> ℹ️ **Por que passou?** O repositório não tem secrets commitados. O pipeline está configurado para **bloquear futuros vazamentos**.

---

### Passo 13 (Opcional): Testar Pipeline com Secret

Para ver o Gitleaks **falhando** no pipeline:

```bash
# Criar arquivo com secret
cat > test-secret.txt << 'EOF'
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
EOF

# Commitar e fazer push
git add test-secret.txt
git commit -m "teste: adicionar secret para demo"
git push origin main
```

**No GitHub Actions, você verá:**
```
❌ Secret Scan - Failed

Finding: AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
Secret:  AKIAIOSFODNN7REALKEY
RuleID:  aws-access-key-id
```

**Limpar (IMPORTANTE!):**
```bash
# Remover o secret do histórico
git rm test-secret.txt
git commit -m "fix: remover secret de teste"
git push origin main
```

> ⚠️ **Atenção**: Em um cenário real, se um secret vazar, você deve **rotacionar a credencial** imediatamente, pois ela permanece no histórico do Git!

---

## 🔧 Troubleshooting

| Erro | Causa | Solução |
|------|-------|---------|
| `gitleaks: command not found` | Não instalado | Instalar via brew/choco |
| Pipeline falha com "leak found" | Secret real detectado | Remover secret e rotacionar credencial |
| `fetch-depth` erro | Histórico não disponível | Adicionar `fetch-depth: 0` |

---

## ✅ Checkpoint

Ao final deste vídeo você deve ter:

- [ ] Entender o risco de secrets vazados
- [ ] Gitleaks instalado localmente
- [ ] Testado detecção de secret falso
- [ ] Workflow de security no pipeline
- [ ] Pipeline passando

---

**FIM DO VÍDEO 2.1** ✅
