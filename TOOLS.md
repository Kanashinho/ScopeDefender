# 🧰 Mapeamento de Ferramentas (Nodes)

No framework do **N8N**, cada nó (*Node*) funciona como uma ferramenta específica para a arquitetura do agente.

O Scope Defender utiliza uma separação explícita entre **processamento semântico, cálculo matemático, tomada de decisão e geração de linguagem**.

---

## 🔹 On Manual Click — Trigger

**Função:** inicia a execução controlada do workflow.

Utilizado para:

- Demonstrações.
- Testes.
- Validação das diferentes ramificações.

---

## 🔹 Edit Fields — Set

**Função:** ferramenta de injeção de texto.

Simula a caixa de entrada interceptando a demanda do gestor.

Pode fornecer:

- Demanda solicitada.
- Prazo.
- Contexto da cobrança.
- Estimativa de esforço.

---

## 🔹 Read/Write Files from Disk — Leitura

**Função:** ferramenta de extração.

Carrega a capacidade da equipe através de um arquivo JSON local.

Os dados podem representar:

- Capacidade disponível.
- Alocação atual.
- Outras variáveis necessárias para o cálculo.

---

## 🔹 Extract from File

**Função:** faz o parse do arquivo para propriedades manipuláveis pelo workflow.

Transforma o conteúdo estruturado em dados que podem ser utilizados pelos próximos nós.

---

## 🔹 Code — JavaScript

**Função:** motor aritmético determinístico.

Executa o cálculo exato do saldo de horas:

```text
saldo = capacidade - alocação
```

O cálculo foi deliberadamente retirado do LLM para evitar **alucinações matemáticas**, erros de subtração ou interpretações incorretas de sinais.

### Princípio

> **LLM interpreta. Code calcula.**

O resultado produzido pelo Code Node serve como fonte numérica confiável para o restante do workflow.

---

## 🔹 Switch — Roteador Determinístico

**Função:** roteamento baseado em regras explícitas.

O Switch avalia o saldo calculado e divide o fluxo em três cenários:

### 🔴 Inviável

```text
saldo < 0
```

Direciona para o comunicador defensivo.

### 🟡 Parcial

```text
saldo >= 0 && saldo < 8
```

Direciona para o comunicador de alerta de risco.

### 🟢 Viável

```text
saldo >= 8
```

Direciona para o comunicador de aceite limpo.

A decisão não depende do LLM.

---

## 🔹 Ollama — Comunicadores IA (3x)

**Função:** geração de linguagem natural com **Prompt Isolation**.

Existem três nós Ollama independentes, cada um com um prompt específico.

### 1. Comunicador — Inviável

**Objetivo:** defesa de escopo.

Características:

- Tom corporativo.
- Fundamentação baseada em números.
- Recusa justificada do prazo.
- Proposta de divisão do escopo quando aplicável.

---

### 2. Comunicador — Parcial

**Objetivo:** alerta de risco.

Características:

- Reconhece que existe capacidade limitada.
- Comunica o risco associado ao prazo.
- Evita uma recusa absoluta quando a entrega parcial é possível.
- Pode propor priorização de funcionalidades.

---

### 3. Comunicador — Viável

**Objetivo:** aceite limpo.

Características:

- Confirma a viabilidade.
- Mantém comunicação objetiva.
- Não introduz riscos inexistentes.
- Não cria justificativas desnecessárias.

---

### Por que três comunicadores?

Um único prompt genérico apresentou comportamento inconsistente entre cenários.

O **Prompt Isolation** resolve esse problema restringindo cada agente a uma única função comunicacional.

```text
                    SWITCH
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      INVIÁVEL       PARCIAL       VIÁVEL
          │            │            │
          ▼            ▼            ▼
      Ollama #1      Ollama #2     Ollama #3
      Defensivo       Alerta        Aceite
```

---

## 🔹 Write to File — Gravação

**Função:** ferramenta de auditoria.

Salva o e-mail gerado diretamente em arquivos `.txt`, permitindo registrar a trajetória do agente.

Arquivos esperados:

```text
/data/trajectory_01_inviavel.txt
/data/trajectory_02_parcial.txt
/data/trajectory_03_viavel.txt
```

Esses arquivos permitem verificar posteriormente qual resposta foi produzida para cada cenário.

---

# 🧩 Resumo da Arquitetura

| Node | Função | Determinístico? |
|---|---|---|
| **On Manual Click** | Trigger | Sim |
| **Edit Fields** | Entrada | Sim |
| **Read/Write Files** | Leitura de dados | Sim |
| **Extract from File** | Parse | Sim |
| **Code** | Cálculo matemático | **Sim** |
| **Switch** | Roteamento | **Sim** |
| **Ollama #1** | Comunicação inviável | Não |
| **Ollama #2** | Comunicação parcial | Não |
| **Ollama #3** | Comunicação viável | Não |
| **Write to File** | Auditoria | Sim |

---

# 🎯 Princípio da Arquitetura

O Scope Defender segue uma divisão explícita de responsabilidades:

```text
┌─────────────────────────────────────┐
│           CAMADA SEMÂNTICA          │
│                                     │
│             Ollama                  │
│       Interpretação / Texto         │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│           CAMADA LÓGICA             │
│                                     │
│       Code Node + Switch            │
│        Cálculo + Decisão            │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│           CAMADA DE SAÍDA           │
│                                     │
│          Write to File              │
│             Auditoria               │
└─────────────────────────────────────┘
```

O resultado é uma arquitetura mais previsível, auditável e resistente a falhas de raciocínio matemático ou mistura de instruções entre diferentes cenários.