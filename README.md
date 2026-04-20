 📊 CRM Case | Análise e Estratégia de Leads

Projeto desenvolvido com foco em identificar leads qualificados e propor estratégias de conversão e retenção com base em dados de CRM.
Todos os dados utilizados são ficticios apenas com a finalidade de estudos!

🎯 Objetivo

Identificar leads com maior propensão à conversão utilizando critérios de origem, engajamento e comportamento recente, além de propor segmentações e estratégias de comunicação orientadas a dados.

🧠 Definição de Leads Qualificados

A query foi desenvolvida para identificar leads qualificados com base em critérios de origem, engajamento e comportamento recente.

Foram considerados:
- Origem: Google Ads e tráfego orgânico  
- Pontuação de engajamento > 50  
- Status de inscrição: "Pendente"  

Para recência, utilizei a data mais recente da base como referência (`MAX(data)`), filtrando leads com interação entre 7 e 30 dias.

Também foi considerado um nível mínimo de interesse:
- Pelo menos 2 interações nos últimos 60 dias  

Critérios adicionais:
- Leads que já receberam campanhas  
- Exclusão de leads descadastrados nos últimos 30 dias  

Leads foram segmentados por propensão à conversão com base em engajamento e comportamento, direcionando estratégias de conversão ou nutrição. Foi estruturado um fluxo de reativação baseado em objeções, com acompanhamento por KPIs de funil e análise de LIFT. A análise Cohort apoiou estratégias de onboarding e retenção.

 📊 Impacto da Análise

A abordagem permite:
- Priorização de leads com maior probabilidade de conversão  
- Redução de esforço em leads desqualificados  
- Otimização de campanhas com base em comportamento real  

Mesmo com LIFT negativo, a análise permitiu identificar falhas de segmentação e oportunidades de melhoria.

> ⚠️ Observação: Devido ao baixo volume da base (~100 leads) e à combinação dos filtros, o resultado pode ser reduzido ou inexistente.


## 💻 Query SQL

```sql
SELECT 
    l.id_lead,
    l.origem,
    l.status_inscricao,
    l.pontuacao_engajamento
FROM leads l

WHERE 
    l.origem IN ('Google Ads', 'Orgânico')
    AND l.pontuacao_engajamento > 50

    AND l.ultima_interacao BETWEEN 
        DATE_SUB(
            (SELECT MAX(ultima_interacao) FROM leads), 
            INTERVAL 30 DAY
        )
        AND DATE_SUB(
            (SELECT MAX(ultima_interacao) FROM leads), 
            INTERVAL 7 DAY
        )

    AND l.status_inscricao = 'Pendente'

    AND EXISTS (
        SELECT 1
        FROM campanhas c
        WHERE c.id_lead = l.id_lead
    )

    AND (
        SELECT COUNT(*)
        FROM interacao i
        WHERE i.id_lead = l.id_lead
          AND i.data_interacao >= DATE_SUB(
              (SELECT MAX(data_interacao) FROM interacao),
              INTERVAL 60 DAY
          )
    ) >= 2

    AND NOT EXISTS (
        SELECT 1
        FROM campanhas c2
        WHERE c2.id_lead = l.id_lead
          AND c2.status = 'Descadastrou'
          AND c2.data_envio >= DATE_SUB(
              (SELECT MAX(data_envio) FROM campanhas),
              INTERVAL 30 DAY
          )
    );
