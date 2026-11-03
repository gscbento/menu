# 🍽️ Menu

App multiplataforma (Android e iOS) em **Flutter** onde empresas publicam seu cardápio do dia e o usuário acompanha, direto no **widget da tela inicial do celular**, os cardápios dos lugares que escolheu seguir.

> A ideia: saber de relance "onde eu quero comer hoje" sem nem abrir o app — os cardápios das suas empresas favoritas aparecem no widget da home screen.

## 💡 Conceito

- **Empresas** publicam e editam o cardápio do dia.
- **Usuários** descobrem empresas e seguem as que quiserem.
- O **widget** da tela inicial mostra, de forma compacta, os cardápios de hoje das empresas seguidas.

É um modelo multi-empresa com assinaturas (follow): um "feed de cardápios do dia" na tela inicial.

## 👥 Atores

| Ator | Papel |
|------|-------|
| **Empresa** | Publica e edita o cardápio do dia |
| **Usuário** | Segue empresas e visualiza cardápios |
| **Widget** | Exibe os cardápios seguidos do dia na home screen |

## 🏗️ Arquitetura

```
┌──────────────────────────────────────────────┐
│  Backend (fonte de verdade)                    │
│  - Empresas, Cardápios, Follows                │
│  - API REST                                    │
│  - "cardápios de hoje das empresas que sigo"   │
│  - Push (FCM) quando empresa seguida publica   │
└───────────────┬────────────────────────────────┘
                │ HTTP + push
┌───────────────▼────────────────────────────────┐
│  App Flutter                                    │
│  Lado empresa:  CRUD do cardápio do dia         │
│  Lado usuário:  descobrir/seguir empresas,      │
│                 ver cardápios                    │
│  → grava "cardápios de hoje seguidos" no        │
│    storage compartilhado (home_widget)          │
└───────────────┬────────────────────────────────┘
                │ shared storage
┌───────────────▼────────────────────────────────┐
│  Widget nativo (Glance / WidgetKit)             │
│  Lista compacta: "Hoje — [empresas seguidas]"   │
│  Toque → abre o app no cardápio completo        │
└─────────────────────────────────────────────────┘
```

> **Importante:** o Flutter não desenha o widget da tela inicial. Home widgets são nativos — `AppWidgetProvider`/**Glance** no Android e **WidgetKit + SwiftUI** no iOS. O pacote [`home_widget`](https://pub.dev/packages/home_widget) faz a ponte: o app grava os dados num storage compartilhado e o código nativo lê e renderiza.

## 🗃️ Modelo de dados (núcleo)

```
Empresa
  id, nome, logo, categoria, localização

Cardapio
  id, empresa_id, data (dia), disponível
  itens: [ { nome, descrição, preço, categoria } ]

Usuario
  id, nome

Follow  (quem segue quem — personaliza o widget)
  usuario_id, empresa_id, criado_em
```

O `Follow` é o coração do produto: o widget mostra os cardápios de hoje **das empresas que o usuário segue**.

## 🎨 Decisões de design

- **Um app, dois modos** — conta de usuário (segue/vê) e conta de empresa (publica). Sem apps separados.
- **Widget mostra pouco, por design** — WidgetKit (iOS) é estático e sem scroll. Exibir destaques (prato do dia) de N empresas; toque abre o app.
- **O app prepara o resumo** — não grava o cardápio inteiro no storage do widget, apenas `{ empresa, logo, destaque_do_dia }`. O widget só renderiza.
- **Atualização** — push (FCM) quando uma empresa seguida publica + background fetch periódico como rede de segurança.
- **Virada do dia** — como é "cardápio de hoje", o backend resolve a data/fuso e o widget faz refresh pela manhã.

## 🧰 Stack

- **Flutter** + [`home_widget`](https://pub.dev/packages/home_widget)
- **Android:** Jetpack **Glance** (Kotlin)
- **iOS:** **WidgetKit** + SwiftUI
- **Backend:** API REST + **FCM** (push); autenticação com papéis (usuário/empresa)
- **Estado (Flutter):** Riverpod ou Bloc

## 🗺️ Roadmap (MVP primeiro)

- [ ] **Fase 1 — Núcleo (sem widget):** empresa publica cardápio; usuário segue e vê a lista no app.
- [ ] **Fase 2 — Widget:** `home_widget` + Glance/WidgetKit exibindo as empresas seguidas.
- [ ] **Fase 3 — Push + background refresh:** atualização automática e virada de dia.
- [ ] **Fase 4 — Descoberta:** busca por categoria, localização ("perto de mim"), favoritos.

O valor central (seguir e ver cardápios do dia) já se prova na Fase 1; o widget é a evolução.

## 🚀 Começando

> Em breve — instruções de setup conforme o projeto evolui.

```bash
# Pré-requisitos: Flutter SDK
flutter pub get
flutter run
```

## 📄 Licença

A definir.
