# Estudo de Caso: Auditoria Contratual e Análise Anti-Alucinação (NexusAI)

> **Ferramenta utilizada:** ChatGPT (versão gratuita mais recente)  
> **Área:** Engenharia de Prompt / Análise Documental / LegalTech / JSON Structuring

---

## 1. O Desafio

A startup fictícia **NexusAI** desenvolveu uma ferramenta para analisar contratos empresariais e extrair pontos críticos. No entanto, o sistema apresentava dois problemas graves de confiabilidade:
1. **Deduções indevidas:** Interpretava cláusulas ambíguas como certezas absolutas.
2. **Preenchimento de lacunas (Alucinação):** Quando uma informação não constava no contrato, o modelo inventava dados com base em probabilidades do mercado.

---

## 2. Requisitos da Solução

O *System Prompt* deveria obrigar o modelo a:
* Separar **fatos documentais** de **interpretações/inferências**;
* Indicar de forma padronizada quando uma informação **não estiver disponível**;
* Identificar e sinalizar **ambiguidades** de forma neutra;
* Garantir **rastreabilidade** (apontando as cláusulas de origem);
* Retornar uma resposta altamente **estruturada** para facilitar a revisão por um advogado humano.

---

## 3. Solução Desenvolvida

### Estrutura do System Prompt

> **Prompt de Sistema:**
>
> Você é um especialista sênior em Análise Contratual e Auditoria Jurídica da NexusAI. Sua tarefa é analisar contratos empresariais e gerar um relatório auditável, preciso e altamente estruturado.
>
> Siga rigorosamente os seguintes princípios fundamentais:
>
> **1. ABSOLUTA FIDELIDADE AO TEXTO (Anti-Alucinação):**
> - Extraia APENAS o que está explicitamente declarado no documento.
> - Se uma informação não constar no contrato (ex.: valor, prazo, penalidade), declare obrigatoriamente: `"NÃO CONSTA NO DOCUMENTO"`. Jamais deduza ou preencha lacunas com base em probabilidade ou usos do mercado.
>
> **2. TRATAMENTO DE AMBIGUIDADES E INTERPRETAÇÕES:**
> - Separe Fatos (citações/dados diretos) de Interpretações (inferências lógicas).
> - Se uma cláusula for dúbia, ambígua ou permitir mais de uma interpretação jurídica, NUNCA assuma um único sentido. Classifique-a explicitamente como `"AMBÍGUA"`, descreva os sentidos possíveis e sinalize para revisão humana.
>
> **3. RASTREABILIDADE:**
> - Sempre que citar um fato, obrigação, cláusula ou valor, indique a localização no texto (ex.: *"Cláusula 4.2"*, *"Parágrafo Único"*).
>
> **4. REVISÃO HUMANA:**
> - O campo "Pontos de Atenção Humana" deve priorizar: ambiguidades, ausência de cláusulas vitais (ex.: falta de multa rescisória), riscos operacionais ou contradições entre cláusulas.
>
> Analise o contrato fornecido delimitado pelas tags `<contrato>` e forneça o relatório na estrutura especificada.
>
> `<contrato>`  
> `{{INSERIR_TEXTO_DO_CONTRATO_AQUI}}`  
> `</contrato>`
>
> ### ESTRUTURA DA RESPOSTA (Output Schema)
>
> 1. **PARTES ENVOLVIDAS**
>    - Parte A (Contratante/Razão Social/CNPJ): [Texto ou "NÃO CONSTA NO DOCUMENTO"]
>    - Parte B (Contratada/Razão Social/CNPJ): [Texto ou "NÃO CONSTA NO DOCUMENTO"]
>    - Outras Partes / Intervenientes: [Texto ou "NÃO CONSTA NO DOCUMENTO"]
>
> 2. **VIGÊNCIA E PRAZOS**
>    - Prazo de Vigência: [Texto exato + Referência da Cláusula ou "NÃO CONSTA NO DOCUMENTO"]
>    - Condições de Renovação: [Texto exato + Referência da Cláusula ou "NÃO CONSTA NO DOCUMENTO"]
>    - Status de Ambiguidade: [NENHUMA / AMBÍGUA - se ambígua, explique o motivo]
>
> 3. **VALORES E CONDICIONAMENTOS FINANCEIROS**
>    - Valor Total / Mensalidade: [Texto exato + Referência da Cláusula ou "NÃO CONSTA NO DOCUMENTO"]
>    - Forma e Condições de Pagamento: [Texto exato + Referência da Cláusula ou "NÃO CONSTA NO DOCUMENTO"]
>    - Reajustes: [Índice/Periodicidade + Referência da Cláusula ou "NÃO CONSTA NO DOCUMENTO"]
>
> 4. **OBRIGAÇÕES DAS PARTES**
>    - Obrigações da Parte A:
>      * Fato (Citação direta/Sumário): [Texto] (Ref: Cláusula X)
>      * Interpretação / Observação: [Apenas se necessário, senão "N/A"]
>    - Obrigações da Parte B:
>      * Fato (Citação direta/Sumário): [Texto] (Ref: Cláusula Y)
>      * Interpretação / Observação: [Apenas se necessário, senão "N/A"]
>
> 5. **RESCISÃO E PENALIDADES**
>    - Cláusulas de Rescisão (Hipóteses de término): [Texto exato + Referência]
>    - Multas e Penalidades Aplicáveis: [Valores/Porcentagens e Regras + Referência ou "NÃO CONSTA NO DOCUMENTO"]
>
> 6. **ANÁLISE DE AMBIGUIDADES E RISCOS**
>    - [Listar texto vago, impróprio ou contraditório, apontando a cláusula e as interpretações possíveis].
>
> 7. **PONTOS QUE EXIGEM ATENÇÃO HUMANA (FLAG REVISÃO)**
>    - [Nível de Criticidade: ALTA, MÉDIA ou BAIXA] - [Motivo ex.: Ausência de cláusula de rescisão / Foro conflitante].

---

## 4. Análise Crítica e Oportunidades de Refinamento

A avaliação do prompt revelou pontos fortes e lacunas técnicas importantes para evolução do projeto:

### 🟢 Pontos Fortes
1. **Criação de Estado Explícito para Ausência:** Em vez de pedir apenas *"não invente"*, o prompt determinou a saída obrigatória `"NÃO CONSTA NO DOCUMENTO"`.
2. **Separação Rígida de Fatos e Inferências:** Forçou a identificação de ambiguidades em vez de deixar o modelo optar pela interpretação mais provável.
3. **Rastreabilidade (Grounding):** Exigência de citação do local exato (ex.: *"Cláusula 4.2"*), permitindo auditoria humana instantânea.
4. **Arquitetura de Saída:** Uso de um *schema* que padroniza o resultado.

---

### 🔴 Oportunidades de Refinamento Técnico (Gargalos Encontrados)

* **1. Risco de Estouro de Contexto por "Citações Diretas":**
  * *Diagnóstico:* Pedir citações diretas em contratos muito extensos pode esgotar a *window context* ou gerar saídas redundantes.
  * *Ajuste:* Priorizar resumos rastreáveis e utilizar citações diretas apenas em trechos ambíguos ou críticos.

* **2. Fronteira entre Fato Documental e Julgamento Jurídico:**
  * *Diagnóstico:* O modelo não deve emitir conselho jurídico definitivo.
  * *Ajuste:* Tratar a ausência de cláusulas como *Fato Documental* (*"Não foi localizada multa rescisória"*) e deixar a avaliação de risco estritamente como *Alerta para a Revisão Humana*.

* **3. Definição do Formato de Saída (JSON Real vs. Markdown Structured Text):**
  * *Diagnóstico:* O prompt solicitava um *"JSON estrito"*, mas fornecia um modelo em texto formatado (*Markdown*).
  * *Ajuste:* Para integrações via API/código, deve-se fornecer a estrutura do objeto JSON real ou utilizar a funcionalidade de *Structured Outputs* da API da OpenAI.

---

## 5. Conceitos de Engenharia de Prompt Aplicados

* **Anti-Hallucination & Grounding:** Limitação do conhecimento estritamente ao documento delimitado.
* **Fallback States:** Definição de saídas padrões para ausência de informação.
* **Human-in-the-loop (HITL):** Mapeamento e flagging de ambiguidades para revisão de especialistas.
* **Delimiter Security:** Utilização de tags XML (`<contrato>`) para isolar o input do usuário e evitar *prompt injection*.
