# Sistema de Automação de Prospecção, Qualificação e Gestão de Leads com IA

## Visão Geral

Este sistema é uma arquitetura de automação comercial desenvolvida para operar de forma contínua em ambiente self-hosted, integrando captura de leads locais, processamento inteligente de dados, qualificação automatizada, gestão de agendamentos de reuniões e automação de comunicação via WhatsApp.

Toda a solução funciona de maneira integrada, unificando três fluxos principais:

- Geração automática de leads via Google Maps  
- Geração manual de leads via formulário e busca dirigida  
- Gestão de reuniões e atualização de status via Calendly  
- Agente de WhatsApp para comunicação e atualização de leads  

O objetivo central é transformar dados públicos e eventos comerciais em um pipeline estruturado de oportunidades qualificadas prontas para abordagem ou atendimento humano.

---

## Objetivo do Sistema

Automatizar o ciclo completo de prospecção e atendimento inicial de leads, incluindo:

- Descoberta de empresas locais  
- Enriquecimento e normalização de dados  
- Qualificação automática com base em critérios comerciais  
- Registro centralizado em Google Sheets  
- Atualização de status baseada em eventos de agendamento  
- Automação de conversas via WhatsApp  
- Transferência para atendimento humano quando necessário  

---

## Arquitetura Geral

O sistema é dividido em quatro fluxos integrados:

                  ┌─────────────────────────────────────────────────┐
                  │           Google Sheets (CRM Leve)              │
                  │         planilha: leads_maps                    │
                  │  3 abas: Leads | eventos | (config)            │
                  └──────────┬──────────────┬──────────────────────┘
                             │              │
              ┌──────────────┘              └──────────────┐
              ▼                                            ▼
┌─────────────────────────┐              ┌─────────────────────────┐
│   PROJETO 1             │              │   PROJETO 2             │
│   Gerador de Leads      │              │   CRM + Disparo         │
│   (Google Maps + IA)    │              │   (Calendly + Sheets)   │
│                         │              │                         │
│  ⏰ Agendado (50 min)   │              │  ⏰ WF2: Outbound (1h)  │
│  📋 Manual (form)       │              │  📥 WF3: Inbound (cont) │
│  🧹 Limpeza + Score     │              │  🔄 WF4: Follow-up (2h) │
│  🤖 IA SDR (insights)   │              │  📅 Calendly Trigger    │
└─────────────┬───────────┘              └─────────────┬───────────┘
              │                                        │
              │                                        │
              │              ┌─────────────────┐        │
              │              │   PROJETO 3     │        │
              │              │   Agente WhatsApp│        │
              └──────────────►   (IA Local)    ◄────────┘
                             │                 │
                             │  🧠 Ollama      │
                             │  Llama 3.2 3B   │
                             │  💬 Memória     │
                             │  🔧 Parser JSON │
                             └─────────────────┘
---

### Fluxo 1 — Aquisição Automática de Leads (Google Maps + IA)

- Execução agendada  
- Rotação de nichos e cidades  
- Geração de queries com IA  
- Captura de dados via Google Maps (HasData)  
- Normalização e limpeza  
- Score de qualificação  
- Geração de insights comerciais com IA  
- Persistência em Google Sheets  

---

### Aquisição Manual de Leads (Formulário Web)

- Entrada manual de parâmetros:
  - Nicho  
  - Cidade / Bairro  
  - Limite de resultados  

- Execução de busca direcionada no Google Maps  
- Reutilização do pipeline de processamento:
  - Limpeza de dados  
  - Normalização  
  - Score de qualificação  
  - Geração de insights via IA  

- Armazenamento unificado em Google Sheets  
- Deduplicação baseada em telefone  

---

###  Fluxo 2 — Gestão de Reuniões (Calendly + Atualização de CRM)

#### Trigger de Evento

O fluxo é iniciado quando ocorre o evento:

- `invitee.created` no Calendly  

---

### Processamento de Dados (Code Node)

Os dados recebidos são normalizados e estruturados:

- Extração de payload do Calendly  
- Captura de:
  - Email do participante  
  - Nome do participante  
  - Telefone via tracking (UTM)  
  - Horário da reunião  

- Padronização de email (lowercase e trim)  
- Definição de status inicial: `REUNIAO`  

Saída estruturada:

- telefone_normalizado  
- email  
- nome  
- horario  
- status_novo  

---

### Consulta de Lead Existente

O sistema realiza busca no Google Sheets:

- Base: planilha central de leads  
- Aba: Leads  
- Chave de busca: `telefone_normalizado`  

Objetivo:

- Identificar registro existente do lead  
- Garantir consistência de dados  
- Evitar duplicação de registros  

---

### Atualização do Registro

Após localizar o lead, o sistema atualiza:

- status: `REUNIAO`  
- telefone_normalizado (chave de controle)  
- empresa (se existente no registro anterior)  
- human_handoff: `TRUE`  

---

### (WAHA + Automação + CRM)

(WF2, WF3, WF4) é responsável pela comunicação ativa e reativa com leads através do WhatsApp, integrando automação, IA e atualização de CRM.

#### Entrada de Mensagens

O sistema utiliza o WAHA (WhatsApp HTTP API) para capturar:

- Mensagens recebidas de leads  
- Eventos de interação (respostas, confirmações, dúvidas)  
- Identificação do número do contato  

---

#### Processamento da Mensagem

Ao receber uma mensagem:

- O número do lead é normalizado  
- O sistema consulta o Google Sheets usando `telefone_normalizado`  
- O histórico do lead é recuperado  
- A mensagem é classificada (interesse, dúvida, objeção ou agendamento)  

---

#### Interação com IA

A IA é utilizada para:

- Gerar respostas automáticas contextualizadas  
- Identificar intenção do lead  
- Sugerir próximos passos comerciais  
- Adaptar comunicação conforme estágio do funil  

---

#### Atualização do CRM

Após cada interação:

- last_message_at é atualizado  
- followup_count é incrementado  
- intencao pode ser ajustada  
- status pode evoluir conforme resposta do lead  

Exemplos de evolução:

- Novo lead → Engajado  
- Engajado → Qualificado  
- Qualificado → Reunião  
- Sem resposta → Follow-up automático  

---

#### Transferência para Humano

O sistema ativa `human_handoff = TRUE` quando:

- Lead demonstra alta intenção  
- Solicita atendimento humano  
- Falha em automações repetidas  
- Chega ao estágio de fechamento  

---

## Modelo de Dados (Google Sheets)

O sistema utiliza uma base centralizada com os seguintes campos:

- empresa  
- telefone  
- telefone_normalizado  
- email  
- endereco  
- website  
- categoria  
- nicho  
- cidade  
- bairro  
- score  
- prioridade  
- status  
- data_captura  
- last_message_at  
- created_at  
- updated_at  
- human_handoff  
- next_followup_at  
- vsl_url  
- intencao  
- followup_count  

---

## Regras de Processamento

### Deduplicação

- Chave primária: `telefone_normalizado`  
- Evita duplicação de leads  
- Atualização sempre preferida à inserção redundante  

---

### Qualificação de Leads

Cada lead recebe um score baseado em:

- Existência de WhatsApp  
- Presença de website  
- Avaliação pública  
- Número de reviews  
- Potencial do nicho  

Classificação final:

- Alta prioridade  
- Média prioridade  
- Baixa prioridade  

---

### Enriquecimento com IA

A IA gera automaticamente insights comerciais com foco em:

- Problemas operacionais prováveis  
- Gargalos de atendimento  
- Oportunidades de automação  

Exemplo de saída:

- "Demora no atendimento via WhatsApp pode estar reduzindo conversões"  

---

### Gestão de Status

O status do lead evolui conforme eventos:

- Captura inicial → novo lead  
- Qualificação → lead estruturado  
- Agendamento → REUNIAO  
- Interação WhatsApp → engajamento  
- Transferência → human_handoff = TRUE  

---
### Projeto 3 — Agente WhatsApp com IA Local (WAHA + n8n + Ollama)
Arquivo: workflows/agente_whatsapp_ollama.json

Visão Geral:
Agente conversacional 100% offline rodando IA local (Llama 3.2 3B via Ollama), com memória por usuário e envio anti-spam.

Pipeline do Agente
kotlin
🌐 WAHA Trigger: message.any  
       │  
       ▼  
🔄 Normalize Event  
  • Extrai: from, body, fromMe, notifyName  
  • Garante formato @c.us  
       │  
       ▼  
🔻 Filtrar Mensagens  
  • Ignora vazias  
  • Ignora FROM_ME (eco do próprio bot)  
  • Ignora mídia sem texto  
       │  
       ▼  
✏️ Edit Fields (prepara payload)  
  • session, data.from, data.message, data.nome  
  • data.primeiro_nome, data.hasMedia = false  
       │  
       ▼  
🧠 AI Agent (Ollama — Llama 3.2 3B)  
  ┌─────────────────────────────────────────┐  
  │ Model:        Llama 3.2 3B              │  
  │ Temperature:  0.3 (baixa = respostas     │  
  │               consistentes)              │  
  │ System Prompt: "Você é atendente de      │  
  │ WhatsApp.                                │ 
  │ WhatsApp. Responda curto, direto e       │
  │ amigável."                               │
  │                                          │
  │ Formato saída obrigatório:               │
  │ {"paragraphs":["resposta curta"]}        │
  └─────────────────────────────────────────┘
       │
       ▼
🧠 Simple Memory (Buffer Window)
  • Chave: session_{data}_{from}
  • Mantém últimas 10 interações
  • Contexto contínuo por usuário
       │
       ▼
🔧 Parsear Resposta (robusto)
  • Remove aspas quebradas no final do JSON
  • Tenta JSON.parse()
  • Fallback 1: regex extrai texto bruto
  • Fallback 2: "Desculpe, pode repetir? 😊"
       │
       ▼
🔄 Loop por Parágrafos
  • Divide resposta em blocos menores
       │
       ▼
⏳ Delay Anti-Spam: 3s entre cada parágrafo
       │
       ▼
📤 WAHA: Enviar Mensagem
  • chatId, paragraph, session
  
## Características Técnicas
# Característica	|  Detalhe
Modelo de IA	      Llama 3.2 3B (Ollama local)
Temperatura	        0.3 (baixa — respostas consistentes)
Memória	            Buffer de janela (10 turnos por usuário)
Parser              JSON	Três níveis de fallback
Anti-spam	          3s de delay entre mensagens
Formato saída	      {"paragraphs": ["resposta"]}
Alternativa	        OpenRouter (Google Gemma 3) disponível mas desativado

💡 Dica: O agente é propositalmente simples e direto — sem RAG, sem ferramentas externas. Ele atua como primeiro atendimento, qualificando leads e transferindo para humano quando necessário (via human_handoff).

## ⚙️ Stack Técnica

Ferramenta	    Versão	                    Função	                    Custo
n8n	            ≥ 1.0	                      Orquestrador low-code	      Gratuito (self-hosted)
Ollama	        ≥ 0.3	                      IA local (LLM)	            Gratuito
WAHA	          ≥ 2.0	                      API WhatsApp	              Gratuito (self-hosted)
Google Sheets	    —	                        CRM leve / banco de dados	  Gratuito
HasData	        API v2	                    Scraping Google Maps	      Gratuito (100 créditos/mês)
OpenRouter	      —	                        IA cloud (fallback SDR)	    Gratuito (modelos free)
Calendly	        —	                        Agendamento de reuniões	     Gratuito (básico)


### Diagrama de Conexões

┌─────────┐     ┌──────────┐     ┌──────────┐
│ HasData │────▶│   n8n    │────▶│  Ollama  │
│  (API)  │     │  (core)  │     │  (local) │
└─────────┘     └────┬─────┘     └──────────┘
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │ Google  │ │  WAHA   │ │Calendly │
    │ Sheets  │ │WhatsApp │ │Webhook  │
    └─────────┘ └─────────┘ └─────────┘
---

## Infraestrutura de Execução

### Ambiente Virtual

- Hypervisor: VMware  
- Sistema Operacional: Ubuntu Server 22.04.5 LTS (Jammy)  
- Virtualização: VMware full virtualization  
- Rede: NAT  

---

### Hardware da Máquina

- CPU: Intel Core i5-4200U @ 1.60GHz  
- vCPUs: 2  
- RAM: 4.7 GB  
- Armazenamento: 80 GB (77 GB utilizáveis)  
- Swap: 3.8 GB  
- Arquitetura: x86_64  

---

### Stack de Containers (Docker)

- Docker Engine: 29.1.3  
- Docker Compose: v2.27.0  

Containers ativos:

- n8n (1.121.0) — Orquestração de workflows  
- Ollama (0.3.14) — Execução local de modelos de IA  
- WAHA — Integração WhatsApp HTTP API  

---

## Características Técnicas do Sistema

- Execução contínua em ambiente self-hosted  
- Processamento distribuído por workflows  
- Integração entre APIs externas e IA local  
- Pipeline unificado de dados comerciais  
- Automação de comunicação via WhatsApp  
- Baixo consumo de recursos computacionais  
- Atualização incremental de registros  
- Arquitetura orientada a eventos  

---

## Resultado Final do Sistema

O sistema opera como uma infraestrutura completa de:

- Geração de leads automatizada  
- Qualificação inteligente de contatos  
- Centralização de dados comerciais  
- Automação de conversas via WhatsApp  
- Agendamento e atualização de status em tempo real  
- Integração entre automação, IA e CRM leve (Google Sheets)  

---

## Resumo Técnico

Este ambiente combina automação com n8n, IA local via Ollama e integração com WhatsApp para criar um fluxo contínuo de prospecção, qualificação e gestão de leads, operando integralmente em uma única máquina virtual com recursos limitados, mas suficiente para execução estável de múltiplos serviços simultâneos.
