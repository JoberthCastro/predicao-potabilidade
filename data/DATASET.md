# Base de dados

O arquivo `Combined_dataset.csv` **não está versionado** neste repositório devido ao volume (~2,82 milhões de registros).

## Download

1. Acesse o repositório oficial no Figshare:  
   [https://doi.org/10.6084/m9.figshare.27800394](https://doi.org/10.6084/m9.figshare.27800394)

2. Baixe o dataset **A Comprehensive Surface Water Quality Monitoring Dataset (1940-2023): 2.82 Million Record Resource for Empirical and ML-Based Research**.

3. Extraia o arquivo e coloque-o nesta pasta com o nome:

   ```
   data/Combined_dataset.csv
   ```

## Referência

Karim, M. R., et al. (2025). *A Comprehensive Dataset of Surface Water Quality Spanning 1940-2023 for Empirical and ML Adopted Research*.

## Colunas utilizadas no pipeline

| Coluna | Papel |
|--------|--------|
| Ammonia (mg/l) | Feature |
| Biochemical Oxygen Demand (mg/l) | Feature |
| Dissolved Oxygen (mg/l) | Feature |
| Orthophosphate (mg/l) | Feature |
| pH (ph units) | Feature |
| Temperature (cel) | Feature |
| Nitrogen (mg/l) | Feature |
| Nitrate (mg/l) | Feature |
| CCME_WQI | Target (binarizado em potável / não potável) |

Colunas excluídas do modelo: `CCME_Values`, `CCME_WQI` (como feature), `Country`, `Area`, `Waterbody Type`, `Date` — ver documentação no notebook.
