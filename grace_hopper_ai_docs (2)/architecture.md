# Technical Architecture

```mermaid
flowchart LR
    A[Next.js Frontend] --> B[Speech Recognition API]
    B --> C[FastAPI Backend]
    C --> D[Gemini API]
    C --> E[(Supabase)]
```

## Frontend
- Next.js
- Tailwind
- shadcn/ui

## Backend
- FastAPI

## Database
- Supabase PostgreSQL

## AI
- Gemini API

## Deployment
- Vercel
- Render
