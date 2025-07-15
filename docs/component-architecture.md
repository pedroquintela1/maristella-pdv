# Arquitetura de Componentização Extrema - Maristella PDV

## Princípios Fundamentais

### 1. Single Responsibility Principle (SRP)
- Cada componente deve ter **uma única responsabilidade**
- Componentes devem ser **altamente coesos** e **fracamente acoplados**
- Separar lógica de apresentação, estado e efeitos colaterais

### 2. Composition over Inheritance
- Preferir **composição** ao invés de herança
- Usar **Higher-Order Components (HOC)** e **Render Props** quando necessário
- Implementar **Compound Components** para componentes complexos

### 3. Atomic Design Methodology
```
Atoms → Molecules → Organisms → Templates → Pages
```

## Estrutura de Componentes

### Hierarquia de Componentes
```
src/
├── components/
│   ├── atoms/           # Componentes básicos indivisíveis
│   ├── molecules/       # Combinação de atoms
│   ├── organisms/       # Combinação de molecules
│   ├── templates/       # Layout structures
│   └── pages/          # Páginas completas
├── hooks/              # Custom hooks reutilizáveis
├── utils/              # Funções utilitárias puras
├── types/              # TypeScript definitions
├── constants/          # Constantes da aplicação
└── services/           # Serviços e APIs
```

## Regras de Componentização

### ATOMS (Componentes Básicos)
**Características:**
- Não podem ser quebrados em componentes menores
- Não possuem estado próprio (stateless)
- Recebem apenas props
- Altamente reutilizáveis

**Exemplos:**
```typescript
// ❌ ERRADO - Muito específico
const LoginButton = () => <button>Login</button>

// ✅ CORRETO - Genérico e reutilizável
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger'
  size: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  onClick?: () => void
  disabled?: boolean
  loading?: boolean
}

const Button: React.FC<ButtonProps> = ({ 
  variant, size, children, onClick, disabled, loading 
}) => {
  return (
    <button
      className={cn(
        'font-medium rounded-md transition-colors',
        variants[variant],
        sizes[size],
        disabled && 'opacity-50 cursor-not-allowed',
        loading && 'cursor-wait'
      )}
      onClick={onClick}
      disabled={disabled || loading}
    >
      {loading ? <Spinner size={size} /> : children}
    </button>
  )
}
```

### MOLECULES (Combinação de Atoms)
**Características:**
- Combinam 2+ atoms
- Podem ter estado local simples
- Focam em uma funcionalidade específica

**Exemplo:**
```typescript
// SearchInput molecule
interface SearchInputProps {
  placeholder?: string
  onSearch: (value: string) => void
  loading?: boolean
}

const SearchInput: React.FC<SearchInputProps> = ({ 
  placeholder, onSearch, loading 
}) => {
  const [value, setValue] = useState('')
  
  return (
    <div className="relative">
      <Input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder={placeholder}
        onKeyPress={(e) => e.key === 'Enter' && onSearch(value)}
      />
      <Button
        variant="ghost"
        size="sm"
        onClick={() => onSearch(value)}
        loading={loading}
        className="absolute right-2 top-1/2 -translate-y-1/2"
      >
        <SearchIcon />
      </Button>
    </div>
  )
}
```

### ORGANISMS (Componentes Complexos)
**Características:**
- Combinam molecules e atoms
- Podem ter estado complexo
- Representam seções distintas da interface

**Exemplo:**
```typescript
// ProductCard organism
interface ProductCardProps {
  product: Product
  onAddToCart: (product: Product) => void
  onEdit?: (product: Product) => void
  showActions?: boolean
}

const ProductCard: React.FC<ProductCardProps> = ({ 
  product, onAddToCart, onEdit, showActions = true 
}) => {
  return (
    <Card className="p-4">
      <ProductImage src={product.image} alt={product.name} />
      <ProductInfo 
        name={product.name}
        price={product.price}
        stock={product.stock}
      />
      {showActions && (
        <ProductActions
          onAddToCart={() => onAddToCart(product)}
          onEdit={onEdit ? () => onEdit(product) : undefined}
          disabled={product.stock === 0}
        />
      )}
    </Card>
  )
}
```

## Padrões de Reutilização

### 1. Configuration-Driven Components
```typescript
// Configuração centralizada
const FORM_CONFIGS = {
  product: {
    fields: [
      { name: 'name', type: 'text', required: true },
      { name: 'price', type: 'number', required: true },
      { name: 'category', type: 'select', options: 'categories' }
    ]
  },
  customer: {
    fields: [
      { name: 'name', type: 'text', required: true },
      { name: 'email', type: 'email', required: false },
      { name: 'phone', type: 'tel', required: true }
    ]
  }
}

// Componente genérico
const DynamicForm: React.FC<{
  config: FormConfig
  onSubmit: (data: any) => void
}> = ({ config, onSubmit }) => {
  // Renderiza formulário baseado na configuração
}
```

### 2. Render Props Pattern
```typescript
interface DataFetcherProps<T> {
  url: string
  children: (data: T | null, loading: boolean, error: string | null) => React.ReactNode
}

const DataFetcher = <T,>({ url, children }: DataFetcherProps<T>) => {
  const { data, loading, error } = useFetch<T>(url)
  return <>{children(data, loading, error)}</>
}

// Uso
<DataFetcher<Product[]> url="/api/products">
  {(products, loading, error) => (
    loading ? <Skeleton /> : 
    error ? <ErrorMessage error={error} /> :
    <ProductList products={products} />
  )}
</DataFetcher>
```

### 3. Compound Components
```typescript
const Modal = {
  Root: ModalRoot,
  Trigger: ModalTrigger,
  Content: ModalContent,
  Header: ModalHeader,
  Body: ModalBody,
  Footer: ModalFooter,
  Close: ModalClose
}

// Uso
<Modal.Root>
  <Modal.Trigger>
    <Button>Open Modal</Button>
  </Modal.Trigger>
  <Modal.Content>
    <Modal.Header>Title</Modal.Header>
    <Modal.Body>Content</Modal.Body>
    <Modal.Footer>
      <Modal.Close>
        <Button variant="secondary">Cancel</Button>
      </Modal.Close>
    </Modal.Footer>
  </Modal.Content>
</Modal.Root>
```

## Custom Hooks para Lógica Reutilizável

### 1. Data Management Hooks
```typescript
// useEntity - Hook genérico para CRUD
const useEntity = <T>(endpoint: string) => {
  const [data, setData] = useState<T[]>([])
  const [loading, setLoading] = useState(false)
  
  const create = async (item: Omit<T, 'id'>) => { /* ... */ }
  const update = async (id: string, item: Partial<T>) => { /* ... */ }
  const remove = async (id: string) => { /* ... */ }
  const fetch = async () => { /* ... */ }
  
  return { data, loading, create, update, remove, fetch }
}

// Uso específico
const useProducts = () => useEntity<Product>('/api/products')
const useCustomers = () => useEntity<Customer>('/api/customers')
```

### 2. Form Management Hooks
```typescript
const useForm = <T>(initialValues: T, validationSchema?: any) => {
  const [values, setValues] = useState<T>(initialValues)
  const [errors, setErrors] = useState<Record<string, string>>({})
  
  const handleChange = (name: keyof T, value: any) => { /* ... */ }
  const validate = () => { /* ... */ }
  const reset = () => { /* ... */ }
  
  return { values, errors, handleChange, validate, reset }
}
```

## Padrões Anti-Hardcode

### 1. Configuração Centralizada
```typescript
// constants/app-config.ts
export const APP_CONFIG = {
  api: {
    baseUrl: process.env.NEXT_PUBLIC_API_URL,
    timeout: 5000
  },
  ui: {
    itemsPerPage: 10,
    debounceDelay: 300,
    animationDuration: 200
  },
  business: {
    maxDiscountPercentage: 50,
    lowStockThreshold: 10,
    currencySymbol: 'R$'
  }
} as const
```

### 2. Theme System
```typescript
// styles/theme.ts
export const theme = {
  colors: {
    primary: {
      50: '#eff6ff',
      500: '#3b82f6',
      900: '#1e3a8a'
    }
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem'
  },
  typography: {
    sizes: {
      xs: '0.75rem',
      sm: '0.875rem',
      base: '1rem'
    }
  }
}
```

### 3. Internationalization Ready
```typescript
// hooks/useTranslation.ts
const useTranslation = () => {
  const t = (key: string, params?: Record<string, any>) => {
    // Lógica de tradução
    return translations[key] || key
  }
  
  return { t }
}

// Uso
const { t } = useTranslation()
return <Button>{t('common.save')}</Button>
```

## Testes e Qualidade

### 1. Component Testing Strategy
```typescript
// Teste de atom
describe('Button', () => {
  it('should render with correct variant', () => {
    render(<Button variant="primary">Click me</Button>)
    expect(screen.getByRole('button')).toHaveClass('bg-primary-500')
  })
})

// Teste de organism
describe('ProductCard', () => {
  it('should call onAddToCart when button is clicked', () => {
    const mockAddToCart = jest.fn()
    render(<ProductCard product={mockProduct} onAddToCart={mockAddToCart} />)
    
    fireEvent.click(screen.getByText('Add to Cart'))
    expect(mockAddToCart).toHaveBeenCalledWith(mockProduct)
  })
})
```

### 2. Storybook Integration
```typescript
// Button.stories.tsx
export default {
  title: 'Atoms/Button',
  component: Button,
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['primary', 'secondary', 'danger']
    }
  }
}

export const Primary = {
  args: {
    variant: 'primary',
    children: 'Button'
  }
}
```

## Performance Optimization

### 1. Lazy Loading
```typescript
const LazyProductList = lazy(() => import('./ProductList'))
const LazyReports = lazy(() => import('./Reports'))

// Uso com Suspense
<Suspense fallback={<ProductListSkeleton />}>
  <LazyProductList />
</Suspense>
```

### 2. Memoization
```typescript
const ProductCard = memo(({ product, onAddToCart }: ProductCardProps) => {
  // Component implementation
}, (prevProps, nextProps) => {
  return prevProps.product.id === nextProps.product.id &&
         prevProps.product.updatedAt === nextProps.product.updatedAt
})
```

## Documentação de Componentes

### 1. JSDoc Comments
```typescript
/**
 * Button component with multiple variants and sizes
 * 
 * @param variant - Visual style variant
 * @param size - Size of the button
 * @param loading - Shows loading spinner when true
 * @param disabled - Disables the button when true
 * @param onClick - Click handler function
 * @param children - Button content
 * 
 * @example
 * <Button variant="primary" size="md" onClick={handleClick}>
 *   Save Product
 * </Button>
 */
```

### 2. README por Componente
Cada componente complexo deve ter seu próprio README.md explicando:
- Propósito e uso
- Props disponíveis
- Exemplos de uso
- Considerações de performance
- Testes relacionados
