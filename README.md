# Aula 02 - Gestão de Segredos

## 🎯 Objetivo

Implementar práticas de gestão segura de secrets usando Gitleaks, AWS Secrets Manager e Branch Protection.

## 📹 Vídeos desta Aula

| Vídeo | Tema | O que você vai fazer |
|-------|------|---------------------|
| 01 | Secret Scanning | Configurar Gitleaks no pipeline |
| 02 | Branch Protection | Configurar regras e pre-commit hooks |
| 03 | Secrets Manager | Integrar AWS Secrets Manager na aplicação |

## 📁 Estrutura do Repositório

```
.
├── app.py                 # Aplicação Flask
├── requirements.txt       # Dependências
├── Dockerfile
├── .gitleaks.toml        # Configuração do Gitleaks
├── .pre-commit-config.yaml # Hooks locais
├── .github/
│   └── workflows/        # (Criado durante a aula)
└── docs/
    ├── HANDS-ON-02-01.md  # Vídeo 01
    ├── HANDS-ON-02-02.md  # Vídeo 02
    ├── HANDS-ON-02-03.md  # Vídeo 03
    └── CHEATSHEET.md
```

## ⚙️ Pré-requisitos

- [ ] Aula 01 concluída
- [ ] Pipeline CI/CD funcionando
- [ ] Aplicação rodando no ECS

## 🚀 Como Usar

1. **Fork** este repositório
2. Configure os secrets do AWS
3. Siga os arquivos HANDS-ON de cada vídeo

## 📚 Documentação

| Vídeo | Hands-on |
|-------|----------|
| 01 - Secret Scanning | [HANDS-ON-02-01.md](docs/HANDS-ON-02-01.md) |
| 02 - Branch Protection | [HANDS-ON-02-02.md](docs/HANDS-ON-02-02.md) |
| 03 - Secrets Manager | [HANDS-ON-02-03.md](docs/HANDS-ON-02-03.md) |

**Referência rápida**: [Cheatsheet](docs/CHEATSHEET.md)

---

**FIAP - Pós Tech DevSecOps**
