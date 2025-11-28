# Vídeo 2.2 - Branch Protection e Pre-commit Hooks

**Aula**: 2 - Gestão de Segredos
**Vídeo**: 2.2
**Temas**: Defesa em profundidade; Branch Protection; Pre-commit hooks; Múltiplas camadas de segurança

---

## 💡 Conceitos Rápidos

| Termo | O que é |
|-------|---------|
| **Pre-commit Hook** | Script que roda **antes** do `git commit`. Se falhar, o commit é bloqueado. Funciona como um "porteiro" local. |
| **Branch Protection** | Regra do GitHub que protege branches importantes (ex: `main`). Exige PR, aprovações e checks passando. |

---

## 🚀 Antes de Começar

> **Continuação do Vídeo 2.1** - Certifique-se de ter o Gitleaks configurado.

### Pré-requisitos

| Requisito | Como verificar |
|-----------|----------------|
| Vídeo 2.1 completo | Gitleaks rodando no pipeline |
| Python instalado | `python --version` |
| pip instalado | `pip --version` |

---

## 📚 Parte 1: Defesa em Profundidade

### Passo 1: Por que Múltiplas Camadas?

```mermaid
graph TB
    subgraph "CAMADA 1: Local"
        A[Pre-commit Hook]
    end

    subgraph "CAMADA 2: Pipeline"
        B[Gitleaks Action]
    end

    subgraph "CAMADA 3: GitHub"
        C[Branch Protection]
    end

    subgraph "CAMADA 4: Runtime"
        D[Secrets Manager]
    end

    A --> B --> C --> D
```

**Princípio**: Se uma camada falhar, a próxima pega!

| Camada | Quando atua | O que faz |
|--------|-------------|-----------|
| **Pre-commit** | Antes do commit local | Bloqueia commit com secret |
| **Pipeline** | Após push | Escaneia código e histórico |
| **Branch Protection** | Antes do merge | Exige checks passando |
| **Secrets Manager** | Runtime | Armazena secrets de forma segura |

---

## 🛡️ Parte 2: Branch Protection

### Passo 2: O que é Branch Protection?

**Branch Protection** = Regras que protegem branches importantes

```mermaid
graph LR
    A[Developer] -->|push direto| B[main]
    B -->|❌ BLOQUEADO| C[Rejeita]

    A -->|cria branch| D[feature]
    D -->|PR| E[Review]
    E -->|checks passam| B
    B -->|✅ ACEITO| F[Merge]
```

**Benefícios:**
- Impede push direto para `main`
- Exige Pull Request com revisão
- Exige que checks (Gitleaks) passem
- Histórico de aprovações

---

### Passo 3: Configurar Branch Protection

**No GitHub:**

1. Acesse seu repositório
2. Vá em **Settings** > **Branches**
3. Em "Branch protection rules", clique **Add rule**
4. Configure:

| Campo | Valor |
|-------|-------|
| Branch name pattern | `main` |
| ✅ Require a pull request before merging | Marcar |
| ✅ Require status checks to pass before merging | Marcar |
| Status checks | Buscar e selecionar `🔍 Secret Scan` |
| ✅ Do not allow bypassing the above settings | Marcar |

> 💡 **Dica**: Busque por "Secret" no campo de pesquisa. O nome do status check é o `name` do job no workflow, não o ID.

5. Clique **Create**

**Resultado esperado:**
```
✅ Branch protection rule criada para 'main'
```

---

### Passo 4: Testar Branch Protection

**Tentar push direto (deve falhar):**

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets

# Criar arquivo de teste
echo "teste" > teste.txt
git add teste.txt
git commit -m "teste push direto"

# Tentar push direto para main
git push origin main
```

**Resultado esperado:**
```
remote: error: GH006: Protected branch update failed
remote: error: Required status check "gitleaks" is expected.
```

**Forma correta (via PR):**
```bash
# Criar branch
git checkout -b feature/teste

# Push para branch
git push origin feature/teste

# Criar PR no GitHub
# Aguardar checks passarem
# Fazer merge
```

---

## 🪝 Parte 3: Pre-commit Hooks

### Passo 5: O que são Pre-commit Hooks?

**Pre-commit** = Scripts que rodam ANTES do commit

```mermaid
graph LR
    A[git commit] --> B{Pre-commit hook}
    B -->|Passou| C[Commit criado]
    B -->|Falhou| D[Commit bloqueado]
```

**Vantagens:**
- Feedback instantâneo (não precisa esperar CI)
- Bloqueia antes de ir para o repositório
- Pode rodar múltiplas validações

---

### Passo 6: Instalar Pre-commit

**Mac (Homebrew - recomendado):**
```bash
brew install pre-commit

# Verificar instalação
pre-commit --version
# Esperado: pre-commit 3.x.x ou 4.x.x
```

**Linux:**
```bash
pip3 install pre-commit

# Verificar instalação
pre-commit --version
# Esperado: pre-commit 3.x.x
```

**Windows (PowerShell):**
```powershell
pip install pre-commit

# Verificar instalação
pre-commit --version
# Esperado: pre-commit 3.x.x
```

---

### Passo 7: Verificar Configuração

O repositório já tem o arquivo `.pre-commit-config.yaml`:

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets
cat .pre-commit-config.yaml
```

**Conteúdo esperado:**
```yaml
repos:
  # Gitleaks - Secret scanning
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  # Hooks padrão
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
```

---

### Passo 8: Ativar Pre-commit

**Linux/Mac:**
```bash
cd ~/projetos/aula02-secrets

# Instalar hooks no repositório
pre-commit install

# Resultado esperado:
# pre-commit installed at .git/hooks/pre-commit
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\aula02-secrets

# Instalar hooks no repositório
pre-commit install

# Resultado esperado:
# pre-commit installed at .git/hooks/pre-commit
```

---

### Passo 9: Testar Pre-commit (sem secret)

**Linux/Mac:**
```bash
# Criar arquivo normal
echo "# Arquivo de teste" > teste-normal.txt

# Tentar commit
git add teste-normal.txt
git commit -m "teste pre-commit"
```

**Resultado esperado:**
```
gitleaks..........................Passed
trailing-whitespace...............Passed
end-of-file-fixer.................Passed
check-yaml........................Passed
[main abc1234] teste pre-commit
```

---

### Passo 10: Testar Pre-commit (com secret)

> ⚠️ **Importante**: Valores com "EXAMPLE" no final são ignorados pelo Gitleaks. Use valores que pareçam reais.

**Linux/Mac:**
```bash
# Criar arquivo com secret falso
cat > teste-secret.txt << 'EOF'
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
EOF

# Tentar commit
git add teste-secret.txt
git commit -m "teste com secret"
```

**Resultado esperado:**
```
gitleaks..........................Failed
- hook id: gitleaks
- exit code: 1

Finding:     AWS_ACCESS_KEY_ID=AKIAIOSFODNN7REALKEY
Secret:      AKIAIOSFODNN7REALKEY
RuleID:      aws-access-key-id
```

**O commit foi BLOQUEADO!** 🎉

**Limpar:**
```bash
rm teste-secret.txt teste-normal.txt
git reset HEAD
```

---

### Passo 11: Rodar em Todo o Código

**Linux/Mac:**
```bash
# Rodar pre-commit em todos os arquivos
pre-commit run --all-files
```

**Windows (PowerShell):**
```powershell
# Rodar pre-commit em todos os arquivos
pre-commit run --all-files
```

**Resultado esperado:**
```
gitleaks..........................Passed
trailing-whitespace...............Passed
end-of-file-fixer.................Passed
check-yaml........................Passed
check-json........................Passed
```

---

## 🔧 Troubleshooting

| Erro | Causa | Solução |
|------|-------|---------|
| `pre-commit: command not found` | Não instalado | `pip install pre-commit` |
| Hook não executa | Não instalado no repo | `pre-commit install` |
| Push direto funciona | Branch protection não ativa | Verificar Settings > Branches |
| Gitleaks muito lento | Histórico grande | Usar `--no-git` flag |

---

**FIM DO VÍDEO 2.2** ✅
