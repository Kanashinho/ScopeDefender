# Registro de Melhorias (Improvement Changelog)

Este documento detalha a evolução do Scope Defender, destacando os modos de falha encontrados em LLMs locais (Ollama / Llama3) e as decisões arquitetônicas no N8N para mitigá-los.

### v1.0 — Baseline Manual
* **O que foi tentado:** Resposta direta e manual a e-mails de cobrança sem ingestão de dados.
* **Resultado:** Respostas genéricas e argumentativas, incapazes de defender prazos sob pressão com dados concretos.
* **Decisão:** Necessidade de automatizar a extração do escopo e cruzar com o JSON de tarefas da equipe.

### v1.1 — Análise de Capacidade Automática (O problema do Template)
* **O que foi tentado:** Agente Analista avaliando o JSON com o prompt solicitando a saída no formato `"VIAVEL | PARCIAL | INVIAVEL"`.
* **Modo de Falha:** O modelo copiou o template inteiro de forma literal na saída em vez de processar a decisão lógica, quebrando o roteamento.
* **Solução:** Refinamento do Output Formatting. O prompt foi alterado para proibir texto adicional e forçar a saída de apenas uma das três palavras-chave na primeira linha.

### v1.2 — Identificação do Responsável (Cegueira de Agregação)
* **O que foi tentado:** Forçar o LLM a ler o array bruto de tarefas (18+ itens) e calcular a carga horária de todos os membros.
* **Modo de Falha:** O modelo multiplicou os 4 membros por 40h, ignorando as tarefas já alocadas e gerando um falso positivo de disponibilidade (160h livres).
* **Solução:** Direcionamento explícito. Modificação das regras no prompt para que o modelo ignore o array solto e leia apenas o objeto `capacity_summary` pré-calculado.

### v1.3 — Roteamento Determinístico (Matemática vs IA)
* **O que foi tentado:** Usar o LLM para classificar matematicamente se o saldo final (-60h) significava "PARCIAL" ou "INVIAVEL", confiando nele como um roteador de fluxo.
* **Modo de Falha:** Limitação do LLM com operadores booleanos em números negativos. Ele classificou um saldo negativo extremo como "PARCIAL".
* **Solução (Insight Principal):** LLMs não devem ser roteadores lógicos. O prompt foi alterado para gerar apenas um objeto estruturado estrito (`{"disponivel": -40, "exigido": 20, "saldo": -60}`). O N8N (Nó Switch) assumiu a responsabilidade de rotear a decisão com base em expressões matemáticas (`< 0`), eliminando a margem de erro.

### v1.4 — Geração Diplomática (Colapso de Contexto)
* **O que foi tentado:** Injetar a saída JSON bruta do Analista diretamente no prompt do Agente Comunicador para embasar o e-mail final.
* **Modo de Falha:** O LLM entrou em colapso semântico, gerou erros graves de tradução literal (loop de texto, trocando "prazo" por "prato") e perdeu a coesão.
* **Solução:** Injeção limpa de variáveis via N8N. As propriedades numéricas foram extraídas do JSON usando expressões (`{{ JSON.parse($json.conteúdo).saldo }}`) antes de serem passadas ao prompt, entregando ao modelo apenas texto limpo e legível. Resultado: E-mail impecável e irrefutável.

### v1.5 — O Fim da Alucinação Matemática (Code Node)
* **O que foi tentado:** Manter o cálculo de saldo = disponível - exigido dentro da interpretação do LLM Analista.
* **Modo de Falha:** O modelo frequentemente errava a subtração ou se confundia com a regra de sinais, gerando saldos incorretos que quebravam o fluxo.
* **Solução (Insight Crítico):** LLMs não devem realizar cálculos matemáticos. A aritmética foi transferida para um nó de Código nativo (JavaScript) no N8N. O motor calcula o saldo com 100% de precisão e entrega os dados processados para a IA apenas redigir, eliminando a alucinação matemática.

### v1.6 — Isolamento de Prompt (Prompt Isolation)
* **O que foi tentado:** Usar apenas um nó de IA Comunicador no final do fluxo, injetando uma variável de status (VIAVEL ou INVIAVEL) em um prompt genérico.
* **Modo de Falha:** O modelo tentava misturar tons corporativos, gerando e-mails confusos que recusavam o prazo mas agradeciam de forma submissa, ou aceitavam a tarefa criando problemas irreais.
* **Solução**: Implementação de Prompt Isolation. O fluxo foi ramificado em três nós Ollama independentes após o Switch. Cada nó possui um prompt restrito e desenhado especificamente para o seu cenário:
Inviável: comunicação defensiva e fundamentada em capacidade.
Parcial: comunicação com alerta explícito de risco.
Viável: aceite limpo e objetivo.