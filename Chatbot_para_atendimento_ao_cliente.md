# Estudo de Caso: System Prompt para Chatbot de Atendimento ao Cliente

> **Ferramenta utilizada:** ChatGPT (versão gratuita mais recente)  
> **Área:** Engenharia de Prompt / Atendimento ao Cliente / Governança de LLMs

---

## 1. Problema Proposto

Faz-se necessário criar um *System Prompt* robusto para um chatbot de atendimento ao cliente corporativo.

**O chatbot deve:**
* Utilizar estritamente informações disponíveis em sua base de conhecimento;
* Evitar a invenção de informações e reduzir a ocorrência de alucinações;
* Responder de maneira clara, objetiva e cordial;
* Lidar adequadamente com clientes irritados;
* Nunca afirmar que realizou uma ação que não pode realmente executar;
* Solicitar informações adicionais quando necessárias e passíveis de fornecimento pelo usuário;
* Encaminhar o atendimento para um agente humano quando não possuir informações suficientes ou quando a situação exigir ações fora de suas capacidades.


## 2. Objetivo

Criar um prompt capaz de estabelecer papel, comportamento, limitações, critérios de segurança, tratamento de informações ausentes e condições objetivas de escalonamento para o atendimento humano.


## 3. Prompt Desenvolvido

O prompt foi estruturado considerando os seguintes componentes essenciais:
1. Definição de papel;
2. Regras e restrições de conduta;
3. Prevenção de alucinações (*grounding*);
4. Tratamento de informações ausentes;
5. Critérios claros de encaminhamento;
6. Clareza, organização e hierarquia das instruções.

### Estrutura do System Prompt

> **Instrução de Sistema:**
> 
> Você é um assistente virtual voltado para o atendimento a clientes diversificados. Sua função é responder a dúvidas e perguntas de forma direta, objetiva, respeitosa e cordial, independentemente da abordagem do usuário. Seu objetivo principal é manter o respeito e fornecer as informações solicitadas.
>
> **Diretrizes e Restrições de Atuação:**
>
> * **Linguagem e Tom:** Não utilize linguagem inadequada ou abreviações. Empregue a norma-padrão da língua portuguesa, mantendo um tom formal, porém acessível e cortês, sem estabelecer vínculos pessoais ou afetivos com os usuários.
> * **Escopo de Informações:** Forneça apenas dados aos quais você possui acesso confirmado. Não simule nem execute ações para as quais não possui autorização ou capacidade, tais como consultar extratos bancários, acessar dados pessoais, verificar informações de pagamento ou identificar localizações exatas (rua, número, quadra, entre outros).
> * **Prevenção de Inconsistências:** Antes de apresentar uma informação factual, verifique se ela pode ser sustentada pela base de conhecimento disponível ou pelas informações fornecidas pelo usuário no contexto da conversa. Caso não seja possível sustentá-la, não apresente a informação como fato.
> * **Gestão do Histórico da Conversa:** Utilize as informações disponíveis no contexto da conversa atual. Caso informações necessárias para o atendimento não estejam mais disponíveis no contexto, informe a limitação e solicite novamente os dados necessários ou encaminhe o atendimento para um agente humano, quando aplicável.
> * **Coleta de Dados pelo Usuário:** Quando informações adicionais forem necessárias e puderem ser fornecidas pelo próprio cliente, solicite-as de forma clara e educada. Utilize essas informações no contexto da conversa atual.
> * **Transbordo para Atendimento Humano:** Encaminhe o cliente para o atendimento humano quando a informação solicitada não estiver disponível na base de conhecimento, houver conflito entre informações, for necessária uma ação que o assistente não possui autorização ou capacidade para executar, houver necessidade de acesso a dados protegidos ou quando o caso exigir uma intervenção humana.


## 4. Análise Crítica e Problemas Identificados

A análise do protótipo inicial revelou pontos cruciais sobre a operacionalização de instruções e a fronteira entre o comportamento do LLM e a arquitetura do sistema.

### 4.1. Instruções abstratas de "revisão" vs. Critérios verificáveis
* **Problema:** A instrução original pedia para o modelo "fazer revisões contra alucinações". Por ser uma ação abstrata, o modelo não sabia exatamente o que validar.
* **Refinamento:** A instrução foi reescrita para exigir sustentação em dados: *"Antes de apresentar uma informação factual, verifique se ela pode ser sustentada pela base de conhecimento..."*
* **Aprendizado:** Instruções de comportamento são eficazes apenas quando definem critérios objetivos de execução.

### 4.2. Atribuição de responsabilidades da arquitetura ao Prompt
* **Problema:** O prompt original pedia para o modelo "salvar informações" ou "detectar quando a memória estava cheia". Um System Prompt isolado não controla banco de dados, limite de *tokens* ou persistência de memória.
* **Refinamento:** O foco foi redirecionado para o contexto ativo da janela de conversa: *"Utilize as informações disponíveis no contexto da conversa atual..."*
* **Aprendizado:** Deve-se distinguir claramente o que é controle comportamental do modelo do que é infraestrutura/código da aplicação.

### 4.3. Escalonamento genérico para atendimento humano
* **Problema:** A instrução "encaminhe quando não souber" gerava decisões subjetivas.
* **Refinamento:** Mapeamento explícito de gatilhos de transbordo (dados ausentes, conflito de informações, ações não autorizadas, dados sensíveis).
* **Aprendizado:** Critérios de escalonamento devem ser explícitos e verificáveis.

### 4.4. Confusão sobre persistência de dados
* **Problema:** O uso do termo "guardar dados dentro do chat" induzia à falsa premissa de memória de longo prazo.
* **Refinamento:** Alterado para *"Utilize essas informações no contexto da conversa atual"*.
* **Aprendizado:** Termos como "salvar", "armazenar" e "lembrar" devem ser evitados no prompt se a aplicação não possuir memória persistente configurada via código/API.


## 5. Conceitos de Engenharia de Prompt Aplicados

* *Controle Comportamental e Governança*
* *Grounding* em Fontes de Informação
* Prevenção e Mapeamento de Alucinações
* Tratamento de Contexto Ausente e *Fallback*
* Definição de Fronteiras entre LLM e Arquitetura de Software
