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

O sistema é dividido em três fluxos integrados:

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

### Fluxo 2 — Aquisição Manual de Leads (Formulário Web)

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

### Fluxo 3 — Gestão de Reuniões (Calendly + Atualização de CRM)

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
- Transferência → human_handoff = TRUE

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
- Baixo consumo de recursos computacionais
- Atualização incremental de registros
- Arquitetura orientada a eventos

---

## Resultado Final do Sistema

O sistema opera como uma infraestrutura completa de:

- Geração de leads automatizada
- Qualificação inteligente de contatos
- Centralização de dados comerciais
- Agendamento e atualização de status em tempo real
- Integração entre automação, IA e CRM leve (Google Sheets)

---

## Resumo Técnico

Este ambiente combina automação com n8n, IA local via Ollama e integração com WhatsApp para criar um fluxo contínuo de prospecção e gestão de leads, operando integralmente em uma única máquina virtual com recursos limitados, mas suficiente para execução estável de múltiplos serviços simultâneos.
