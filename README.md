# Sistema de Automação de Prospecção, Qualificação e Gestão de Leads com IA

## Visão Geral

Este sistema é uma arquitetura de automação comercial desenvolvida para operar de forma contínua em ambiente self-hosted, integrando captura de leads locais, processamento inteligente de dados, qualificação automatizada e gestão de agendamentos de reuniões.

Toda a solução funciona de maneira integrada, unificando três fluxos principais:

- Geração automática de leads via Google Maps
- Geração manual de leads via formulário e busca dirigida
- Gestão de reuniões e atualização de status via Calendly

O objetivo central é transformar dados públicos e eventos comerciais em um pipeline estruturado de oportunidades qualificadas prontas para abordagem ou atendimento humano.

---

## Objetivo do Sistema

Automatizar o ciclo completo de prospecção e atendimento inicial de leads, incluindo:

- Descoberta de empresas locais
- Enriquecimento e normalização de dados
- Qualificação automática com base em critérios comerciais
- Registro centralizado em Google Sheets
- Atualização de status baseada em eventos de agendamento
- Transferência para atendimento humano quando necessário

---

## Arquitetura Geral

### Diagrama do Sistema

```mermaid
flowchart TD

A[Fluxo Automático - Google Maps + IA] --> D[Google Sheets Base Central]
B[Fluxo Manual - Formulário Web] --> D
C[Calendly Trigger - Reuniões] --> E[Code Node - Normalização]

E --> F[Busca Lead no Sheets]
F --> D

D --> G[Qualificação e Score IA]
D --> H[Atualização de CRM]

Fluxos do Sistema
Fluxo 1 — Aquisição Automática de Leads (Google Maps + IA)
Execução agendada
Rotação de nichos e cidades
Geração de queries com IA
Captura de dados via Google Maps (HasData)
Normalização e limpeza
Score de qualificação
Geração de insights comerciais com IA
Persistência em Google Sheets
Fluxo 2 — Aquisição Manual de Leads (Formulário Web)
Entrada manual de parâmetros:
Nicho
Cidade / Bairro
Limite de resultados
Execução de busca direcionada no Google Maps
Reutilização do pipeline:
Limpeza de dados
Normalização
Score de qualificação
Geração de insights via IA
Armazenamento unificado em Google Sheets
Deduplicação baseada em telefone
Fluxo 3 — Gestão de Reuniões (Calendly + CRM)
Trigger de Evento
Evento: invitee.created (Calendly)
Processamento de Dados

O sistema extrai e normaliza:

Email do lead
Nome do participante
Telefone (via tracking UTM)
Horário da reunião

Saída estruturada:

telefone_normalizado
email
nome
horario
status_novo = REUNIAO
Consulta de Lead
Busca no Google Sheets
Base central de leads
Chave: telefone_normalizado

Objetivo:

Identificar registro existente
Evitar duplicação
Garantir consistência de dados
Atualização de Registro

Campos atualizados:

status = REUNIAO
telefone_normalizado
empresa (se existir)
human_handoff = TRUE
Modelo de Dados (Google Sheets)

Campos utilizados:

empresa
telefone
telefone_normalizado
email
endereco
website
categoria
nicho
cidade
bairro
score
prioridade
status
data_captura
created_at
updated_at
last_message_at
human_handoff
next_followup_at
followup_count
intencao
Regras de Processamento
Deduplicação
Chave primária: telefone_normalizado
Atualização sempre preferida à inserção duplicada
Qualificação de Leads

Critérios utilizados:

WhatsApp disponível
Website ativo
Avaliação pública
Número de reviews
Potencial do nicho

Classificação:

Alta prioridade
Média prioridade
Baixa prioridade
Enriquecimento com IA

A IA gera insights comerciais como:

Problemas operacionais prováveis
Gargalos de atendimento
Oportunidades de automação

Exemplo:

"Demora no atendimento via WhatsApp pode estar reduzindo conversões"

Gestão de Status
Novo lead → captura inicial
Qualificado → scoring aplicado
REUNIAO → Calendly confirmado
human_handoff → atendimento humano
Infraestrutura de Execução
Ambiente Virtual
Hypervisor: VMware
Sistema Operacional: Ubuntu Server 22.04.5 LTS
Rede: NAT
Hardware
CPU: Intel i5-4200U @ 1.60GHz
vCPUs: 2
RAM: 4.7 GB
Disco: 80 GB
Swap: 3.8 GB
Stack Docker
Docker Engine: 29.1.3
Docker Compose: v2.27.0

Containers:

n8n (1.121.0)
Ollama (0.3.14)
WAHA (WhatsApp API)
Características Técnicas
Execução contínua em self-hosted
Integração entre APIs e IA local
Pipeline unificado de dados comerciais
Baixo consumo de recursos
Atualização incremental de registros
Arquitetura orientada a eventos
Resultado Final

O sistema funciona como uma infraestrutura completa de:

Geração automática de leads
Qualificação inteligente
Centralização de dados comerciais
Automação de agendamentos
Atualização de CRM leve em tempo real
Resumo Técnico

Sistema construído com n8n, IA local (Ollama) e integração com WhatsApp, operando em ambiente virtualizado com recursos limitados, mas suficiente para execução contínua de automações comerciais complexas.
