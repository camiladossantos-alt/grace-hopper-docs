# Projeto Grace Hopper - Architecture Decision Record (ADR)

## Overview
Este documento registra as principais decisões arquiteturais do projeto Grace Hopper, as alternativas consideradas e as justificativas técnicas.

---

## ADR-001: Framework Frontend

### Contexto
Necessidade de escolher um framework React moderno que suporte SSR, performance otimizada e integração com APIs.

### Decisão
**Usar Next.js 14+ com TypeScript**

### Alternativas Consideradas
1. **React SPA + Vite** 
   - Pros: Mais leve, build mais rápido
   - Cons: Sem SSR, SEO limitado, routing manual

2. **Remix**
   - Pros: Excelente para formulários e data loading
   - Cons: Curva de aprendizado maior, comunidade menor

3. **Next.js** ✅
   - Pros: SSR/SSG, SEO, performance, ecosystem maduro
   - Cons: Overhead maior para SPAs puras

### Justificativa
- **Performance**: Image optimization, Font optimization automáticos
- **SEO**: SSR nativo para melhor indexação
- **Developer Experience**: File-based routing, API routes integradas
- **Deployment**: Vercel integration perfeita
- **Ecosystem**: Suporte a shadcn/ui, TypeScript, Tailwind

### Implementação
```
NextJS 14
├── App Router (não Pages Router)
├── TypeScript strict mode
├── Tailwind CSS
├── shadcn/ui components
└── API Routes para endpoints simples
```

---

## ADR-002: Backend Framework

### Contexto
Necessidade de framework Python escalável, rápido e com suporte nativo a IA/LLMs.

### Decisão
**Usar FastAPI ao invés de Django/Flask**

### Alternativas Consideradas
1. **Django + DRF**
   - Pros: Muito completo, ORM excelente, comunidade enorme
   - Cons: Overhead pesado, opinionado, mais lento

2. **Flask**
   - Pros: Minimalista, flexible
   - Cons: Muito manual para escala, comunidade menor

3. **FastAPI** ✅
   - Pros: Performance, async/await nativo, OpenAPI automático
   - Cons: Ecossistema menor que Django

### Justificativa
- **Performance**: Async-first, rodas bem em serverless (Render)
- **IA-Ready**: Integração direta com APIs de LLM, bom para AI workloads
- **Developer Experience**: Auto-documentation, type hints, validação automática
- **Escalabilidade**: Suporta WebSockets, streaming responses
- **Deployment**: Fácil no Render, suporta auto-scaling

### Implementação
```
FastAPI
├── Pydantic para validação
├── SQLAlchemy para ORM
├── Async endpoints
├── Background tasks
├── Logging estruturado
└── Rate limiting
```

---

## ADR-003: Database

### Contexto
Necessidade de database confiável, com suporte a real-time e auth integrado.

### Decisão
**Usar Supabase (PostgreSQL + extras) ao invés de Firebase/MongoDB**

### Alternativas Consideradas
1. **Firebase/Firestore**
   - Pros: Serverless, real-time, fácil de setup
   - Cons: Query limitations, vendor lock-in, caro em escala

2. **MongoDB Atlas**
   - Pros: NoSQL, flexible schema
   - Cons: Não ideal para dados estruturados, real-time complexo

3. **Supabase** ✅
   - Pros: PostgreSQL completo, Auth, Real-time, Vector search
   - Cons: Self-hosted pode ser complexo (mas usamos managed)

### Justificativa
- **Dados Estruturados**: PostgreSQL é ideal para usuarios, interviews, feedback
- **Real-time**: Supabase Realtime para atualizar dashboard live
- **Auth Integrado**: Supabase Auth com OAuth Google nativo
- **RLS (Row-Level Security)**: Segurança em nível de database
- **Full-Text Search**: Para buscar interviews/feedback
- **Vector Search**: Futuro para embedding-based search
- **Backup**: Managed backups automáticos

### Schema Design
```sql
-- Normalization: 3NF
-- Tables separadas: users, interviews, feedback, analytics
-- Indices em foreign keys e queries frequentes
-- RLS policies para segurança
-- Audit log via triggers
```

---

## ADR-004: API de IA / LLM

### Contexto
Necessidade de LLM para análise conversacional, feedback estruturado e prompt engineering.

### Decisão
**Usar Google Gemini API ao invés de OpenAI/Anthropic Claude**

### Alternativas Consideradas
1. **OpenAI GPT-4/4o**
   - Pros: SOTA performance, excelente para instruções
   - Cons: Mais caro, rate limits mais apertados

2. **Anthropic Claude**
   - Pros: Excelente em análise estruturada, context window grande
   - Cons: Latência mais alta, menos integrations

3. **Google Gemini** ✅
   - Pros: Latência baixa, boa performance, preço competitive, vision
   - Cons: Menos opiniões online vs OpenAI

### Justificativa
- **Performance**: Latência ~ 200-500ms (melhor que competidores)
- **Custo**: $0.075/M input tokens vs $0.30/M GPT-4o
- **Vision Capability**: Futuro para análise de body language via webcam
- **Integration**: Fácil com Vertex AI (escala enterprise)
- **Structured Output**: Suporta JSON schema para feedback estruturado

### Implementação
```python
# Prompt template com exemplos in-context
# Guardrails via prompt engineering
# Caching de prompts para economia
# Rate limiting: 60 req/min
# Fallback: cache de feedback anterior
```

---

## ADR-005: Speech-to-Text

### Contexto
Necessidade de transcrever áudio do usuário em tempo real, com alta precisão.

### Decisão
**Usar Web Audio API (frontend) + Google Cloud Speech-to-Text (backend)**

### Alternativas Consideradas
1. **Apenas Web Audio API (local)**
   - Pros: Privacidade, sem latência de rede
   - Cons: Precisão ruim, sem punctuation

2. **Whisper (OpenAI local)**
   - Pros: Excelente precisão, open source
   - Cons: Computacionalmente pesado, lento

3. **Google Cloud Speech-to-Text** ✅
   - Pros: Excelente precisão, real-time, handles accents
   - Cons: Custa money, dependência de API

### Justificativa
- **Precisão**: 95%+ accuracy em português/inglês
- **Real-time**: Streaming support, transcrição enquanto grava
- **Handling**: Punctuation, capitalization automáticas
- **Cost**: $0.006 per 15 seconds (~$0.024 por entrevista de 1min)
- **Multilingual**: Português e Inglês

### Implementação
```javascript
// Frontend: Web Audio API para captura
// Enviar chunks de 1s para backend
// Backend: Google Cloud STT
// Cache resultados para retry
```

---

## ADR-006: Deploy & Hosting

### Contexto
Necessidade de deploy rápido, confiável e escalável com boa DX.

### Decisão
**Frontend: Vercel | Backend: Render.com**

### Alternativas Consideradas
1. **Monolithic (tudo um lugar)**
   - Pros: Operacionalmente simples
   - Cons: Escalada acoplada, difícil separar recursos

2. **AWS EC2 + RDS**
   - Pros: Máximo controle
   - Cons: Ops complexo, auto-scaling manual

3. **Vercel + Render** ✅
   - Pros: Serverless, auto-scaling, DX excelente, CI/CD free
   - Cons: Menos customização que AWS

### Justificativa
- **Frontend (Vercel)**
  - Edge functions para middleware
  - Automatic image optimization
  - Preview deployments automáticas
  - Analytics integrado
  - $0 a $20/mês para MVP

- **Backend (Render)**
  - Auto-scaling horizontal
  - Ambiente managed (não precisa DevOps)
  - PostgreSQL managed
  - $7/mês por dyno + database
  - Webhooks e CRON jobs integrados

### Arquitetura
```
┌─────────────────────────────────────┐
│ Client (Browser)                    │
└────────┬────────────────────────────┘
         │
    ┌────▼─────┐
    │ Vercel   │ (Edge)
    └────┬─────┘
         │
    ┌────▼──────────┐
    │ FastAPI on    │
    │ Render        │ (Auto-scaling)
    └────┬──────────┘
         │
    ┌────▼──────────────┐
    │ Supabase          │
    │ PostgreSQL + Auth │
    └───────────────────┘
```

---

## ADR-007: Real-time Features

### Contexto
Necessidade de atualizar dashboard em tempo real quando novos feedbacks chegam.

### Decisão
**Usar Supabase Realtime (Postgres replication) para MVP**

### Alternativas Consideradas
1. **WebSockets custom**
   - Pros: Total controle
   - Cons: Operacionalmente pesado, escalação difícil

2. **Socket.io**
   - Pros: Fallbacks, excelente DX
   - Cons: Infrastructure pesada, caro escalar

3. **Supabase Realtime** ✅
   - Pros: Usa Postgres nativamente, auto-scaling
   - Cons: Menos flexible que custom

### Justificativa
- **Simplicidade**: Subscribe a tabelas diretamente do frontend
- **Escalabilidade**: Sem servidor separado de WebSocket
- **Custo**: Incluído no plano Supabase
- **MVP**: Suficiente para fase inicial

### Implementação
```typescript
// Frontend
const subscription = supabase
  .from('interviews')
  .on('INSERT', (payload) => {
    updateDashboard(payload.new)
  })
  .subscribe()
```

---

## ADR-008: Autenticação & Segurança

### Contexto
Necessidade de autenticação segura, fácil de usar, com suporte a Google OAuth.

### Decisão
**Usar Supabase Auth com Google OAuth + JWT**

### Alternativas Consideradas
1. **Auth0**
   - Pros: Muito completo, enterprise-ready
   - Cons: Caro ($15+/mês), overkill para MVP

2. **NextAuth.js**
   - Pros: Opensource, flexible
   - Cons: Setup complexo, mais manutenção

3. **Supabase Auth** ✅
   - Pros: Integrado com database, OAuth Google, JWT
   - Cons: Menos features que Auth0

### Justificativa
- **Integração**: Está no mesmo banco de dados
- **Segurança**: JWT com refresh tokens, RLS automático
- **UX**: Google login de 1-click para usuários novos
- **Custo**: Free tier generoso
- **LGPD**: Dados sempre sob controle (self-hosted option se necessário)

### Implementação
```
JWT Token: {userId, email, exp}
Refresh Token: 7 dias
Access Token: 1 hora
HttpOnly Cookies para refresh
CORS configurado corretamente
```

---

## ADR-009: Video/Recording Storage

### Contexto
Necessidade de armazenar áudio das entrevistas para processamento e possível replay.

### Decisão
**Usar Supabase Storage (S3-compatible) com processamento via background jobs**

### Alternativas Consideradas
1. **AWS S3 direto**
   - Pros: Mais flexível, mais features
   - Cons: Setup complexo, mais caro

2. **Google Cloud Storage**
   - Pros: Bem integrado com Google APIs
   - Cons: Custo, vendor lock-in

3. **Supabase Storage** ✅
   - Pros: Integrado, S3 compatible, managed
   - Cons: Menos features que S3 puro

### Justificativa
- **Integração**: Mesmo bucket que Supabase
- **RLS**: Controle de acesso a nível de bucket
- **Cost**: Competitivo com S3
- **Simplicity**: Não precisa AWS console

### Implementação
```
Audio files → Supabase Storage
Metadata → interviews table
Processing → Background task (Render)
Cleanup → 30 dias após entrevista
```

---

## ADR-010: Analytics & Monitoring

### Contexto
Necessidade de entender uso, comportamento de usuários e detectar problemas.

### Decisão
**Vercel Analytics (frontend) + Custom events (backend) + Supabase logs**

### Alternativas Consideradas
1. **Google Analytics 4**
   - Pros: Muito completo
   - Cons: Privacy concerns, overkill para MVP

2. **Mixpanel/Amplitude**
   - Pros: Excelente para product analytics
   - Cons: Caro ($500+), setup complexo

3. **Custom (Vercel + Supabase + simple dashboard)**
   - Pros: Controle total, barato, privacy-first
   - Cons: Menos polished que Enterprise tools

### Justificativa
- **Frontend**: Vercel Analytics para Web Vitals
- **Backend**: Eventos custom em Supabase (analytics_events table)
- **Dashboards**: Metabase ou Looker Studio (free)
- **Logs**: Render + CloudWatch para debugging
- **Cost**: < $100/mês total

### Eventos Principais
```
signup_complete
first_interview_started
interview_completed
feedback_viewed
score_improvement
error_occurred
```

---

## ADR-011: Testing Strategy

### Contexto
Necessidade de qualidade de código, confiabilidade e regressão prevention.

### Decisão
**Unit tests + Integration tests + E2E coverage mínima (80% coverage para core)**

### Alternativas Consideradas
1. **Sem testes (YOLO)**
   - Pros: Rápido inicialmente
   - Cons: Bugs em produção, refactoring assustador

2. **Testes completos (100%)**
   - Pros: Máxima confiança
   - Cons: Tempo, manutenção pesada

3. **Balanced approach** ✅
   - Pros: Qualidade + velocidade
   - Cons: Equilibrio complicado

### Justificativa
- **Unit tests (Jest/Pytest)**
  - Funções puras, utils, componentes simples
  - Target: 80% coverage para core

- **Integration tests**
  - API endpoints com mock/stub
  - Database queries
  - Auth flows

- **E2E tests (Playwright)**
  - Happy paths principais
  - Signup → Interview → Feedback
  - Desktop + Mobile

---

## ADR-012: CI/CD Pipeline

### Contexto
Necessidade de deploy automático, confiável e com rollback fácil.

### Decisão
**GitHub Actions + Preview Deployments + Staging + Production**

### Estágios
```
1. PR aberta
   ↓ GitHub Actions
   - Lint (ESLint + Prettier)
   - Tests (Jest + Pytest)
   - Build check

2. Preview Deploy (Vercel)
   - Automatic para cada PR
   - Link compartilhável

3. Main branch
   ↓ Merge → Deploy automático
   - Staging (auto-deploy)
   - Manual approval → Production

4. Production
   - Monitoring ativo (24h)
   - Rollback fácil (1-click)
```

---

## ADR-013: Error Handling & Logging

### Contexto
Necessidade de entender o que quebrou em produção e poder debugar.

### Decisão
**Structured logging + Sentry para frontend + CloudWatch para backend**

### Implementação
```
Frontend:
- Sentry for exceptions
- Console logs estruturados (JSON)
- Network errors captured

Backend:
- Python logging module
- CloudWatch/Render logs
- Request/response logging
- Slow query alerting

Database:
- Supabase logs
- Query performance monitoring
```

---

## Decisões Futuras (V2+)

### Quando considerar mudanças:

1. **Se performance degradar**
   - Migrar para Postgres cache (Redis)
   - Edge computing for image processing

2. **Se escala aumentar 100x**
   - Considerar tirar content estático para CDN edge
   - Database sharding
   - Queue system (Bull/Celery)

3. **Se precisar de compliance enterprise**
   - HIPAA/SOC2 compliance
   - Self-hosted option
   - Migrar para AWS/GCP

---

## Trade-offs & Compromissos

| Decisão | Benefício | Custo |
|---------|-----------|-------|
| Next.js | Performance + DX | Overhead vs SPA puro |
| FastAPI | Performance | Comunidade menor |
| Supabase | Integrado + Simple | Lock-in |
| Gemini | Latência baixa | Menos opções de customização |
| Vercel+Render | DX + Serverless | Menos controle que self-hosted |
| Real-time via Supabase | Simples | Não pode fazer complex sync |

---

## Review Schedule

- **ADR Review**: Trimestral
- **Technology Review**: Anual
- **Next Review Date**: Q4 2026

---

**Versão:** 1.0  
**Atualizado:** Maio 2026  
**Owner:** Tech Lead
