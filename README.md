# Daora Kids - DK Forms v0.28.0 (Headless SPA) 📝

Módulo de gestão do funil de voluntariado da plataforma Daora Kids. Este sistema gerencia desde a candidatura inicial até a formalização jurídica e a entrega final de materiais de narração através de uma esteira de produção ativa, agora com suporte total a comunicação multilíngue personalizada.

## 🏗️ Arquitetura e Fluxos

O sistema segue uma arquitetura **Headless** com uma SPA em React no frontend e uma API RESTful em PHP 8 no backend.

### 1. Diagrama de Classes (Modelo de Dados)

```mermaid
classDiagram
    class SubmissionCore {
        +String id
        +String status
        +String language_preference
        +String contact_language
        +String narrated_language
        +String image_fs_path
        +String child_name
        +Date child_birthdate
        +String child_gender
        +String guardian_name
        +String guardian_email
        +String guardian_phone
        +Date guardian_birthdate
        +String ip_address
        +Int abuse_score
        +String email_token_hash
        +Timestamp email_token_expires_at
        +Timestamp email_confirmed_at
        +Text status_history_log
        +JSON raw_payload
        +Timestamp created_at
    }

    class Lesson {
        +Int id
        +String submission_id
        +Int year
        +Int quarter
        +Int lesson_number
        +String language
        +Text narration_text
        +Text narration_notes
        +String status
        +Timestamp created_at
        +Timestamp updated_at
    }

    class LegalContractState {
        +String clicksign_envelope_key
        +String document_status
        +String pdf_fs_path
        +Int followup_emails_sent
        +Timestamp last_followup_at
    }

    class AdminReviewLog {
        +String admin_identifier
        +String review_notes
        +String action_taken
        +Timestamp created_at
    }

    SubmissionCore "1" -- "0..1" LegalContractState : vincula
    SubmissionCore "1" -- "0..*" AdminReviewLog : possui logs
    SubmissionCore "1" -- "0..*" Lesson : possui lições atribuídas
```

### 2. Fluxo de Operação (Sequência)

O diagrama abaixo ilustra a interação entre o Voluntário, a SPA em React, o Backend PHP e as APIs externas.

```mermaid
sequenceDiagram
    participant U as Voluntário (Pais)
    participant F as React SPA
    participant A as PHP API
    participant DB as MySQL
    participant C as ClickSign API

    Note over U,DB: Fase A: Onboarding
    U->>F: Preenche Formulário + Foto
    F->>A: POST /submit
    A->>DB: INSERT submission_core
    A->>U: E-mail de Confirmação
    
    Note over U,DB: Fase B: Formalização
    Admin->>F: Aprova Candidatura
    F->>A: POST /submissions/{id}/approve
    A->>U: E-mail com link de Consentimento
    U->>A: GET /validate-consent?id={id}
    A->>C: Template ClickSign (Digital Sign)
    C->>U: E-mail para Assinatura Digital
    
    Note over U,DB: Fase C: Produção Ativa
    Admin->>F: Seleciona Voluntário e Atribui Lição
    F->>A: POST /admin/lessons
    A->>DB: INSERT lessons (Status: SELECTED)
    Admin->>F: Envia Instruções (E-mail automático)
    F->>A: POST /lessons/{id}/send-instructions
    A->>U: E-mail com Roteiro de Narração
    U->>A: Envio do material (WhatsApp/E-mail)
    Admin->>F: Marca como Recebido / Revisado
    F->>A: POST /lessons/{id}/complete-recording
    A->>DB: Status -> COMPLETED
```

Para uma descrição verbal completa de cada etapa, consulte: 👉 **[Documentação de Workflow (BPM)](./workflow.md)**

## 📊 Inteligência e Monitoramento (Dashboard)

O Dashboard Administrativo atua como um centro de comando da produção:

1.  **Funil de Conversão:** Visualização em tempo real de quantos candidatos estão em cada etapa crítica (Inscritos -> Confirmados -> Formalizados -> Concluídos).
2.  **Gestão de Backlog:** Indicador de voluntários aptos que ainda não receberam instruções de gravação.
3.  **Fila de Curadoria:** Destaque para materiais recebidos que aguardam revisão técnica.
4.  **SLA Automático:** O sistema monitora prazos de 7 dias para confirmação de e-mail e 14 dias para assinatura de contrato, cancelando registros inativos automaticamente via Pseudo-Cron.

## 🛠️ Tecnologias Utilizadas
- **Frontend**: React 19, Tailwind CSS, Lucide Icons, Framer Motion.
- **Backend**: PHP 8 (Pure), PDO MySQL.
- **Integrações**: ClickSign (Assinatura), Cloudflare Turnstile (Segurança), AbuseIPDB (Detecção de Fraude).

## 🔐 Segurança e Performance
- **Zero Inline Styles**: Todo o design é baseado em utilitários CSS e Tailwind.
- **Semantic Keys**: Abandono de siglas legadas em favor de nomes claros para facilitar auditoria e manutenção.
- **Audit Log**: Sistema de log atômico (`status_history_log`) que registra cada ação humana ou automática no registro.
- **Auto-Healing**: O sistema detecta a ausência de tabelas de produção e as cria automaticamente na primeira carga administrativa.

---
© 2026 Daora Kids Project.
