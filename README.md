# 3. Sistema de agendamento

## Objetivo

Criar uma API para reservar horários de recursos, como salas, equipamentos ou profissionais, impedindo que duas reservas confirmadas ocupem o mesmo intervalo.

**Nome planejado do repositório GitHub:** `sistema-agendamento`.

## Escopo sugerido

- Recursos com nome, fuso horário, duração mínima e períodos de funcionamento.
- Reservas com horário inicial e final, solicitante, estado e motivo de cancelamento.
- Consulta de disponibilidade por recurso e período.
- Estados: `PENDING`, `CONFIRMED`, `CANCELLED` e `EXPIRED`.
- Regras para cancelamento, prazo de confirmação e lista de espera.

## Conceitos de backend e banco

- Intervalos de data/hora e conversão entre fuso local e UTC.
- Transações e concorrência ao reservar o último horário livre.
- Restrição no banco que impeça sobreposição de reservas confirmadas.
- Paginação e filtros por recurso, estado e intervalo.
- Testes concorrentes e documentação OpenAPI.

## Modelo inicial

`resource(id, name, timezone, active)`; `reservation(id, resource_id, requester_id, starts_at, ends_at, status, created_at)`; `waitlist_entry(id, resource_id, requester_id, desired_start, desired_end, created_at)`.

## Critério de conclusão

Duas solicitações simultâneas para o mesmo intervalo não podem produzir duas reservas confirmadas. A API deve explicar o conflito com HTTP 409 e manter o banco consistente.

Esta pasta registra o escopo; o serviço ainda não foi implementado.

## Organização de pastas planejada

```text
sistema-agendamento/
├── docs/
│   ├── API.md
│   ├── ARQUITETURA.md
│   └── GUIA_DE_ESTUDO.md
├── src/main/java/br/com/portfolio/agendamento/
│   ├── resource/       # recursos e janelas de disponibilidade
│   ├── reservation/    # reservas e transições de estado
│   └── shared/         # erros e tipos comuns
├── src/main/resources/db/migration/
├── compose.yaml
├── pom.xml
└── README.md
```

Esta árvore é uma proposta para o projeto futuro, não uma lista de arquivos existentes.

## API proposta

| Método | Rota | Uso |
|---|---|---|
| `POST` | `/api/resources` | Criar recurso e fuso horário |
| `GET` | `/api/resources/{id}/availability` | Consultar horários livres |
| `POST` | `/api/reservations` | Criar reserva pendente |
| `GET` | `/api/reservations/{id}` | Consultar estado |
| `POST` | `/api/reservations/{id}/confirm` | Confirmar reserva |
| `POST` | `/api/reservations/{id}/cancel` | Cancelar e liberar intervalo |

## Ordem recomendada de implementação

1. Definir a semântica do intervalo (início inclusivo, fim exclusivo) e fuso por recurso.
2. Criar a migração SQL e a restrição de não sobreposição no PostgreSQL.
3. Implementar consulta de disponibilidade e criação de reserva transacional.
4. Adicionar expiração e lista de espera.
5. Demonstrar o conflito com duas solicitações concorrentes e resposta HTTP 409.
