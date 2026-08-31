# 🛡️ Scope Defender

**Scope Defender** é um orquestrador de IA desenvolvido para auxiliar profissionais e equipes de engenharia na negociação de prazos e escopos de demandas urgentes.

## 🎯 O Problema

Equipes de engenharia frequentemente aceitam prazos irreais sob pressão porque não possuem tempo suficiente para calcular sua capacidade técnica no momento da cobrança.

Isso transforma uma decisão que deveria ser baseada em dados em uma negociação baseada em pressão, estimativas mentais e percepção subjetiva.

## 💡 A Solução

O **Scope Defender** utiliza **N8N + LLMs locais** para interceptar uma demanda urgente, cruzá-la com a capacidade atual da equipe e gerar uma resposta corporativa fundamentada em dados concretos.

A arquitetura separa claramente as responsabilidades:

- **LLM:** interpretação semântica e geração de linguagem.
- **N8N:** orquestração do processo.
- **Code Node:** cálculos matemáticos determinísticos.
- **Switch:** tomada de decisão baseada em regras.
- **Ollama:** inferência local.
- **Write to File:** registro das trajetórias para auditoria.

---

## 🏗️ Arquitetura do Workflow

> **TODO:** Adicione aqui um screenshot atualizado mostrando as 3 ramificações finais no N8N.

O sistema opera em uma arquitetura determinística, separando **raciocínio semântico** de **execução lógica**:

### 1. Input

Recebe o e-mail de cobrança do gestor e extrai os dados estruturados da equipe.

Os dados utilizados pelo workflow incluem informações como:

- Capacidade disponível.
- Alocação atual.
- Esforço estimado da nova demanda.
- Prazo solicitado.

### 2. Cálculo Nativo — Code Node

Um script JavaScript simples processa a matemática real:

```text
capacidade - alocação = saldo
```

O cálculo é executado diretamente pelo N8N, eliminando o risco de alucinação numérica da IA.

A IA não é responsável por realizar a operação matemática.

### 3. Switch Determinístico — N8N

O Switch avalia o saldo calculado e determina qual cenário deve ser executado.

As três condições são:

```text
saldo < 0
```

**INVIÁVEL**

```text
saldo >= 0 && saldo < 8
```

**PARCIAL**

```text
saldo >= 8
```

**VIÁVEL**

Dessa forma, o roteamento é realizado por regras determinísticas, sem depender da interpretação subjetiva do LLM.

### 4. Agentes Comunicadores Isolados — Ollama

Após o Switch, o workflow possui **três nós Ollama independentes**.

Cada agente possui um prompt específico e restrito ao seu cenário:

| Cenário | Agente | Comportamento |
|---|---|---|
| 🔴 Inviável | Comunicador Defensivo | Defesa do escopo e justificativa baseada no déficit |
| 🟡 Parcial | Comunicador de Alerta | Aceite condicionado e explicitação dos riscos |
| 🟢 Viável | Comunicador de Aceite | Aceite limpo e objetivo |

Essa arquitetura implementa **Prompt Isolation**, evitando que um único prompt genérico tente interpretar simultaneamente diferentes cenários de negócio.

As variáveis calculadas pelo workflow são injetadas diretamente nos prompts, mantendo os números fora da responsabilidade do modelo.

### 5. Gravação de Trajetórias

Cada ramificação grava sua saída em um arquivo `.txt` separado para permitir auditoria e análise posterior da execução.

Os arquivos são armazenados em:

```text
/data/
```

---

## 🧠 Princípio de Engenharia

Um dos principais princípios arquiteturais do Scope Defender é:

> **A IA interpreta e comunica. O código calcula. O workflow decide.**

O fluxo pode ser representado como:

```text
                    ┌─────────────────┐
                    │      INPUT      │
                    │ Demanda + Dados │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   CODE NODE     │
                    │ Cálculo exato   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     SWITCH      │
                    │ Regras de saldo │
                    └─────┬───┬───┬───┘
                          │   │   │
                 < 0 ─────┘   │   └───── >= 8
                          0–<8│
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       ┌────────────┐  ┌────────────┐  ┌────────────┐
       │  OLLAMA #1 │  │  OLLAMA #2 │  │  OLLAMA #3 │
       │  INVIÁVEL  │  │  PARCIAL   │  │   VIÁVEL   │
       └──────┬─────┘  └──────┬─────┘  └──────┬─────┘
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                    ┌─────────────────┐
                    │  WRITE TO FILE  │
                    │   Trajetória    │
                    └─────────────────┘
```

---

## ⚙️ Pré-requisitos

Para executar o projeto localmente:

- Docker
- Docker Compose
- N8N
- Ollama
- NVIDIA Container Toolkit
- GPU NVIDIA compatível com aceleração CUDA

### Portas utilizadas

| Serviço | Porta |
|---|---:|
| N8N | `5678` |
| Ollama | `11434` |

---

## 🖥️ Hardware de Referência

O projeto foi desenvolvido, testado e otimizado localmente utilizando:

| Componente | Especificação |
|---|---|
| CPU | AMD Ryzen 7 5700X |
| GPU | NVIDIA RTX 3070 8 GB |
| RAM | 16 GB |
| Inferência | Ollama + GPU NVIDIA |
| Modelo | Llama 3 8B |

A GPU acelera a inferência local do modelo, permitindo o processamento do workflow em poucos segundos em condições adequadas de execução.

---

# 🚀 Guia de Reprodução

## 1. Clone o repositório

```bash
git clone https://github.com/SEU-USUARIO/ScopeDefender.git
cd ScopeDefender
cp .env.example .env
```

Configure o arquivo `.env` conforme necessário.

---

## 2. Suba a infraestrutura

```bash
docker-compose up -d
```

Verifique os containers:

```bash
docker ps
```

---

## 3. Instale a Inteligência

Na primeira execução, o servidor Ollama estará sem modelos.

Baixe o modelo:

```bash
docker exec -it ollama_scope_defender ollama run llama3
```

Aguarde o download terminar.

Quando aparecer:

```text
>>>
```

execute:

```text
/bye
```

e pressione `Enter`.

---

## 4. Importe os Workflows

Acesse:

```text
http://localhost:5678
```

Depois:

1. Vá em **Workflows**.
2. Clique em **Add Workflow**.
3. Abra o menu de opções.
4. Selecione **Import from File**.
5. Importe:

```text
baseline_workflow.json
scope_defender_workflow.json
```

---

## 5. Teste o Sistema

Abra o fluxo **Scope Defender**, clique no nó manual de gatilho e execute o workflow.

O fluxo gravará automaticamente as saídas na pasta `/data/` como:

```text
trajectory_01_inviavel.txt
trajectory_02_parcial.txt
trajectory_03_viavel.txt
```

Cada arquivo representa a trajetória de execução correspondente ao cenário processado.

---

# 📥 Input e 📤 Output

## Input

Exemplo de dados processados pelo workflow:

```json
{
  "disponivel": -40,
  "exigido": 20
}
```

O **Code Node** realiza o cálculo:

```text
saldo = disponível - exigido
saldo = -40 - 20
saldo = -60
```

Resultado:

```json
{
  "disponivel": -40,
  "exigido": 20,
  "saldo": -60
}
```

---

## Output

Como o saldo é inferior a zero, o Switch direciona a execução para o comunicador **INVIÁVEL**.

Exemplo:

> Olá, Gestor,
>
> Lamento informar que não posso atender ao prazo de hoje às 17h para a nova demanda. A equipe já está trabalhando com um déficit de -40h e agora precisa absorver mais 20h. Isso resultaria em um déficit total de -60h, comprometendo significativamente a qualidade e a segurança da entrega.
>
> Para resolvermos de forma realista, sugiro dividir o escopo em duas etapas: Fase 1, contemplando as funcionalidades essenciais, e Fase 2, contemplando as demais integrações e funcionalidades complementares.

---

# 📊 Métricas de Melhoria

| Métrica | Baseline Manual | Scope Defender |
|---|---|---|
| **Tempo de análise** | ~25 min | **< 10 segundos** |
| **Precisão do cálculo** | Estimativa mental sob pressão | **100% via Code Node** |
| **Argumentação utilizada** | Sem números concretos | **Saldo calculado automaticamente** |
| **Alternativa proposta** | Raramente elaborada sob estresse | **Gerada conforme cenário** |
| **Controle do tom** | Prompt genérico | **3 prompts isolados** |
| **Roteamento** | Dependente da interpretação da IA | **Determinístico via Switch** |
| **Auditoria** | Inexistente/manual | **Trajetórias `.txt`** |

> As métricas representam o cenário de referência utilizado durante o desenvolvimento e os testes do projeto.

---

# 🔐 Privacidade e Processamento Local

O Scope Defender utiliza **Ollama** para executar os modelos localmente.

A arquitetura reduz a necessidade de enviar dados internos da equipe para APIs externas de modelos de linguagem.

O processamento pode permanecer dentro do ambiente local:

- Dados da equipe.
- Inferência do LLM.
- Workflows do N8N.
- Resultados gerados.
- Trajetórias de execução.

---

# 🛠️ Disclosure de IA

Este projeto utiliza:

- **Orquestração:** N8N via Docker
- **Motor de inferência:** Ollama
- **Aceleração:** NVIDIA Container Toolkit
- **Modelo:** Llama 3 8B
- **Arquitetura:** Agentes comunicadores + regras determinísticas
- **Cálculos:** JavaScript via N8N Code Node
- **Auditoria:** arquivos `.txt`

---

# 📁 Estrutura do Projeto

```text
ScopeDefender/
│
├── README.md
├── TOOLS.md
├── CHANGELOG.md
├── docker-compose.yml
├── .env.example
├── .gitignore
│
├── workflows/
│   ├── baseline_workflow.json
│   └── scope_defender_workflow.json
│
├── data/
│   ├── team_capacity.json
│   ├── trajectory_01_inviavel.txt
│   ├── trajectory_02_parcial.txt
│   └── trajectory_03_viavel.txt
│
└── screenshots/
    └── workflow.png
```

---

# 📜 Licença

Defina aqui a licença do projeto caso o repositório seja publicado.

Exemplo:

```text
MIT License
```