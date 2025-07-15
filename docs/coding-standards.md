# Padrões de Código - Maristella PDV

## Estrutura de Arquivos e Nomenclatura

### Convenções de Nomenclatura
```typescript
// ✅ CORRETO
// Componentes: PascalCase
const ProductCard = () => {}
const SearchInput = () => {}

// Arquivos: kebab-case
product-card.tsx
search-input.tsx
use-products.ts

// Hooks: camelCase com 'use'
const useProducts = () => {}
const useLocalStorage = () => {}

// Constantes: SCREAMING_SNAKE_CASE
const API_ENDPOINTS = {}
const DEFAULT_CONFIG = {}

// Types/Interfaces: PascalCase
interface Product {}
type ApiResponse<T> = {}

// ❌ ERRADO
const productCard = () => {} // deve ser PascalCase
const UseProducts = () => {} // hooks não são componentes
const api_endpoints = {} // deve ser SCREAMING_SNAKE_CASE
```

### Estrutura de Pastas
```
src/
├── app/                    # Next.js 13+ App Router
│   ├── (auth)/            # Route groups
│   ├── api/               # API routes
│   └── globals.css        # Global styles
├── components/
│   ├── atoms/             # Componentes básicos
│   │   ├── button/
│   │   │   ├── button.tsx
│   │   │   ├── button.stories.tsx
│   │   │   ├── button.test.tsx
│   │   │   └── index.ts
│   │   └── input/
│   ├── molecules/         # Combinação de atoms
│   ├── organisms/         # Componentes complexos
│   ├── templates/         # Layouts
│   └── ui/               # ShadcnUI components
├── hooks/                # Custom hooks
├── lib/                  # Utilities e configurações
├── services/             # API services
├── stores/               # Zustand stores
├── types/                # TypeScript definitions
├── utils/                # Funções utilitárias
└── constants/            # Constantes da aplicação
```

## Padrões de Componentes

### Template de Componente Atom
```typescript
// components/atoms/button/button.tsx
import React, { forwardRef } from 'react'
import { Slot } from '@radix-ui/react-slot'
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
  loading?: boolean
}

/**
 * Button component with multiple variants and sizes
 * 
 * @param variant - Visual style variant
 * @param size - Size of the button
 * @param loading - Shows loading spinner when true
 * @param asChild - Render as child component
 * @param className - Additional CSS classes
 * @param children - Button content
 */
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, loading, children, disabled, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button'
    
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        disabled={disabled || loading}
        {...props}
      >
        {loading ? (
          <>
            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
            {children}
          </>
        ) : (
          children
        )}
      </Comp>
    )
  }
)

Button.displayName = 'Button'

export { Button, buttonVariants }
```

### Template de Componente Molecule
```typescript
// components/molecules/search-input/search-input.tsx
import React, { useState, useCallback } from 'react'
import { Search, X } from 'lucide-react'
import { Input } from '@/components/atoms/input'
import { Button } from '@/components/atoms/button'
import { cn } from '@/lib/utils'
import { useDebounce } from '@/hooks/use-debounce'

interface SearchInputProps {
  placeholder?: string
  value?: string
  onSearch: (value: string) => void
  onClear?: () => void
  loading?: boolean
  debounceMs?: number
  className?: string
}

/**
 * Search input with debounced search and clear functionality
 */
const SearchInput: React.FC<SearchInputProps> = ({
  placeholder = 'Buscar...',
  value: controlledValue,
  onSearch,
  onClear,
  loading = false,
  debounceMs = 300,
  className
}) => {
  const [internalValue, setInternalValue] = useState(controlledValue || '')
  const value = controlledValue !== undefined ? controlledValue : internalValue
  
  const debouncedSearch = useDebounce((searchValue: string) => {
    onSearch(searchValue)
  }, debounceMs)

  const handleChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value
    if (controlledValue === undefined) {
      setInternalValue(newValue)
    }
    debouncedSearch(newValue)
  }, [controlledValue, debouncedSearch])

  const handleClear = useCallback(() => {
    if (controlledValue === undefined) {
      setInternalValue('')
    }
    onSearch('')
    onClear?.()
  }, [controlledValue, onSearch, onClear])

  return (
    <div className={cn('relative', className)}>
      <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
      <Input
        value={value}
        onChange={handleChange}
        placeholder={placeholder}
        className="pl-10 pr-10"
        disabled={loading}
      />
      {value && (
        <Button
          variant="ghost"
          size="icon"
          onClick={handleClear}
          className="absolute right-1 top-1/2 h-8 w-8 -translate-y-1/2"
          disabled={loading}
        >
          <X className="h-4 w-4" />
        </Button>
      )}
    </div>
  )
}

export { SearchInput }
export type { SearchInputProps }
```

### Template de Custom Hook
```typescript
// hooks/use-products.ts
import { useState, useEffect, useCallback } from 'react'
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toast } from 'sonner'
import { productsService } from '@/services/products'
import type { Product, CreateProductData, UpdateProductData } from '@/types/product'

interface UseProductsOptions {
  enabled?: boolean
  search?: string
  category?: string
}

interface UseProductsReturn {
  products: Product[]
  loading: boolean
  error: string | null
  createProduct: (data: CreateProductData) => Promise<void>
  updateProduct: (id: string, data: UpdateProductData) => Promise<void>
  deleteProduct: (id: string) => Promise<void>
  refetch: () => void
}

/**
 * Hook for managing products data and operations
 */
export const useProducts = (options: UseProductsOptions = {}): UseProductsReturn => {
  const { enabled = true, search, category } = options
  const queryClient = useQueryClient()

  const {
    data: products = [],
    isLoading: loading,
    error,
    refetch
  } = useQuery({
    queryKey: ['products', { search, category }],
    queryFn: () => productsService.getAll({ search, category }),
    enabled,
    staleTime: 5 * 60 * 1000, // 5 minutes
  })

  const createMutation = useMutation({
    mutationFn: productsService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
      toast.success('Produto criado com sucesso!')
    },
    onError: (error) => {
      toast.error(`Erro ao criar produto: ${error.message}`)
    }
  })

  const updateMutation = useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateProductData }) =>
      productsService.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
      toast.success('Produto atualizado com sucesso!')
    },
    onError: (error) => {
      toast.error(`Erro ao atualizar produto: ${error.message}`)
    }
  })

  const deleteMutation = useMutation({
    mutationFn: productsService.delete,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
      toast.success('Produto excluído com sucesso!')
    },
    onError: (error) => {
      toast.error(`Erro ao excluir produto: ${error.message}`)
    }
  })

  const createProduct = useCallback(async (data: CreateProductData) => {
    await createMutation.mutateAsync(data)
  }, [createMutation])

  const updateProduct = useCallback(async (id: string, data: UpdateProductData) => {
    await updateMutation.mutateAsync({ id, data })
  }, [updateMutation])

  const deleteProduct = useCallback(async (id: string) => {
    await deleteMutation.mutateAsync(id)
  }, [deleteMutation])

  return {
    products,
    loading: loading || createMutation.isPending || updateMutation.isPending || deleteMutation.isPending,
    error: error?.message || null,
    createProduct,
    updateProduct,
    deleteProduct,
    refetch
  }
}
```

## Padrões de Configuração

### Configuração de Constantes
```typescript
// constants/app-config.ts
export const APP_CONFIG = {
  api: {
    baseUrl: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000/api',
    timeout: 10000,
    retries: 3
  },
  ui: {
    itemsPerPage: 20,
    debounceDelay: 300,
    animationDuration: 200,
    toastDuration: 4000
  },
  business: {
    maxDiscountPercentage: 50,
    lowStockThreshold: 10,
    currencySymbol: 'R$',
    currencyLocale: 'pt-BR'
  },
  storage: {
    tokenKey: 'maristella_auth_token',
    userKey: 'maristella_user_data',
    cartKey: 'maristella_cart_data'
  }
} as const

export type AppConfig = typeof APP_CONFIG
```

### Configuração de Tema
```typescript
// lib/theme.ts
export const theme = {
  colors: {
    primary: {
      50: '#eff6ff',
      100: '#dbeafe',
      500: '#3b82f6',
      600: '#2563eb',
      900: '#1e3a8a'
    },
    success: {
      50: '#f0fdf4',
      500: '#22c55e',
      600: '#16a34a'
    },
    warning: {
      50: '#fffbeb',
      500: '#f59e0b',
      600: '#d97706'
    },
    error: {
      50: '#fef2f2',
      500: '#ef4444',
      600: '#dc2626'
    }
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem'
  },
  borderRadius: {
    sm: '0.25rem',
    md: '0.375rem',
    lg: '0.5rem',
    xl: '0.75rem'
  }
} as const
```

## Padrões de Testes

### Teste de Componente
```typescript
// components/atoms/button/button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './button'

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument()
  })

  it('applies correct variant classes', () => {
    render(<Button variant="destructive">Delete</Button>)
    const button = screen.getByRole('button')
    expect(button).toHaveClass('bg-destructive')
  })

  it('handles click events', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click me</Button>)
    
    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('shows loading state', () => {
    render(<Button loading>Loading</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument()
  })
})
```

### Teste de Hook
```typescript
// hooks/use-products.test.ts
import { renderHook, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { useProducts } from './use-products'
import { productsService } from '@/services/products'

jest.mock('@/services/products')

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } }
  })
  
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

describe('useProducts', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  it('fetches products successfully', async () => {
    const mockProducts = [{ id: '1', name: 'Product 1' }]
    ;(productsService.getAll as jest.Mock).mockResolvedValue(mockProducts)

    const { result } = renderHook(() => useProducts(), {
      wrapper: createWrapper()
    })

    await waitFor(() => {
      expect(result.current.loading).toBe(false)
    })

    expect(result.current.products).toEqual(mockProducts)
    expect(result.current.error).toBeNull()
  })
})
```

## Padrões de Commit

### Conventional Commits
```bash
# Formato: <type>(<scope>): <description>

# Tipos principais:
feat: nova funcionalidade
fix: correção de bug
docs: documentação
style: formatação, ponto e vírgula, etc
refactor: refatoração de código
test: adição ou correção de testes
chore: tarefas de build, configuração, etc

# Exemplos:
feat(auth): add login functionality
fix(products): resolve stock calculation bug
docs(readme): update installation instructions
style(button): improve hover animations
refactor(hooks): extract common logic to useEntity
test(products): add unit tests for product service
chore(deps): update dependencies to latest versions
```

## Checklist de Qualidade

### Para cada Componente:
- [ ] Segue padrões de nomenclatura
- [ ] Props tipadas com TypeScript
- [ ] JSDoc documentation
- [ ] Testes unitários
- [ ] Storybook story (se aplicável)
- [ ] Acessibilidade (ARIA labels, keyboard navigation)
- [ ] Performance (memo, useMemo, useCallback quando necessário)
- [ ] Error boundaries (para componentes críticos)

### Para cada Hook:
- [ ] Nome descritivo começando com 'use'
- [ ] Tipagem completa de parâmetros e retorno
- [ ] Testes unitários
- [ ] Documentação de uso
- [ ] Cleanup de efeitos (useEffect)
- [ ] Memoização adequada

### Para cada Service:
- [ ] Tratamento de erros
- [ ] Tipagem de requests/responses
- [ ] Retry logic quando apropriado
- [ ] Logging adequado
- [ ] Testes de integração
