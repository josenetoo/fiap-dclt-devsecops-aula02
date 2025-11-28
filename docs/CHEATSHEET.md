# Aula 02 - Cheatsheet

## Gitleaks

```bash
# Instalar local
brew install gitleaks  # Mac
# ou docker

# Scan local
gitleaks detect --source .

# Scan com verbose
gitleaks detect -v --source .
```

## Pre-commit

```bash
# Instalar
pip install pre-commit

# Instalar hooks
pre-commit install

# Rodar em todos arquivos
pre-commit run --all-files

# Atualizar hooks
pre-commit autoupdate
```

## AWS Secrets Manager

```bash
# Criar secret
aws secretsmanager create-secret \
  --name devsecops/app/credentials \
  --secret-string '{"key":"value"}' \
  --profile fiapaws

# Ler secret
aws secretsmanager get-secret-value \
  --secret-id devsecops/app/credentials \
  --profile fiapaws

# Atualizar secret
aws secretsmanager update-secret \
  --secret-id devsecops/app/credentials \
  --secret-string '{"key":"new-value"}' \
  --profile fiapaws

# Deletar secret
aws secretsmanager delete-secret \
  --secret-id devsecops/app/credentials \
  --profile fiapaws
```

## Python - Secrets Manager

```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')
response = client.get_secret_value(SecretId='devsecops/app/credentials')
secrets = json.loads(response['SecretString'])
```
