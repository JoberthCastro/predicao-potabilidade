# Previsão de Qualidade da Água via Aprendizado Supervisionado: Prevenção de Data Leakage e Escalabilidade com XGBoost

Trabalho de Conclusão de Curso (TCC) em Ciência da Computação. O repositório contém o pipeline reprodutível de classificação binária da potabilidade da água superficial, implementado em `water_quality_pipeline.ipynb`, com ênfase em validade metodológica e capacidade de processamento em larga escala.

---

## Sobre o Projeto

Este trabalho investiga a previsão da potabilidade da água a partir de parâmetros físico-químicos, no formato **Potável** versus **Não Potável**. O objetivo central não é apenas maximizar métricas, mas **auditar falhas metodológicas recorrentes na literatura aplicada a qualidade hídrica** — em especial o vazamento de informação (*data leakage*) pela inclusão de variáveis derivadas do próprio índice de qualidade — e propor um **pipeline escalável, documentado e cientificamente defensável**.

O código organiza o fluxo em módulos com responsabilidade única (`DataLoader`, `DataPreprocessor`, `ModelTrainer`, `Evaluator`), registra eventos via `logging` estruturado e persiste artefatos em `outputs/` (modelo, gráficos e relatórios).

---

## Base de Dados

O experimento utiliza medições reais de qualidade da água superficial, com o índice **CCME Water Quality Index (WQI)** como referência para a rotulagem. A base reúne aproximadamente **2,82 milhões de registros** (1940–2023), consolidando parâmetros físico-químicos e a classificação qualitativa do CCME.

| Atributo | Descrição |
|----------|-----------|
| **Fonte** | [Figshare — DOI 10.6084/m9.figshare.27800394](https://doi.org/10.6084/m9.figshare.27800394) |
| **Título do conjunto** | *A Comprehensive Surface Water Quality Monitoring Dataset (1940-2023): 2.82 Million Record Resource for Empirical and ML-Based Research* |
| **Referência** | Karim, M. R., et al. (2025). *A Comprehensive Dataset of Surface Water Quality Spanning 1940-2023 for Empirical and ML Adopted Research* |
| **Arquivo local** | `data/Combined_dataset.csv` (não versionado; ver [data/DATASET.md](data/DATASET.md)) |

**Features (8 variáveis físico-químicas):** Amônia, DBO, Oxigênio dissolvido, Ortofosfato, pH, Temperatura, Nitrogênio e Nitrato.

**Target binário:** derivado de `CCME_WQI` — *Potável* (1): `Excellent`, `Good`; *Não Potável* (0): `Fair`, `Marginal`, `Poor`. Classes intermediárias são tratadas como não potáveis por critério conservador de saúde pública.

---

## Metodologia

O diferencial metodológico do repositório está na separação rigorosa entre preparação de dados, otimização e avaliação.

### Gestão de grandes volumes de dados

- Leitura do CSV em **chunks** de 100.000 linhas para controle de memória.
- Classificador **XGBoost** com `tree_method='hist'`, adequado a bases com milhões de observações.

### Prevenção de vazamento de dados (*data leakage*)

- **Hold-out estrito:** 80% treino / 20% teste (`stratify`, `random_state=42`), sem conjunto de validação que reutilize o teste.
- **IQR clipping** e **MinMaxScaler** ajustados **somente no treino**; transformações aplicadas depois ao teste.
- Exclusão de `CCME_Values` e `CCME_WQI` como *features* (o índice WQI define o rótulo).
- Exclusão de metadados (`Country`, `Area`, `Waterbody Type`, `Date`) sem relação físico-química direta com a potabilidade modelada.

### Otimização de hiperparâmetros

- **Optuna** (150 *trials*), maximizando acurácia em **validação cruzada estratificada** (K-Fold = 7).
- Busca executada sobre amostra estratificada de **300.000 registros** do treino, preservando a proporção de classes e reduzindo custo computacional.
- **Modelo final** treinado no conjunto de treino completo com os hiperparâmetros selecionados.

### Binarização orientada à saúde pública

A agregação de classes `Fair`, `Marginal` e `Poor` como *Não Potável* adota postura conservadora: erros na direção de subestimar risco sanitário (falsos negativos) são penalizados na análise de *recall* da classe minoritária.

### Análise exploratória (EDA)

Gráficos gerados sobre amostra aleatória de 100.000 registros para viabilizar exploração sem comprometer a representatividade visual.

---

## Resultados Alcançados

Métricas obtidas no **conjunto de teste** (565.596 amostras — 20% da base total), limiar de decisão 0,5:

| Métrica | Valor |
|---------|-------|
| **Acurácia global** | 98,11% |
| **AUC-ROC** | 0,9986 |
| **Recall — Não Potável (classe 0)** | 96,94% (~3,1% de falsos negativos) |
| **Recall — Potável (classe 1)** | 98,55% |
| **F1 macro** | 0,9763 |

Todas as metas pré-definidas no pipeline foram atingidas (acurácia ≥ 90%, AUC-ROC ≥ 90%, *recall* Não Potável ≥ 85%). Detalhes em `outputs/reports/evaluation_report.txt`.

### Interpretação das variáveis preditivas

A importância de *features* do XGBoost aponta **Ortofosfato** e **Amônia** como os maiores ganhos de informação para a separação das classes. Ortofosfato é indicador clássico de carga antrópica (eutrofização, esgoto); amônia associa-se à degradação orgânica recente. Esse comportamento converge com parâmetros monitorados na legislação brasileira de vigilância da qualidade da água para consumo humano, em especial a [Portaria GM/MS nº 888](https://www.gov.br/saude/pt-br) do Ministério da Saúde, reforçando a aderência ecológica do modelo — além do ajuste estatístico.

---

## Estrutura do Repositório

```
.
├── water_quality_pipeline.ipynb   # Pipeline principal (notebook)
├── requirements.txt               # Dependências Python
├── README.md                      # Este arquivo
├── .gitignore
├── data/
│   ├── DATASET.md                 # Instruções de download do dataset
│   └── Combined_dataset.csv       # (local; não versionado)
└── outputs/
    ├── models/
    │   └── xgb_water_quality.json
    ├── plots/                     # Figuras da EDA e avaliação
    └── reports/                   # Relatórios JSON/TXT e resumo do dataset
```

> **Nota:** O notebook permanece na raiz para execução direta. Uma evolução natural do repositório é extrair classes para `src/` e manter apenas orquestração no notebook; a estrutura atual prioriza reprodutibilidade imediata do TCC.

---

## Como Executar o Projeto

### Pré-requisitos

- Python 3.10 ou superior
- RAM recomendada: ≥ 16 GB (processamento de ~2,82 M de linhas)
- Espaço em disco para o CSV completo

### 1. Clonar o repositório

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>
```

### 2. Criar ambiente virtual e instalar dependências

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/macOS
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. Baixar e posicionar o dataset

Siga as instruções em [data/DATASET.md](data/DATASET.md). O arquivo deve ficar em:

```
data/Combined_dataset.csv
```

### 4. Executar o pipeline

```bash
jupyter notebook water_quality_pipeline.ipynb
```

Execute as células **em ordem**. O treinamento completo (carga dos dados, 150 *trials* Optuna e ajuste final do XGBoost no treino integral) pode demandar **várias horas**, dependendo do hardware.

Os artefatos são gravados automaticamente em `outputs/`. A célula `WaterQualityPipeline().run()` existe para reexecução programática ponta a ponta; ao rodar o notebook sequencialmente, as seções 1–5 já executam o fluxo — evite chamar `run()` em seguida para não reprocessar a base inteira.

---

## Referências Bibliográficas

1. **Karim, M. R., et al. (2025).** *A Comprehensive Dataset of Surface Water Quality Spanning 1940-2023 for Empirical and ML Adopted Research.* Disponível em: https://doi.org/10.6084/m9.figshare.27800394

2. **Santos, L. A. S.; Polo, A. C. (2025).** *Previsão da Qualidade da Água: Uma Aplicação do Modelo de Aprendizado Supervisionado XGBoost.* Revista Mirante, v. 18, n. 1, p. 17-31.

3. **Chen, T.; Guestrin, C. (2016).** *XGBoost: A Scalable Tree Boosting System.* Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining.

4. **Lai, M.; et al. (2023).** *Tree-Based Machine Learning Models with Optuna.*

5. **Brasil. Ministério da Saúde.** Portaria GM/MS nº 888, de 4 de maio de 2021 — estabelece os procedimentos de controle e de vigilância da qualidade da água para consumo humano.

---

## Licença e uso

Projeto acadêmico de TCC. Ao reutilizar o código ou os resultados, cite o dataset original (Karim et al., 2025) e as referências metodológicas acima. O modelo **não substitui** análises laboratoriais oficiais nem laudos de vigilância sanitária.

---

## Autor

Repositório preparado para divulgação do Trabalho de Conclusão de Curso. Ajuste a seção **Autor** com seu nome, instituição e orientador antes da publicação no GitHub.
