# Configuração do Repositório GitHub - Maristella PDV

## Criação do Repositório

### 1. Criar Repositório no GitHub
1. Acesse [GitHub](https://github.com/new)
2. Configure o repositório:
   - **Repository name**: `maristella-pdv`
   - **Description**: `Sistema de controle de estoque e PDV com funcionamento online/offline - Next.js, ShadcnUI, TailwindCSS, MySQL/SQLite`
   - **Visibility**: Public (ou Private se preferir)
   - **Initialize with**: 
     - ✅ Add a README file
     - ✅ Add .gitignore (Node template)
     - ✅ Choose a license (MIT recommended)

### 2. Configurar Repositório Local
```bash
# Clonar o repositório
git clone https://github.com/pedroquintela1/maristella-pdv.git
cd maristella-pdv

# Adicionar arquivos de documentação existentes
cp -r ../maristella/docs ./
cp ../maristella/.augment-guidelines ./

# Commit inicial da documentação
git add .
git commit -m "docs: add initial project documentation and guidelines"
git push origin main
```

## Estrutura de Branches

### Branch Strategy (Git Flow Simplificado)
```
main (production)
├── develop (development)
├── feature/auth-system
├── feature/product-management
├── feature/pos-system
├── feature/inventory-control
├── feature/reports
├── feature/offline-sync
└── hotfix/critical-fixes
```

### Comandos para Configurar Branches
```bash
# Criar branch de desenvolvimento
git checkout -b develop
git push -u origin develop

# Configurar branch develop como padrão para PRs
# (fazer via interface do GitHub em Settings > Branches)

# Criar branches de feature (exemplo)
git checkout develop
git checkout -b feature/project-setup
git push -u origin feature/project-setup
```

## Configuração de Issues e Projects

### 1. Labels Recomendados
Criar os seguintes labels no repositório:

**Por Tipo:**
- `bug` - Correção de bugs
- `feature` - Nova funcionalidade
- `enhancement` - Melhoria de funcionalidade existente
- `documentation` - Documentação
- `refactor` - Refatoração de código
- `test` - Testes

**Por Prioridade:**
- `priority: high` - Alta prioridade
- `priority: medium` - Média prioridade
- `priority: low` - Baixa prioridade

**Por Módulo:**
- `module: auth` - Autenticação
- `module: products` - Gestão de produtos
- `module: inventory` - Controle de estoque
- `module: pos` - Ponto de venda
- `module: reports` - Relatórios
- `module: sync` - Sincronização offline

### 2. Issue Templates
Criar templates em `.github/ISSUE_TEMPLATE/`:

#### Bug Report Template
```markdown
---
name: Bug Report
about: Create a report to help us improve
title: '[BUG] '
labels: bug
assignees: ''
---

**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. Scroll down to '....'
4. See error

**Expected behavior**
A clear and concise description of what you expected to happen.

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Environment:**
- OS: [e.g. Windows, macOS, Linux]
- Browser: [e.g. Chrome, Firefox, Safari]
- Version: [e.g. 22]

**Additional context**
Add any other context about the problem here.
```

#### Feature Request Template
```markdown
---
name: Feature Request
about: Suggest an idea for this project
title: '[FEATURE] '
labels: feature
assignees: ''
---

**Is your feature request related to a problem? Please describe.**
A clear and concise description of what the problem is.

**Describe the solution you'd like**
A clear and concise description of what you want to happen.

**Describe alternatives you've considered**
A clear and concise description of any alternative solutions.

**Additional context**
Add any other context or screenshots about the feature request here.
```

### 3. GitHub Project Board
Criar um projeto com as seguintes colunas:
- **Backlog** - Issues planejados
- **Todo** - Próximas tarefas
- **In Progress** - Em desenvolvimento
- **Review** - Em revisão
- **Testing** - Em teste
- **Done** - Concluído

## Configuração de CI/CD

### 1. GitHub Actions Workflows
Criar `.github/workflows/ci.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run type check
      run: npm run type-check
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3

  build:
    runs-on: ubuntu-latest
    needs: test
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Use Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '20.x'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build application
      run: npm run build
    
    - name: Run E2E tests
      run: npm run test:e2e

  deploy:
    runs-on: ubuntu-latest
    needs: [test, build]
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        vercel-args: '--prod'
```

### 2. Dependabot Configuration
Criar `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "pedroquintela1"
    assignees:
      - "pedroquintela1"
```

## Configuração de Proteção de Branches

### Branch Protection Rules (main)
1. Ir em Settings > Branches
2. Adicionar regra para `main`:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (1)
   - ✅ Dismiss stale PR approvals when new commits are pushed
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Require conversation resolution before merging
   - ✅ Include administrators

### Branch Protection Rules (develop)
1. Adicionar regra para `develop`:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging

## Configuração de Secrets

### Secrets Necessários
Adicionar em Settings > Secrets and variables > Actions:

**Para Deploy:**
- `VERCEL_TOKEN` - Token do Vercel
- `ORG_ID` - ID da organização Vercel
- `PROJECT_ID` - ID do projeto Vercel

**Para Banco de Dados:**
- `DATABASE_URL` - URL do banco MySQL (produção)
- `DATABASE_URL_SQLITE` - Path do banco SQLite (desenvolvimento)

**Para Autenticação:**
- `NEXTAUTH_SECRET` - Secret para NextAuth.js
- `NEXTAUTH_URL` - URL da aplicação

## Comandos Úteis

### Configuração Inicial do Projeto
```bash
# Após clonar o repositório
npm init -y
npm install next@latest react@latest react-dom@latest typescript @types/node @types/react @types/react-dom

# Instalar dependências do projeto
npm install @shadcn/ui tailwindcss prisma @prisma/client zustand next-auth

# Configurar Prisma
npx prisma init
```

### Workflow de Desenvolvimento
```bash
# Criar nova feature
git checkout develop
git pull origin develop
git checkout -b feature/nome-da-feature

# Desenvolver e commitar
git add .
git commit -m "feat: add new feature"
git push -u origin feature/nome-da-feature

# Criar PR via GitHub interface
# Após aprovação e merge, limpar branch local
git checkout develop
git pull origin develop
git branch -d feature/nome-da-feature
```

## Próximos Passos

1. **Criar o repositório** seguindo as instruções acima
2. **Configurar as branches** e proteções
3. **Adicionar a documentação** existente
4. **Configurar CI/CD** com GitHub Actions
5. **Criar primeiro milestone** para setup do projeto
6. **Começar desenvolvimento** seguindo o roadmap em `docs/development-roadmap.md`
