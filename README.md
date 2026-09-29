# CP2 SERS - Energias renovaveis e aprendizado de maquina

Entrega do checkpoint com duas tarefas independentes:

- classificacao da fonte renovavel a partir de dados da ANEEL/SIGA;
- regressao da radiacao solar a partir da API historica Open-Meteo.

## Dados

### Classificacao

Arquivo: `aneel_classificacao_orange.csv`

Origem: SIGA/ANEEL. Cada linha representa um empreendimento de geracao no Brasil. As entradas usadas foram `potencia_kw`, `latitude` e `longitude`; o alvo foi `fonte`.

- Linhas: 3876
- Ausentes: 0
- Classes: Hidráulica=1476, Solar=1200, Eólica=1200

### Regressao

Arquivo: `meteo_regressao_orange.csv`

Origem: Open-Meteo historico para Petrolina (PE), de 01/04/2025 a 30/06/2025, entre 7h e 17h. As entradas usadas foram `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh` e `hora`; o alvo foi `radiacao_w_m2`.

- Linhas: 1001
- Ausentes: 0
- Divisao: primeiras 80% das horas para treino e ultimas 20% para teste

## Como executar

```bash
pip install -r requirements.txt
jupyter notebook checkpoint_energia_ml.ipynb
```

Execute o notebook de cima para baixo. Os CSVs ja estao no repositorio.

## Modelos e resultados

### Classificacao

Foram treinados tres classificadores. A divisao treino/teste foi estratificada, com 80% para treino, 20% para teste e semente 42.

As metricas multiclasses usam media `weighted`, adequada ao desbalanceamento entre as classes.

| modelo | accuracy | precision_weighted | recall_weighted | f1_weighted |
| --- | --- | --- | --- | --- |
| Random Forest | 0.9678 | 0.9684 | 0.9678 | 0.9677 |
| KNN | 0.9601 | 0.9606 | 0.9601 | 0.9599 |
| Regressao Logistica | 0.8247 | 0.8313 | 0.8247 | 0.8233 |

Melhor resultado: **Random Forest**, com F1 weighted de **0.9677**.

### Regressao

Foram treinados tres regressores. A avaliacao preserva a ordem temporal.

| modelo | MAE | MSE | R2 |
| --- | --- | --- | --- |
| Random Forest | 67.2010 | 7478.8690 | 0.8410 |
| KNN Regressor | 73.9180 | 8344.3270 | 0.8220 |
| Regressao Linear | 145.2050 | 30034.2010 | 0.3600 |

Melhor resultado: **Random Forest**, com MAE de **67.20 W/m2**.

## Orange

O arquivo `Fluxos_para_Classificacao_e_Regressao.ows` contem os fluxos para Orange Data Mining. Tambem foram adicionadas imagens legiveis dos fluxos:

- `orange_classificacao.png`
- `orange_regressao.png`

## Conclusoes

Na classificacao, localizacao e potencia ajudam a separar parte dos empreendimentos, especialmente porque as fontes tendem a se concentrar em regioes diferentes. A limitacao principal e que esses tres atributos nao descrevem relevo, disponibilidade do recurso natural, tecnologia, fase do projeto ou detalhes regulatorios.

Na regressao, a hora do dia foi decisiva, pois a radiacao segue o ciclo solar. Variaveis meteorologicas como nuvens, temperatura e umidade ajustam esse comportamento. Mesmo assim, radiacao solar nao e geracao eletrica: a geracao depende de painel, area instalada, inclinacao, eficiencia, perdas, sombreamento e operacao do sistema.
