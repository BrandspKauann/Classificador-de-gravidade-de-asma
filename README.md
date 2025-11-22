# 🌬️ Classificador de Gravidade de Asma (Lógica Fuzzy)

---

## 🎯 Objetivo e Razão da Escolha

O projeto visa otimizar a triagem e o tratamento de emergência de crises de asma, classificando a **Gravidade** (Leve, Moderada, Severa, Risco Iminente) com base em dados vitais (Saturação de $\text{O}_2$, Frequência Respiratória e Cardíaca).

### Por que Lógica Fuzzy?

A Lógica Fuzzy é ideal para o domínio médico porque lida com a **imprecisão inerente** de sintomas clínicos. Diferente da lógica booleana (0 ou 1), ela permite que uma medição pertença **parcialmente** a múltiplos conjuntos linguísticos . Isso traduz a ambiguidade da clínica para o modelo (e.g., uma $\text{SPO}_2$ de 92% é 50% 'Média' e 50% 'Alta').

---

## ⚙️ Arquitetura do Sistema de Inferência Fuzzy (SIF)

O SIF segue quatro etapas principais: Fuzzificação, Inferência, Agregação e Defuzzificação.

### 1. Fuzzificação: Variáveis e Funções de Pertinência

Definimos as funções de pertinência (geralmente triangulares ou trapezoidais) para mapear os valores de entrada em graus de pertinência (entre 0 e 1) aos termos linguísticos.

#### Exemplo: Funções de Pertinência para Saturação de $\text{O}_2$ ($\text{SPO}_2$)

| Conjunto Linguístico | Faixa Crítica (%) | Função de Pertinência (Ex.) |
| :--- | :--- | :--- |
| **Baixa** | $\le 88$ | `fuzz.trapmf([70, 70, 85, 88])` |
| **Média** | $85 - 93$ | `fuzz.trimf([85, 89, 93])` |
| **Alta** | $\ge 91$ | `fuzz.trapmf([91, 95, 100, 100])` |



### 2. Base de Regras (Inferência)

As regras são a base de conhecimento médico, utilizando operadores **MÍNIMO** ($\land$ AND) e **MÁXIMO** ($\lor$ OR) para combinar os graus de pertinência das entradas.

**Exemplo de Regras Críticas:**

* **Risco Iminente:** **SE** $\text{SPO}_2$ é **Baixa** $\land$ $\text{FC}$ é **Taquicardia** **ENTÃO** Gravidade é **Risco Iminente**.
* **Crise Moderada:** **SE** $\text{SPO}_2$ é **Média** $\land$ $\text{FR}$ é **Alta** **ENTÃO** Gravidade é **Moderada**.

### 3. Defuzzificação: Resultado Final

O conjunto de saída resultante da inferência é convertido em um único **Score Nítido** (crisp value) usando o método do **Centróide** (Center of Gravity). Este score (de 0 a 100) é o nosso valor final de Gravidade/Risco.

---

## 🔬 Curva de Aprendizado e Interpretabilidade (White Box)

A grande vantagem deste modelo é a sua total **interpretabilidade (White Box)**. O processo é totalmente rastreável:

* **Rastreamento de Decisão:** Para um score de **90.71** (Risco Iminente), o sistema revela que a $\text{SPO}_2$ baixa ativou as regras de emergência com força de $\mathbf{0.875}$, dominando o cálculo final.
* **Ajuste Contínuo:** Se o resultado for clinicamente incorreto, o especialista pode modificar diretamente a **Base de Regras** ou as **Funções de Pertinência** (ex: mover o limite de 'Baixa' de $88\%$ para $89\%$), refinando o sistema sem precisar retreinar um modelo complexo.

### Testes de Cenário

| Cenário de Entrada | Inputs | Score de Gravidade | Recomendação de Triagem |
| :--- | :--- | :--- | :--- |
| **Moderada Padrão** | $\text{SPO}_2=90\%$, $\text{FR}=25$, $\text{FC}=105$ | **40.00** | Moderada (Intervenção Rápida) |
| **Risco Iminente** | $\text{SPO}_2=80\%$, $\text{FR}=35$, $\text{FC}=130$ | **90.71** | Risco Iminente (Emergência Imediata) |

---

### 💻 Tecnologias

* `Python`
* `NumPy`
* `scikit-fuzzy` (`skfuzzy.control`)
