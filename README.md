# Claude Debug and Refactor Skills Plugin

45 framework-specific debugging and refactoring skills for Claude Code. Each skill auto-triggers based on your codebase context.

## Installation

### Option 1: Git Clone (Recommended)

```bash
git clone https://github.com/snakeo/claude-debug-and-refactor-skills-plugin.git ~/.claude/plugins/debug-and-refactor
```

Then restart Claude Code.

### Option 2: Manual Copy

Copy the `skills/` directory contents to `~/.claude/skills/`

## Skills Included

### Debug Skills (25)

| Skill | Frameworks/Languages | Use When |
|-------|---------------------|----------|
| `debug:angular` | Angular, TypeScript, RxJS | DI errors, change detection issues, zone.js problems |
| `debug:django` | Django, Python, DRF | TemplateDoesNotExist, N+1 queries, migration conflicts |
| `debug:docker` | Docker, Containers | Exit codes, OOM kills, networking issues, build failures |
| `debug:dotnet` | .NET, C#, ASP.NET Core | DI errors, EF Core issues, middleware pipeline problems |
| `debug:express` | Express.js, Node.js | Middleware issues, routing problems, async error handling |
| `debug:fastapi` | FastAPI, Python, Pydantic | Async/await issues, 422 validation errors, CORS problems |
| `debug:flask` | Flask, Python, Jinja2 | Routing 404/405, template errors, SQLAlchemy sessions |
| `debug:flutter` | Flutter, Dart | RenderFlex overflow, setState after dispose, platform channels |
| `debug:kubernetes` | Kubernetes, K8s, Helm | CrashLoopBackOff, ImagePullBackOff, OOMKilled, RBAC issues |
| `debug:laravel` | Laravel, PHP, Eloquent | Class errors, SQLSTATE issues, Blade templates, queues |
| `debug:nestjs` | NestJS, TypeScript | DI errors, circular dependencies, guard/interceptor issues |
| `debug:nextjs` | Next.js, React, SSR | Hydration mismatches, App Router issues, build failures |
| `debug:nuxtjs` | Nuxt.js, Vue, Nitro | SSR errors, composable issues, auto-import problems |
| `debug:pandas` | Pandas, Python, Data | SettingWithCopyWarning, KeyError, merge mismatches |
| `debug:pytorch` | PyTorch, CUDA, Deep Learning | CUDA OOM, NaN loss, shape mismatches, gradient issues |
| `debug:rails` | Ruby on Rails, ActiveRecord | RecordNotFound, N+1 queries, migration failures |
| `debug:react` | React, JSX, Hooks | Infinite re-renders, stale closures, hydration mismatches |
| `debug:react-native` | React Native, iOS, Android | Native module errors, Metro bundler, build failures |
| `debug:scikit-learn` | Scikit-learn, ML, Python | NotFittedError, shape mismatches, data leakage |
| `debug:spring-boot` | Spring Boot, Java, JPA | Bean errors, circular dependencies, Hibernate issues |
| `debug:svelte` | Svelte, SvelteKit | Reactivity failures, runes issues, SSR hydration |
| `debug:swiftui` | SwiftUI, iOS, macOS | View update failures, state management, NavigationStack |
| `debug:tensorflow` | TensorFlow, Keras, ML | Shape mismatches, GPU/CUDA failures, NaN loss |
| `debug:terraform` | Terraform, IaC | State locks, provider auth, dependency cycles |
| `debug:vue` | Vue.js 3, Pinia | Reactivity issues, computed caching, SSR hydration |

### Refactor Skills (20)

| Skill | Frameworks/Languages | Use When |
|-------|---------------------|----------|
| `refactor:angular` | Angular, TypeScript | Signals migration, standalone components, RxJS patterns |
| `refactor:django` | Django, Python | Fat models, service layer extraction, N+1 fixes |
| `refactor:docker` | Docker, Containers | Multi-stage builds, security hardening, layer optimization |
| `refactor:dotnet` | .NET, C# | SOLID principles, Clean Architecture, async patterns |
| `refactor:express` | Express.js, Node.js | Controller-service-repository, middleware patterns |
| `refactor:fastapi` | FastAPI, Python | Dependency injection, Pydantic v2, async patterns |
| `refactor:flask` | Flask, Python | Application factory, Blueprint organization |
| `refactor:flutter` | Flutter, Dart | Widget composition, state management, Freezed models |
| `refactor:kubernetes` | Kubernetes, K8s | Security contexts, Kustomize/Helm, GitOps patterns |
| `refactor:laravel` | Laravel, PHP | Actions, service layer, PHP 8.3+ features |
| `refactor:nestjs` | NestJS, TypeScript | Module organization, CQRS, repository pattern |
| `refactor:nextjs` | Next.js, React | Server Components, App Router, caching strategies |
| `refactor:nuxtjs` | Nuxt.js, Vue | Composables, data fetching, Nuxt Layers |
| `refactor:pandas` | Pandas, Python | Vectorization, method chaining, PyArrow backend |
| `refactor:pytorch` | PyTorch, Deep Learning | torch.compile, AMP, modular nn.Module design |
| `refactor:rails` | Ruby on Rails | Service objects, concerns, skinny controllers |
| `refactor:react` | React, JSX, Hooks | React 19 Compiler, Server Components, custom hooks |
| `refactor:react-native` | React Native | New Architecture, TurboModules, Fabric Renderer |
| `refactor:scikit-learn` | Scikit-learn, ML | Pipeline construction, cross-validation, feature engineering |
| `refactor:spring-boot` | Spring Boot, Java | Constructor injection, virtual threads, records |

## Usage

### Automatic Triggering

Skills trigger automatically based on your codebase context:

- Working on a React component with issues? `debug:react` loads automatically
- Editing a Laravel controller? `refactor:laravel` activates
- Debugging Kubernetes pods? `debug:kubernetes` kicks in

### Explicit Invocation

You can also invoke skills explicitly:

```
Use the debug:react skill to help me fix this infinite re-render loop
```

```
Use the refactor:laravel skill to clean up this fat controller
```

## What Each Skill Covers

### Debug Skills Include:
- Common error patterns with symptoms, causes, and solutions
- Framework-specific debugging tools and techniques
- Four-phase debugging methodology (Observe → Hypothesize → Test → Fix)
- Quick reference commands and diagnostic snippets

### Refactor Skills Include:
- Core refactoring principles (DRY, SOLID, Single Responsibility)
- Framework-specific patterns and anti-patterns
- Modern version features (React 19, Rails 8, Python 3.12+, etc.)
- Step-by-step refactoring process
- Before/after code examples

## License

MIT
