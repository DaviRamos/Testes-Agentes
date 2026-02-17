# Agents.md - Desenvolvimento .NET Backend + React Vite Frontend

## 🎯 Visão Geral da Stack

### Backend
- **Framework**: ASP.NET Core 8.0+
- **Linguagem**: C# 12
- **ORM**: Entity Framework Core
- **API**: RESTful / GraphQL
- **Autenticação**: JWT / Identity

### Frontend
- **Framework**: React 18+
- **Build Tool**: Vite
- **Linguagem**: TypeScript
- **Estado**: React Query / Zustand / Redux Toolkit
- **UI**: Tailwind CSS / Material-UI / shadcn/ui

## 📁 Estrutura do Projeto

```
projeto-root/
├── backend/
│   ├── src/
│   │   ├── Api/                    # Camada de apresentação
│   │   │   ├── Controllers/
│   │   │   ├── Middlewares/
│   │   │   ├── Filters/
│   │   │   └── Program.cs
│   │   ├── Application/            # Lógica de negócio
│   │   │   ├── DTOs/
│   │   │   ├── Services/
│   │   │   ├── Interfaces/
│   │   │   └── Validators/
│   │   ├── Domain/                 # Entidades e regras
│   │   │   ├── Entities/
│   │   │   ├── Enums/
│   │   │   └── ValueObjects/
│   │   └── Infrastructure/         # Acesso a dados
│   │       ├── Data/
│   │       ├── Repositories/
│   │       └── Configurations/
│   └── tests/
│       ├── UnitTests/
│       └── IntegrationTests/
│
└── frontend/
    ├── src/
    │   ├── components/             # Componentes reutilizáveis
    │   │   ├── ui/
    │   │   └── shared/
    │   ├── features/               # Funcionalidades por domínio
    │   │   └── users/
    │   │       ├── api/
    │   │       ├── components/
    │   │       ├── hooks/
    │   │       └── types/
    │   ├── services/               # Serviços de API
    │   ├── hooks/                  # Custom hooks
    │   ├── utils/                  # Utilitários
    │   ├── types/                  # TypeScript types
    │   ├── routes/                 # Configuração de rotas
    │   ├── store/                  # Gerenciamento de estado
    │   └── App.tsx
    ├── public/
    ├── vite.config.ts
    └── package.json
```

## 🔧 Configuração do Ambiente

### Backend (.NET)

#### Requisitos
- .NET 8 SDK ou superior
- SQL Server / PostgreSQL
- Visual Studio 2022 / VS Code / Rider

#### Configuração Inicial

```bash
# Criar novo projeto Web API
dotnet new webapi -n MyApi

# Adicionar pacotes essenciais
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
dotnet add package FluentValidation.AspNetCore
dotnet add package Swashbuckle.AspNetCore
```

#### Program.cs (Exemplo)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuração de serviços
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:5173")
              .AllowAnyMethod()
              .AllowAnyHeader()
              .AllowCredentials();
    });
});

// DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// AutoMapper
builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseCors("AllowFrontend");
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Frontend (React + Vite)

#### Configuração Inicial

```bash
# Criar projeto Vite com React + TypeScript
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install

# Dependências essenciais
npm install react-router-dom
npm install @tanstack/react-query
npm install axios
npm install zustand

# UI (escolher uma)
npm install tailwindcss postcss autoprefixer
npm install @mui/material @emotion/react @emotion/styled

# Utilitários
npm install date-fns zod
npm install -D @types/node
```

#### vite.config.ts

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'https://localhost:7001',
        changeOrigin: true,
        secure: false,
      },
    },
  },
})
```

## 🏗️ Padrões de Desenvolvimento

### Backend - Padrões .NET

#### 1. Controller Pattern

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<UserDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<UserDto>>> GetAll()
    {
        var users = await _userService.GetAllAsync();
        return Ok(users);
    }
    
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<UserDto>> GetById(int id)
    {
        var user = await _userService.GetByIdAsync(id);
        return user == null ? NotFound() : Ok(user);
    }
    
    [HttpPost]
    [ProducesResponseType(typeof(UserDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<UserDto>> Create(CreateUserDto dto)
    {
        var user = await _userService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }
}
```

#### 2. Service Pattern

```csharp
public interface IUserService
{
    Task<IEnumerable<UserDto>> GetAllAsync();
    Task<UserDto?> GetByIdAsync(int id);
    Task<UserDto> CreateAsync(CreateUserDto dto);
    Task<bool> UpdateAsync(int id, UpdateUserDto dto);
    Task<bool> DeleteAsync(int id);
}

public class UserService : IUserService
{
    private readonly IRepository<User> _repository;
    private readonly IMapper _mapper;
    
    public UserService(IRepository<User> repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
    
    public async Task<IEnumerable<UserDto>> GetAllAsync()
    {
        var users = await _repository.GetAllAsync();
        return _mapper.Map<IEnumerable<UserDto>>(users);
    }
    
    // Implementar outros métodos...
}
```

#### 3. Repository Pattern

```csharp
public interface IRepository<T> where T : class
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }
    
    // Implementar outros métodos...
}
```

### Frontend - Padrões React

#### 1. Serviço de API (Axios)

```typescript
// src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || '/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Interceptor para token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

#### 2. React Query (API Calls)

```typescript
// src/features/users/api/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import api from '@/services/api';
import { User, CreateUserDto } from '../types';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const { data } = await api.get<User[]>('/users');
      return data;
    },
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (userData: CreateUserDto) => {
      const { data } = await api.post<User>('/users', userData);
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

#### 3. Componente com Tipagem

```typescript
// src/features/users/components/UserList.tsx
import React from 'react';
import { useUsers } from '../api/useUsers';

export const UserList: React.FC = () => {
  const { data: users, isLoading, error } = useUsers();
  
  if (isLoading) return <div>Carregando...</div>;
  if (error) return <div>Erro ao carregar usuários</div>;
  
  return (
    <div className="grid gap-4">
      {users?.map((user) => (
        <div key={user.id} className="p-4 border rounded">
          <h3 className="font-bold">{user.name}</h3>
          <p className="text-gray-600">{user.email}</p>
        </div>
      ))}
    </div>
  );
};
```

#### 4. Custom Hook

```typescript
// src/hooks/useDebounce.ts
import { useEffect, useState } from 'react';

export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

## 🔒 Autenticação JWT

### Backend

```csharp
// Adicionar pacotes
// dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });
```

### Frontend

```typescript
// src/contexts/AuthContext.tsx
import React, { createContext, useContext, useState } from 'react';
import api from '@/services/api';

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const { data } = await api.post('/auth/login', { email, password });
    localStorage.setItem('token', data.token);
    setUser(data.user);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated: !!user }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};
```

## ✅ Boas Práticas

### Backend
- ✅ Use injeção de dependência
- ✅ Implemente validação com FluentValidation
- ✅ Use DTOs para transferência de dados
- ✅ Implemente logging (Serilog)
- ✅ Trate exceções globalmente
- ✅ Documente API com Swagger
- ✅ Escreva testes unitários e de integração
- ✅ Use async/await consistentemente
- ✅ Implemente paginação em listagens
- ✅ Versione sua API

### Frontend
- ✅ Use TypeScript para type safety
- ✅ Organize por features/domínio
- ✅ Implemente lazy loading de rotas
- ✅ Use React Query para cache
- ✅ Implemente error boundaries
- ✅ Otimize performance (useMemo, useCallback)
- ✅ Use variáveis de ambiente (.env)
- ✅ Implemente testes (Vitest, Testing Library)
- ✅ Use ESLint e Prettier
- ✅ Implemente acessibilidade (a11y)

## 🚀 Deploy

### Backend
```bash
# Publicar aplicação
dotnet publish -c Release -o ./publish

# Docker
docker build -t myapi .
docker run -p 8080:80 myapi
```

### Frontend
```bash
# Build de produção
npm run build

# Preview local
npm run preview
```

## 📚 Recursos Úteis

### Documentação
- [ASP.NET Core](https://docs.microsoft.com/aspnet/core)
- [React](https://react.dev)
- [Vite](https://vitejs.dev)
- [React Query](https://tanstack.com/query)

### Ferramentas
- Postman / Insomnia (testes de API)
- SQL Server Management Studio
- React DevTools
- Redux DevTools

---

**Última atualização**: 2025
**Versão**: 1.0.0