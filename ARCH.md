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

O projeto original tinha os arquivos com responsabilidades corretas mas nas pastas erradas. O processo foi:
app_errors.dart: screens/ → core/errors/
todo_local_datasource.dart: screens/ → data/datasources/
todo_remote_datasource.dart: utils/ → data/datasources/
todo_repository.dart (interface): services/ → data/repositories/
todo_model.dart: widgets/ → data/models/
E os imports relativos foram corrigidos por sed para refletir os novos caminhos.
Os arquivos foram reescritos diretamente nos novos caminhos com imports já corretos, e as pastas antigas foram deletadas. As mudanças estruturais principais foram:
TodoViewModel passou a depender da interface TodoRepository (domain), não da impl concreta — isso foi uma mudança real no código, não só de pasta.
TodoRepositoryImpl passou a receber remote e local por injeção no construtor, em vez de instanciar diretamente dentro da classe.
Os providers foram movidos de main.dart para app_root.dart, deixando o main() com apenas uma linha.
As pastas services/, utils/, screens/ (datasource), ui/, viewmodels/ e widgets/ foram eliminadas e substituídas pela hierarquia features/todos/{domain,data,presentation}.

