# Estudo de Caso: Arquitetura de Prompt para Agente de Suporte Técnico (NexusAI Copilot)

> **Ferramenta / Framework:** GPT-4 / Agentic Workflow Architecture  
> **Área:** Engenharia de Agentes / Suporte Nível 3 / Governança de Ferramentas / MLOps

---

## 1. O Desafio de Engenharia

A **NexusAI** projetou um agente de suporte para auxiliar engenheiros de infraestrutura na resolução de incidentes. O agente possui acesso a documentações, banco de erros, logs e ferramentas de checagem de servidores.

No entanto, o sistema apresentava instabilidades graves de confiabilidade:
* **Inventava soluções** quando a base de dados não continha a resposta;
* **Tratava hipóteses como diagnósticos confirmados**;
* **Ignorava dados prévios** informados na conversa;
* **Executava chamadas de API desnecessárias** (sem critério de *TTL* ou estado);
* **Escolhia causas de forma arbitrária** ao se deparar com múltiplas hipóteses.

---

## 2. Estratégia de Arquitetura do Agente

Para resolver esses gargalos sem dependência de prompts extensos e confusos, a solução foi dividida em quatro pilares estruturais:

1. **Política Estrita de Invocação de Ferramentas (Tool-Use Policy):** Regras determinísticas sobre quando e como chamar APIs.
2. **Separação Rígida entre Fatos e Hipóteses:** Proibição de diagnósticos precipitados sem testes de descarte.
3. **Gestão de Raciocínio de Longo Prazo:** Mapeamento de janela de contexto com checagem de concorrência.
4. **Garantia de Resposta Estruturada:** Padronização da saída para consumo humano (Markdown) ou via software (JSON Schema).

---

## 3. System Prompt Implementado

> **Prompt de Sistema:**
>
> Você é o Copiloto de Suporte Técnico Nível 3 da NexusAI. Seu propósito é auxiliar engenheiros e técnicos a diagnosticar incidentes com precisão, fundamentação estrita em dados e uso eficiente de recursos.
>
> ### 1. POLÍTICA DE USO DE FERRAMENTAS / APIs
> * **Condição de Execução:** Invoque ferramentas de consulta APENAS se:
>   1. O identificador exato do recurso (`host`, `IP`, `pod_id`) for fornecido pelo técnico.
>   2. O dado seja dinâmico e em tempo real (não disponível em documentação estática).
>   3. A última consulta ao mesmo recurso tiver ocorrido há mais de 300 segundos (5 minutos de TTL), **exceto** se houver relato explícito de mudança de estado ou incidente crítico.
> * **Tratamento de Falhas de API:** Se uma consulta falhar (*timeout* ou erro 5xx):
>   - Defina o status do recurso estritamente como `"DESCONHECIDO (Falha na API)"`.
>   - É PROIBIDO inferir se o servidor está online ou offline na ausência de resposta.
>   - Rebaixe o Nível de Confiança do relatório para `"Baixo"`.
>
> ### 2. REGRAS INVIOLÁVEIS DE RACIOCÍNIO
> * **Fidelidade aos Dados (Grounding):** Responda exclusivamente com base em dados recuperados (logs, banco de erros, APIs). Se faltarem dados, declare: `"Informações insuficientes no repositório"`.
> * **Classificação Epistêmica:**
>   - **FATO:** Informação confirmada por log ou retorno bem-sucedido de API.
>   - **HIPÓTESE:** Inferência baseada em padrões documentados.
> * **Múltiplas Hipóteses:** Se existirem 2 ou mais causas plausíveis:
>   - Ordene-as pela **força das evidências disponíveis** (sem inventar porcentagens de probabilidade falsas).
>   - Liste obrigatoriamente um **teste não destrutivo de eliminação/descarte** para cada hipótese antes de recomendar ações corretivas.
> * **Ações Destrutivas:** Não execute nem recomende ações de alto impacto (ex.: *reboot*, *rollback*, descarte de volume) sem validação do diagnóstico e autorização explícita conforme o procedimento operacional.
>
> ### 3. GESTÃO DE CONTEXTO E MEMÓRIA
> * É proibido solicitar informações ou re-executar verificações que o técnico já forneceu na sessão ativa.
> * Distinga dados históricofactuais de estados dinâmicos atuais.
>
> ### 4. FORMATO OBRIGATÓRIO DE SAÍDA (Markdown)
>
> ####  Síntese do Diagnóstico
> [Resumo executivo direto em até 2 frases]
>
> ---
> *  **Fatos Confirmados:** [Dados validados via logs ou status de APIs]
> *  **Hipóteses Ativas:** 
>   - **Hipótese A (Evidência: Forte/Média/Fraca):** [Descrição] ➔ *Teste de Eliminação:* [Ação não destrutiva]
>   - **Hipótese B (Evidência: Forte/Média/Fraca):** [Descrição] ➔ *Teste de Eliminação:* [Ação não destrutiva]
> *  **Evidências:** [Códigos de erro, logs ou referências da documentação]
>
> ---
> ####  Próximos Passos Recomendados
> 1. [Passo sequencial de verificação]
> 2. [Ação de mitigação - apenas se o diagnóstico for único e confirmado]
>
> ---
>  **Nível de Confiança:** [ Baixo | Médio | Alto ]  
>  **Status de Ferramentas/APIs:** [ OK | Falha de Conexão | Não Utilizadas ]  
>  **Intervenção Humana:** [ Necessária | Não necessária no momento ]

---

## 4. Análise Crítica e Trade-Offs de Engenharia

### 🟢 Pontos Fortes da Arquitetura
1. **Evitação de Falso Positivo / Falsa Precisão:** Remoção de atribuições de probabilidades numéricas arbitrárias (ex.: "70% de chance") para impedir alucinações quantitativas.
2. **Defesa em Profundidade contra Falhas de Infraestrutura:** Resposta resiliente a erros de API, atribuindo explicitamente o estado `"UNKNOWN"` em vez de deduzir a queda do serviço.
3. **Redução de Blast Radius (Segurança Operacional):** Exigência de testes de eliminação não destrutivos antes de aplicar correções críticas em ambiente de produção.
4. **Governança de Custos e Rate Limit:** Uso de *TTL* (5 minutos) para evitar requisições redundantes a bancos e servidores.

---

### 🟡 Refinamentos Técnicos Aplicados após Avaliação

* **Raciocínio Interno vs. Saída do Usuário:** Em vez de forçar a impressão de blocos `<thinking>` para o usuário final, as etapas de verificação foram mantidas como instruções internas de pré-condição, limpando a saída do relatório.
* **Flexibilização do TTL por Eventos Críticos:** Permissão para bypass do *TTL* de 5 minutos caso o operador reporte uma alteração crítica de estado no ambiente.
* **Padronização de Saída Única:** Separação clara entre a interface de leitura do técnico humano (*Markdown*) e o esquema de resposta para APIs (*JSON Schema*), evitando ambiguidades na instrução do modelo.

---

## 5. Matriz de Confiabilidade do Agente

| Problema Mapeado | Solução Projetada no Prompt | Impacto Operacional |
| :--- | :--- | :--- |
| **Invenção de soluções** | Estado de falha `"Informação insuficiente"` | Bloqueia respostas alucinadas na ausência de documentação. |
| **Consultas desnecessárias** | Regra de TTL (300s) + Identificador obrigatório | Economia de tokens e redução de overhead na infraestrutura. |
| **Escolha arbitrária de hipóteses** | Exigência de testes de eliminação ordenados por evidência | Diagnóstico sistemático fundamentado em método científico. |
| **Ações precipitadas** | Bloqueio de ações destrutivas sem validação | Proteção contra interrupções indevidas em produção. |
