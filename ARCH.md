# ARCH — Preencha após refatoração

## Estrutura final (cole a árvore de pastas)

aula02_mobileII/
│
├── lib/
│   ├── main.dart
│   ├── app_root.dart
│   │
│   ├── core/
│   │   └── errors/
│   │       └── app_errors.dart
│   │
│   └── features/
│       └── todos/
│           ├── data/
│           │   ├── datasources/
│           │   │   ├── local_datasource/
│           │   │   │   └── todo_local_datasource.dart
│           │   │   └── remote_datasource/
│           │   │       └── todo_remote_datasource.dart
│           │   ├── models/
│           │   │   └── todo_model.dart
│           │   └── repositories/
│           │       └── todo_repository_impl.dart
│           │
│           ├── domain/
│           │   ├── entities/
│           │   │   └── todo.dart
│           │   └── repositories/
│           │       └── todo_repository.dart
│           │
│           └── presentation/
│               ├── pages/
│               │   └── todos_page.dart
│               ├── viewmodels/
│               │   └── todo_viewmodel.dart
│               └── widgets/
│                   └── add_todo_dialog.dart
│
├── test/
│   └── widget_test.dart
│
├── web/
├── linux/
├── macos/
└── windows/

## Fluxo de dependências
UI -> ViewModel -> Repository -> (RemoteDataSource, LocalDataSource)

[TodosPage / AddTodoDialog]   ← presentation/pages, presentation/widgets
          │ consome estado via context.watch<>
          ▼
    [TodoViewModel]            ← presentation/viewmodels
          │ chama métodos do contrato abstrato
          ▼
    [TodoRepository]           ← domain/repositories  (interface)
          │ implementado por
          ▼
 [TodoRepositoryImpl]          ← data/repositories
     │            │
     ▼            ▼
[TodoRemote   [TodoLocal
DataSource]    DataSource]     ← data/datasources
 (HTTP)        (SharedPrefs)

## Decisões
- Onde ficou a validação?

No `TodoViewModel` — validação de campo vazio (`title.trim().isEmpty`) é regra de apresentação, não de negócio. Se houver regras mais complexas (ex.: título máximo de 200 chars), mover para um `UseCase` no domain.

- Onde ficou o parsing JSON?

Em `TodoModel.fromJson()` dentro de `data/models/`. A entidade `Todo` (domain) não sabe nada de JSON.

- Como você tratou erros?

- `TodoRemoteDataSource` lança `Exception` para HTTP != 2xx.
- `TodoRepositoryImpl` deixa propagar (não engole).
- `TodoViewModel` captura e expõe via `errorMessage` (String) para a UI.
- Rollback otimista em `toggleCompleted`: o item é revertido localmente se a chamada remota falhar.