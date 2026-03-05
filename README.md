# CI/CD Pipelines — Desafio Outsera!!!

[![API Tests](https://github.com/rogerpdas/outsera-automacao-testes-api/actions/workflows/api-tests.yml/badge.svg?branch=main)](https://github.com/rogerpdas/outsera-automacao-testes-api/actions/workflows/api-tests.yml)
[![E2E Web Tests](https://github.com/rogerpdas/outsera-automacao-testes-web/actions/workflows/e2e-tests.yml/badge.svg?branch=main)](https://github.com/rogerpdas/outsera-automacao-testes-web/actions/workflows/e2e-tests.yml)
[![Mobile Tests](https://github.com/rogerpdas/outsera-automacao-testes-mobile/actions/workflows/mobile-ci.yml/badge.svg?branch=main)](https://github.com/rogerpdas/outsera-automacao-testes-mobile/actions/workflows/mobile-ci.yml)

Repositório centralizado com os pipelines de CI/CD para os três projetos de automação de testes do desafio técnico Outsera. Cada pipeline é independente — disparam em paralelo no push para `main` ou `develop` e podem ser acionados manualmente com parâmetros.

---

## Estrutura

```
.github/
└── workflows/
    ├── api-tests.yml      # API REST — RestAssured + Cucumber
    ├── e2e-tests.yml      # Web E2E — Selenium + Cucumber
    └── mobile-ci.yml      # Mobile Android — Appium + Cucumber
```

---

## Pipelines

### API Tests — `api-tests.yml`

Testa a API [DummyJSON](https://dummyjson.com) com RestAssured + Cucumber BDD.

**Jobs:**
```
build  ──►  health-check  ──►  api-tests
```

| Propriedade | Valor |
|---|---|
| Runner | `ubuntu-latest` |
| Java | 17 (Temurin) |
| Stack | RestAssured 5.4.0 + Cucumber 7.15.0 + JUnit 4 |
| Relatório | ExtentReports → GitHub Pages |

**Gatilhos:**

| Evento | Branches | Detalhes |
|---|---|---|
| `push` | `main`, `develop` | A cada commit |
| `pull_request` | `main`, `develop` | Antes de cada merge |
| `schedule` | — | Seg–Sex, 03:00 UTC |
| `workflow_dispatch` | — | Manual com `environment` e `tags` |

**Parâmetros do disparo manual:**

| Parâmetro | Opções | Padrão |
|---|---|---|
| `environment` | `hom`, `prod` | `hom` |
| `tags` | ex: `@smoke`, `@auth` | todas |

**Secrets necessários:**

| Secret | Valor |
|---|---|
| `API_USERNAME` | `emilys` |
| `API_PASSWORD` | `emilyspass` |

**Relatório:** https://rogerpdas.github.io/outsera-automacao-testes-api/

---

### E2E Web Tests — `e2e-tests.yml`

Testa o [SauceDemo](https://www.saucedemo.com) com Selenium 4 + Cucumber BDD. Login e Checkout rodam **em paralelo**.

**Jobs:**
```
build  ──►  health-check  ──►  e2e-login   ──►  summary
                          └──►  e2e-checkout ──┘
                               (paralelo)
```

| Propriedade | Valor |
|---|---|
| Runner | `ubuntu-latest` |
| Java | 17 (Temurin) |
| Stack | Selenium 4.18.1 + Cucumber 7.15.0 + WebDriverManager |
| Browsers | Chrome, Firefox, Edge (via `browser-actions`) |
| Relatório | ExtentReports → GitHub Pages |

**Gatilhos:**

| Evento | Branches | Detalhes |
|---|---|---|
| `push` | `main`, `develop` | A cada commit |
| `pull_request` | `main`, `develop` | Antes de cada merge |
| `schedule` | — | Seg–Sex, 04:00 UTC |
| `workflow_dispatch` | — | Manual com `browser`, `headless` e `tags` |

**Parâmetros do disparo manual:**

| Parâmetro | Opções | Padrão |
|---|---|---|
| `browser` | `chrome`, `firefox`, `edge` | `chrome` |
| `headless` | `true`, `false` | `true` |
| `tags` | ex: `@smoke`, `@login` | padrão por suíte |

**Secrets necessários:**

| Secret | Valor |
|---|---|
| `WEB_VALID_USERNAME` | `standard_user` |
| `WEB_VALID_PASSWORD` | `secret_sauce` |
| `WEB_LOCKED_USERNAME` | `locked_out_user` |

**Relatório:** https://rogerpdas.github.io/outsera-automacao-testes-web/

---

### Mobile Tests — `mobile-ci.yml`

Testa o [Swag Labs Mobile App](https://github.com/saucelabs/sample-app-mobile) com Appium 2 + Cucumber BDD. Emulador Android com aceleração KVM em Ubuntu.

**Jobs:**
```
build (ubuntu)  ──►  test-android (ubuntu + KVM)  ──►  summary
```

| Propriedade | Valor |
|---|---|
| Runner build | `ubuntu-latest` |
| Runner testes | `ubuntu-latest` + KVM |
| Java | 17 (Temurin) |
| Node | 20 |
| Stack | Appium 2.5.4 + UiAutomator2 2.9.1 + Cucumber 7.14.0 |
| Android | API 31 (Pixel 4) |
| Relatório | ExtentReports → GitHub Pages |

**Gatilhos:**

| Evento | Branches | Detalhes |
|---|---|---|
| `push` | `main`, `develop` | A cada commit |
| `pull_request` | `main` | Antes de cada merge |
| `schedule` | — | Seg–Sex, 05:00 UTC |
| `workflow_dispatch` | — | Manual com `tags` |

**Parâmetros do disparo manual:**

| Parâmetro | Exemplo | Padrão |
|---|---|---|
| `tags` | `@smoke`, `@login`, `@checkout` | `@smoke` |

**Relatório:** https://rogerpdas.github.io/outsera-automacao-testes-mobile/

---

## Execução Noturna Escalonada

Os três pipelines têm crons escalonados para evitar competição por recursos:

```
03:00 UTC  ──► API Tests
04:00 UTC  ──► E2E Web Tests
05:00 UTC  ──► Mobile Tests
```

---

## Boas Práticas Implementadas

| Prática | Detalhe |
|---|---|
| **Falha rápida** | Job `build` compila antes de qualquer coisa mais cara |
| **Health check** | Verifica disponibilidade da API/app antes de rodar testes |
| **Cache Maven** | Dependências cacheadas pela hash do `pom.xml` |
| **Cache npm** | Appium + UiAutomator2 cacheados por versão (Mobile) |
| **Concorrência** | `cancel-in-progress` evita runs redundantes |
| **Artefatos** | Relatórios salvos com `if: always()` por 30 dias |
| **Job Summary** | Resultado visível direto na aba Actions sem abrir artefatos |
| **GitHub Pages** | Relatório HTML sempre atualizado após cada execução |
| **Secrets** | Credenciais injetadas via GitHub Secrets — nunca no código |
| **Versões pinadas** | Appium e UiAutomator2 com versões fixas — builds reproduzíveis |

---

## Como Configurar os Secrets

Em cada repositório, acesse **Settings → Secrets and variables → Actions** e adicione os secrets correspondentes conforme as tabelas acima.

---

## Relatórios Publicados

| Pipeline | URL |
|---|---|
| API | https://rogerpdas.github.io/outsera-automacao-testes-api/ |
| Web | https://rogerpdas.github.io/outsera-automacao-testes-web/ |
| Mobile | https://rogerpdas.github.io/outsera-automacao-testes-mobile/ |

> Os relatórios são atualizados automaticamente após cada execução na branch `main`.

---

> Desenvolvido por **Rogério Pereira da Silva** — Automação QA  
> Desafio técnico Outsera · Java 17 · RestAssured · Selenium · Appium · Cucumber · GitHub Actions
