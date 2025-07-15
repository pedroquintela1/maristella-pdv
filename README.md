# 🏪 Maristella PDV

> Sistema completo de controle de estoque e PDV (Ponto de Venda) com funcionamento online/offline

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![Next.js](https://img.shields.io/badge/Next.js-14+-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue?logo=typescript)](https://www.typescriptlang.org/)
[![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3.0+-38B2AC?logo=tailwind-css)](https://tailwindcss.com/)

## 📋 Sobre o Projeto

O **Maristella PDV** é um sistema moderno de controle de estoque e ponto de venda desenvolvido para funcionar tanto online quanto offline. Construído com as mais recentes tecnologias web, oferece uma experiência fluida e confiável para estabelecimentos comerciais.

### ✨ Principais Funcionalidades

- 🛒 **PDV Completo** - Interface otimizada para vendas rápidas
- 📦 **Controle de Estoque** - Gestão completa de produtos e movimentações
- 👥 **Gestão de Clientes** - CRM integrado com programa de fidelidade
- 📊 **Relatórios Avançados** - Analytics e dashboards em tempo real
- 🔄 **Sincronização Offline** - Funciona sem internet e sincroniza automaticamente
- 📱 **PWA** - Instalável como aplicativo nativo
- 🔐 **Multi-usuário** - Sistema de permissões granular

## 🛠️ Stack Tecnológica

### Frontend
- **Framework**: Next.js 14+ com App Router
- **Linguagem**: TypeScript
- **UI Framework**: ShadcnUI + TailwindCSS
- **Estado**: Zustand
- **Formulários**: React Hook Form + Zod
- **Testes**: Jest + React Testing Library
- **Documentação**: Storybook

### Backend & Database
- **API**: Next.js API Routes
- **Banco Online**: MySQL (PlanetScale/Railway)
- **Banco Offline**: SQLite via Prisma
- **ORM**: Prisma
- **Autenticação**: NextAuth.js
- **Cache**: Redis (opcional)

### DevOps & Deploy
- **Deploy**: Vercel
- **CI/CD**: GitHub Actions
- **Monitoramento**: Sentry + Vercel Analytics
- **Versionamento**: Git Flow

## 🏗️ Arquitetura

### Componentização Extrema
Seguimos a metodologia **Atomic Design** para máxima reutilização:

```
src/components/
├── atoms/      # Componentes básicos (Button, Input)
├── molecules/  # Combinações simples (SearchInput, ProductCard)
├── organisms/  # Componentes complexos (ProductList, Dashboard)
├── templates/  # Estruturas de layout
└── pages/      # Páginas completas
```

### Offline-First
- **Banco Local**: SQLite para operações offline
- **Sincronização**: Queue system com resolução de conflitos
- **Cache**: IndexedDB para assets e dados frequentes
- **PWA**: Service Worker com Workbox

## 📚 Documentação

- 📖 [**PRD Completo**](docs/PRD.md) - Requisitos e regras de negócio
- 🏗️ [**Arquitetura de Componentes**](docs/component-architecture.md) - Padrões de componentização
- 🗄️ [**Schema do Banco**](docs/database-diagram.md) - Estrutura e relacionamentos
- 🛣️ [**Roadmap de Desenvolvimento**](docs/development-roadmap.md) - Fases e cronograma
- 💻 [**Padrões de Código**](docs/coding-standards.md) - Convenções e boas práticas
- 🔧 [**Setup do GitHub**](docs/github-setup.md) - Configuração do repositório

## 🚀 Início Rápido

### Pré-requisitos
- Node.js 18+ 
- npm ou yarn
- Git

### Instalação

```bash
# Clonar o repositório
git clone https://github.com/pedroquintela1/maristella-pdv.git
cd maristella-pdv

# Instalar dependências
npm install

# Configurar banco de dados
npx prisma generate
npx prisma db push

# Executar em desenvolvimento
npm run dev
```

### Variáveis de Ambiente

```env
# Database
DATABASE_URL="mysql://user:password@localhost:3306/maristella"
DATABASE_URL_SQLITE="file:./dev.db"

# Auth
NEXTAUTH_SECRET="your-secret-here"
NEXTAUTH_URL="http://localhost:3000"

# Optional
REDIS_URL="redis://localhost:6379"
```

## 📋 Scripts Disponíveis

```bash
npm run dev          # Desenvolvimento
npm run build        # Build de produção
npm run start        # Executar produção
npm run lint         # Linter
npm run test         # Testes unitários
npm run test:e2e     # Testes E2E
npm run storybook    # Documentação de componentes
npm run db:migrate   # Migrations do banco
npm run db:seed      # Popular banco com dados iniciais
```

## 🧪 Testes

```bash
# Testes unitários
npm run test

# Testes com coverage
npm run test:coverage

# Testes E2E
npm run test:e2e

# Testes de componentes (Storybook)
npm run storybook
```

## 📦 Estrutura do Projeto

```
maristella-pdv/
├── src/
│   ├── app/                 # Next.js App Router
│   ├── components/          # Componentes React
│   │   ├── atoms/          # Componentes básicos
│   │   ├── molecules/      # Combinações simples
│   │   ├── organisms/      # Componentes complexos
│   │   └── ui/            # ShadcnUI components
│   ├── hooks/              # Custom hooks
│   ├── lib/                # Utilitários e configurações
│   ├── services/           # Serviços de API
│   ├── stores/             # Zustand stores
│   ├── types/              # Definições TypeScript
│   └── utils/              # Funções utilitárias
├── docs/                   # Documentação
├── prisma/                 # Schema e migrations
├── public/                 # Assets estáticos
└── tests/                  # Testes
```

## 🤝 Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'feat: add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

### Padrões de Commit
Seguimos [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` nova funcionalidade
- `fix:` correção de bug
- `docs:` documentação
- `style:` formatação
- `refactor:` refatoração
- `test:` testes
- `chore:` tarefas de build/config

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👨‍💻 Autor

**Pedro Quintela** - [@pedroquintela1](https://github.com/pedroquintela1)

## 🙏 Agradecimentos

- [Next.js](https://nextjs.org/) - Framework React
- [ShadcnUI](https://ui.shadcn.com/) - Componentes UI
- [TailwindCSS](https://tailwindcss.com/) - Framework CSS
- [Prisma](https://prisma.io/) - ORM TypeScript
- [Vercel](https://vercel.com/) - Plataforma de deploy

---

<div align="center">
  <strong>Desenvolvido com ❤️ para facilitar a gestão comercial</strong>
</div>
