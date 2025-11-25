# Workflows Reutilizáveis de CI/CD

Este repositório centraliza os workflows do GitHub Actions da organização, permitindo que sejam reutilizados em diferentes projetos sem duplicação de código.

## 📋 Workflows Disponíveis

### 1. E2E Tests (`e2e-test.yml`)
Executa testes end-to-end usando Cypress com as seguintes características:
-  Build e inicialização automática da aplicação Next.js
-  Cache do binário do Cypress para execução mais rápida
-  Upload de screenshots (em caso de falha) e vídeos
-  Controle de concorrência para cancelar execuções antigas
-  Timeout de 15 minutos

### 2. Lighthouse CI (`lighthouse-ci.yml`)
Executa análise de performance com Lighthouse CI:
-  Métricas de performance, acessibilidade, SEO e best practices
-  Criação automática de issues quando os limites não são atingidos
-  Upload de relatórios detalhados como artefatos
-  Verificação de assertions customizadas
-  Retenção de artefatos por 14 dias

##  Como Usar nos Seus Projetos

### Integrar E2E Tests

```yaml
name: E2E Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  e2e:
    uses: hastydev-software/.github/.github/workflows/e2e-test.yml@main
    secrets: inherit
```

### Integrar Lighthouse CI

```yaml
name: Lighthouse CI

on:
  push:
    branches: [ main, master ]

permissions:
  contents: read
  issues: write

jobs:
  lighthouse:
    uses: hastydev-software/.github/.github/workflows/lighthouse-ci.yml@main
    secrets: inherit
```

## Secrets Necessários

Os seguintes secrets devem ser configurados no repositório que utiliza os workflows:

### Para E2E Tests:
- `API_KEY` - Chave de API do backend
- `BACKEND_URL` - URL do backend
- `NEXT_PUBLIC_LOGMANAGER_URL` - URL do gerenciador de logs
- `TOKEN_SIGNATURE_SECRET` - Secret para assinatura de tokens
- `TEST_EMAIL` - Email para autenticação nos testes
- `TEST_PASSWORD` - Senha para autenticação nos testes

### Para Lighthouse CI:
- Mesmos secrets do E2E Tests
- `GITHUB_TOKEN` - Fornecido automaticamente pelo GitHub Actions

##  Requisitos do Projeto

Para que os workflows funcionem corretamente, seu projeto deve ter:

### Para E2E Tests:
- `package.json` com scripts:
  - `build` - Para build da aplicação
  - `start` - Para iniciar o servidor de produção
- Configuração do Cypress em `cypress/` ou arquivo de config
- Node.js 20+

### Para Lighthouse CI:
- Arquivo `.lighthouserc.json` na raiz do projeto
- Mesmos requisitos do E2E Tests

## Exemplo de Configuração do Lighthouse

Crie um arquivo `.lighthouserc.json` na raiz do seu projeto:

```json
{
  "ci": {
    "collect": {
      "url": [
        "http://localhost:3000"
      ],
      "numberOfRuns": 3
    },
    "assert": {
      "preset": "lighthouse:recommended",
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 0.9}],
        "categories:best-practices": ["error", {"minScore": 0.9}],
        "categories:seo": ["error", {"minScore": 0.9}]
      }
    },
    "upload": {
      "target": "filesystem",
      "outputDir": "./.lighthouseci"
    }
  }
}
```

## Runners

Os workflows estão configurados para executar em:
```yaml
runs-on: [self-hosted, linux]
```

## Notas Importantes

- Os workflows cancelam automaticamente execuções em andamento quando há um novo push
- Artefatos do Cypress são mantidos por 3 dias
- Artefatos do Lighthouse são mantidos por 14 dias
- Issues são criadas automaticamente quando métricas do Lighthouse falham
- Ambos workflows usam cache para otimizar o tempo de execução