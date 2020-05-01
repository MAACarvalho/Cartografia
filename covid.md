# Exploração de dados COVID-19 com Python e QGIS

### Recolha de dados

Para nos mantermos atualizados, recolhemos os últimos dados a um ficheiro atualizado na internet por fontes fidedignas. 

Para o efeito, usamos o pacote pandas que permite ler ficheiros csv e efetuar parsing do mesmo facilmente.


```python
import pandas as pd
ww_df = pd.read_csv("https://opendata.ecdc.europa.eu/covid19/casedistribution/csv")
ww_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dateRep</th>
      <th>day</th>
      <th>month</th>
      <th>year</th>
      <th>cases</th>
      <th>deaths</th>
      <th>countriesAndTerritories</th>
      <th>geoId</th>
      <th>countryterritoryCode</th>
      <th>popData2018</th>
      <th>continentExp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/05/2020</td>
      <td>1</td>
      <td>5</td>
      <td>2020</td>
      <td>222</td>
      <td>4</td>
      <td>Afghanistan</td>
      <td>AF</td>
      <td>AFG</td>
      <td>37172386.0</td>
      <td>Asia</td>
    </tr>
    <tr>
      <th>1</th>
      <td>30/04/2020</td>
      <td>30</td>
      <td>4</td>
      <td>2020</td>
      <td>122</td>
      <td>0</td>
      <td>Afghanistan</td>
      <td>AF</td>
      <td>AFG</td>
      <td>37172386.0</td>
      <td>Asia</td>
    </tr>
    <tr>
      <th>2</th>
      <td>29/04/2020</td>
      <td>29</td>
      <td>4</td>
      <td>2020</td>
      <td>124</td>
      <td>3</td>
      <td>Afghanistan</td>
      <td>AF</td>
      <td>AFG</td>
      <td>37172386.0</td>
      <td>Asia</td>
    </tr>
    <tr>
      <th>3</th>
      <td>28/04/2020</td>
      <td>28</td>
      <td>4</td>
      <td>2020</td>
      <td>172</td>
      <td>0</td>
      <td>Afghanistan</td>
      <td>AF</td>
      <td>AFG</td>
      <td>37172386.0</td>
      <td>Asia</td>
    </tr>
    <tr>
      <th>4</th>
      <td>27/04/2020</td>
      <td>27</td>
      <td>4</td>
      <td>2020</td>
      <td>68</td>
      <td>10</td>
      <td>Afghanistan</td>
      <td>AF</td>
      <td>AFG</td>
      <td>37172386.0</td>
      <td>Asia</td>
    </tr>
  </tbody>
</table>
</div>



### Seleção dos países

Aqui selecionamos os países nos quais estamos interessados.

Para efeitos de demonstração foram escolhidos alguns países da Europa Ocidental/Central.


```python
# Choosing the countries to watch
selected_countries = ["Portugal", "Spain", "France", "Italy", "Germany", 
                      "Austria", "Belgium", "Netherlands", "Denmark", 
                      "Switzerland", "United Kingdom", "Ireland"]
```

### Preparação dos Dados (Python)

Para a preparação dos dados para análise com o Python, criamos uma função que permite recolher a informação relativa a apenas um país.

Dentro desta função, são também efetuadas algumas alterações ao modo de apresentação dos dados.


```python
from datetime import datetime

def get_country (dataframe, country):
    
    # Keeping original dataframe intact
    df = dataframe
    # Changing date format
    df.loc[:, 'date'] = df.apply(lambda x:datetime.strptime("{0} {1} {2}".format(x['year'],x['month'], x['day']), "%Y %m %d"), axis=1)
    # Setting new date as index
    df = df.set_index('date')
    # Renaming columns
    df = df.rename(columns={"countriesAndTerritories": "country", "continentExp": "continent", "popData2018": "population"})
    # Getting data of a single country
    df["country"] = list(map(lambda s: s.replace("_", " "), df["country"]))
    df = df[df["country"] == country]
    # Returning only relevant data and sorting by date
    return df[["cases", "deaths", "country", "continent", "population"]].sort_index()

get_country(ww_df, "Portugal").head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cases</th>
      <th>deaths</th>
      <th>country</th>
      <th>continent</th>
      <th>population</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-03-03</th>
      <td>2</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-04</th>
      <td>2</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-05</th>
      <td>1</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-06</th>
      <td>4</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-07</th>
      <td>4</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
  </tbody>
</table>
</div>



### Filtragem dos países

Usamos então a função acima para recolher todos os países que listamos acima na seleção dos países.


```python
# Mapping the get_country function to every selected country
eu_dfs = list(map(lambda c: get_country(ww_df, c), selected_countries))

eu_dfs[0].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cases</th>
      <th>deaths</th>
      <th>country</th>
      <th>continent</th>
      <th>population</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-03-03</th>
      <td>2</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-04</th>
      <td>2</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-05</th>
      <td>1</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-06</th>
      <td>4</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
    <tr>
      <th>2020-03-07</th>
      <td>4</td>
      <td>0</td>
      <td>Portugal</td>
      <td>Europe</td>
      <td>10281762.0</td>
    </tr>
  </tbody>
</table>
</div>



### Normalização dos dados

Aqui definimos umas funções que normalizarão os dados entre os nossos países. 

Para que seja possível comparar os dados entre os países, é preciso ter em conta a população de cada país e para o efeito foram criadas as seguintes funções.


```python
# Gets the percentage of positive cases by population
def get_percent_cases_by_pop (df):
    
    return df["cases"].div(df["population"]) * 100
    
# Gets the percentage of deaths by population
def get_percent_deaths_by_pop (df):
    
    return df["deaths"].div(df["population"]) * 100
    
get_percent_cases_by_pop(eu_dfs[1])
```




    date
    2019-12-31    0.000000
    2020-01-01    0.000000
    2020-01-02    0.000000
    2020-01-03    0.000000
    2020-01-04    0.000000
                    ...   
    2020-04-26    0.003700
    2020-04-27    0.003919
    2020-04-28    0.002799
    2020-04-29    0.004589
    2020-05-01    0.001109
    Length: 122, dtype: float64



### Análise de infetados

Aqui é feita a preparação dos dados dos testes que retornaram positivo. É aplicada a função acima para normalizar o número de casos e são concatenados todos os países.

Além disso, é aplicada uma soma cumulativa dos casos que retornaram positivo nos vários dias. 

É de notar que esta soma não reflete a percentagem de população infetada, pois ao somar os testes positivos podemos estar a contar pessoas que testaram positivo várias vezes. 


```python
import math
import numpy as np

# Mapping the get_percentage_of_cases functions and concatenating every country dataframe
eu_df_cases = pd.concat(list(map(lambda df: get_percent_cases_by_pop(df), eu_dfs)), axis=1, keys=selected_countries)
# Getting rid of NaN values
eu_df_cases = eu_df_cases.transform(lambda v: list(map(lambda e: 0.0 if isinstance(e, float) and math.isnan(e) else e, v)))
# Cumulative sum
eu_df_cases = eu_df_cases.cumsum()
# Getting rid of rows of all 0's
eu_df_cases = eu_df_cases[(eu_df_cases > 0).apply(np.any, axis=1)]

eu_df_cases
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Portugal</th>
      <th>Spain</th>
      <th>France</th>
      <th>Italy</th>
      <th>Germany</th>
      <th>Austria</th>
      <th>Belgium</th>
      <th>Netherlands</th>
      <th>Denmark</th>
      <th>Switzerland</th>
      <th>United Kingdom</th>
      <th>Ireland</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-01-25</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000004</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-01-26</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000004</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-01-27</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000004</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-01-28</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000004</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-01-29</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000006</td>
      <td>0.000000</td>
      <td>0.000005</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-04-27</th>
      <td>0.232100</td>
      <td>0.448305</td>
      <td>0.185968</td>
      <td>0.327107</td>
      <td>0.187142</td>
      <td>0.171685</td>
      <td>0.403902</td>
      <td>0.219633</td>
      <td>0.147910</td>
      <td>0.340255</td>
      <td>0.229873</td>
      <td>0.396868</td>
    </tr>
    <tr>
      <th>2020-04-28</th>
      <td>0.233686</td>
      <td>0.451105</td>
      <td>0.187752</td>
      <td>0.329985</td>
      <td>0.188522</td>
      <td>0.172442</td>
      <td>0.408744</td>
      <td>0.221954</td>
      <td>0.150032</td>
      <td>0.341465</td>
      <td>0.236353</td>
      <td>0.404821</td>
    </tr>
    <tr>
      <th>2020-04-29</th>
      <td>0.236555</td>
      <td>0.455693</td>
      <td>0.189342</td>
      <td>0.333445</td>
      <td>0.190094</td>
      <td>0.173098</td>
      <td>0.414408</td>
      <td>0.222947</td>
      <td>0.152671</td>
      <td>0.342639</td>
      <td>0.242363</td>
      <td>0.409539</td>
    </tr>
    <tr>
      <th>2020-04-30</th>
      <td>0.238335</td>
      <td>0.455693</td>
      <td>0.191741</td>
      <td>0.336897</td>
      <td>0.191876</td>
      <td>0.173663</td>
      <td>0.419005</td>
      <td>0.225187</td>
      <td>0.155379</td>
      <td>0.344318</td>
      <td>0.248494</td>
      <td>0.417286</td>
    </tr>
    <tr>
      <th>2020-05-01</th>
      <td>0.243694</td>
      <td>0.456802</td>
      <td>0.193441</td>
      <td>0.339994</td>
      <td>0.191876</td>
      <td>0.174714</td>
      <td>0.424783</td>
      <td>0.228170</td>
      <td>0.157966</td>
      <td>0.346420</td>
      <td>0.257566</td>
      <td>0.424683</td>
    </tr>
  </tbody>
</table>
<p>98 rows × 12 columns</p>
</div>



### Análise de mortes

Aqui é feita a preparação dos dados das fatalidades causadas pelo COVID-19 ao longo dos dias. É aplicada a função acima para normalizar o número de mortos e são concatenados todos os países.

Além disso, é aplicada uma soma cumulativa dos mortos pelos vários dias. 

Esta soma já reflete melhor a percentagem de população morta pelo COVID-19, apesar do número de pessoas por país se manter estática com a passagem dos dias (o que não acontece na vida real). 


```python
# Mapping the get_percentage_of_deaths functions and concatenating every country dataframe
eu_df_deaths = pd.concat(list(map(lambda df: get_percent_deaths_by_pop(df), eu_dfs)), axis=1, keys=selected_countries)
# Getting rid of NaN values
eu_df_deaths = eu_df_deaths.transform(lambda v: list(map(lambda e: 0.0 if isinstance(e, float) and math.isnan(e) else e, v)))
# Cumulative sum
eu_df_deaths = eu_df_deaths.cumsum()
# Getting rid of rows of all 0's
eu_df_deaths = eu_df_deaths[(eu_df_deaths > 0).apply(np.any, axis=1)]

eu_df_deaths
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Portugal</th>
      <th>Spain</th>
      <th>France</th>
      <th>Italy</th>
      <th>Germany</th>
      <th>Austria</th>
      <th>Belgium</th>
      <th>Netherlands</th>
      <th>Denmark</th>
      <th>Switzerland</th>
      <th>United Kingdom</th>
      <th>Ireland</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2020-02-15</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-02-16</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-02-17</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-02-18</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2020-02-19</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000001</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-04-27</th>
      <td>0.008783</td>
      <td>0.049632</td>
      <td>0.034120</td>
      <td>0.044090</td>
      <td>0.006934</td>
      <td>0.006126</td>
      <td>0.062108</td>
      <td>0.025971</td>
      <td>0.007279</td>
      <td>0.015687</td>
      <td>0.036179</td>
      <td>0.022396</td>
    </tr>
    <tr>
      <th>2020-04-28</th>
      <td>0.009026</td>
      <td>0.050985</td>
      <td>0.034772</td>
      <td>0.044641</td>
      <td>0.007130</td>
      <td>0.006205</td>
      <td>0.063097</td>
      <td>0.026220</td>
      <td>0.007365</td>
      <td>0.015875</td>
      <td>0.036687</td>
      <td>0.022705</td>
    </tr>
    <tr>
      <th>2020-04-29</th>
      <td>0.009220</td>
      <td>0.051954</td>
      <td>0.035320</td>
      <td>0.045273</td>
      <td>0.007374</td>
      <td>0.006432</td>
      <td>0.064183</td>
      <td>0.026499</td>
      <td>0.007486</td>
      <td>0.016192</td>
      <td>0.038054</td>
      <td>0.023880</td>
    </tr>
    <tr>
      <th>2020-04-30</th>
      <td>0.009463</td>
      <td>0.051954</td>
      <td>0.035958</td>
      <td>0.045807</td>
      <td>0.007582</td>
      <td>0.006556</td>
      <td>0.065671</td>
      <td>0.027340</td>
      <td>0.007641</td>
      <td>0.016521</td>
      <td>0.039250</td>
      <td>0.024518</td>
    </tr>
    <tr>
      <th>2020-05-01</th>
      <td>0.009619</td>
      <td>0.052528</td>
      <td>0.036389</td>
      <td>0.046279</td>
      <td>0.007582</td>
      <td>0.006601</td>
      <td>0.066485</td>
      <td>0.027828</td>
      <td>0.007797</td>
      <td>0.016697</td>
      <td>0.040264</td>
      <td>0.025384</td>
    </tr>
  </tbody>
</table>
<p>77 rows × 12 columns</p>
</div>



### Apresentação dos dados

Aqui apresentamos na forma de gráficos os dados que estivemos a analisar. 

Nestes gráficos podemos ver as curvas tão faladas pelos cientistas e a capacidade que cada país teve para "suavizar a curva" com as medidas de isolamento social.

Além disso, apresentamos os gráficos de infetados e mortes lado a lado para que seja possível testemunhar a capacidade que os países de tratar os infetados e evitar que a infeção levasse à fatalidade.


```python
import matplotlib.pyplot as plt

# Plotting the percentage of cases in the chosen countries 
eu_df_cases.plot()
plt.xlabel("Date")
plt.ylabel("Percentage of positive tests by population")
plt.title("Infected People")

plt.figure()

# Plotting the percentage of deaths in the chosen countries 
eu_df_deaths.plot()
plt.xlabel("Date")
plt.ylabel("Percentage of population killed by COVID-19")
plt.title("COVID-19 Death Toll")
```




    Text(0.5, 1.0, 'COVID-19 Death Toll')




![png](covid_files/covid_16_1.png)



    <Figure size 432x288 with 0 Axes>



![png](covid_files/covid_16_3.png)


### Preparação dos dados QGIS

A apresentação em gráficos apesar de muito explícita, pode ser um pouco limitada. 

Então, dada a natureza geográfica do problema em questão, recorremos ao QGIS para apresentar de maneira geográfica os dados.

Então começamos por preparar uns csvs para facilitar a leitura e apresentação dos dados no QGIS.


```python
# Keeping original dataframe intact
df = ww_df
# Changing the date format
df.loc[:, 'date'] = df.apply(lambda x:datetime.strptime("{0} {1} {2}".format(x['year'],x['month'], x['day']), "%Y %m %d"), axis=1)
# Renaming columns
df = df.rename(columns={"countriesAndTerritories": "country", "continentExp": "continent", "popData2018": "population"})
# Reshaping the dataframe
df = df[["date","cases", "deaths", "country", "continent", "population"]].sort_index()

# Renaming some countries ("United_Kingdom" => "United Kingdom")
df["country"] = list(map(lambda s: s.replace("_", " "), df["country"]))
# Dropping continent info
df = df.drop(["continent"], axis=1)
# Getting only selected countries
df = df[df.apply(lambda v: True if v[3] in selected_countries else False, axis=1)]

# Applying normalization by population of the number of cases and deaths
df["cases"] = df["cases"] / df["population"] * 100
df["deaths"] = df["deaths"] / df["population"] * 100

# Getting cuulative cases and deaths
df.insert(2, "cases_cumulative", "")
df.insert(4, "deaths_cumulative", "")
for c in selected_countries:
    df.loc[df["country"] == c, 'cases_cumulative'] = df.sort_values("date").loc[df["country"] == c, "cases"].cumsum()
    df.loc[df["country"] == c, 'deaths_cumulative'] = df.sort_values("date").loc[df["country"] == c, "deaths"].cumsum()

# Saving the dataframe as csv
df.to_csv("cases_deaths_percent.csv")

# Saving a dataframe with only the countries as csv (for filtering in QGIS)
pd.DataFrame(data={'country': selected_countries}).to_csv("selected_countries.csv")

df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>cases</th>
      <th>cases_cumulative</th>
      <th>deaths</th>
      <th>deaths_cumulative</th>
      <th>country</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>787</th>
      <td>2020-05-01</td>
      <td>0.001051</td>
      <td>0.174714</td>
      <td>0.000045</td>
      <td>0.00660108</td>
      <td>Austria</td>
      <td>8847037.0</td>
    </tr>
    <tr>
      <th>788</th>
      <td>2020-04-30</td>
      <td>0.000565</td>
      <td>0.173663</td>
      <td>0.000124</td>
      <td>0.00655587</td>
      <td>Austria</td>
      <td>8847037.0</td>
    </tr>
    <tr>
      <th>789</th>
      <td>2020-04-29</td>
      <td>0.000656</td>
      <td>0.173098</td>
      <td>0.000226</td>
      <td>0.00643153</td>
      <td>Austria</td>
      <td>8847037.0</td>
    </tr>
    <tr>
      <th>790</th>
      <td>2020-04-28</td>
      <td>0.000757</td>
      <td>0.172442</td>
      <td>0.000079</td>
      <td>0.00620547</td>
      <td>Austria</td>
      <td>8847037.0</td>
    </tr>
    <tr>
      <th>791</th>
      <td>2020-04-27</td>
      <td>0.000622</td>
      <td>0.171685</td>
      <td>0.000068</td>
      <td>0.00612634</td>
      <td>Austria</td>
      <td>8847037.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>13654</th>
      <td>2020-01-04</td>
      <td>0.000000</td>
      <td>0</td>
      <td>0.000000</td>
      <td>0</td>
      <td>United Kingdom</td>
      <td>66488991.0</td>
    </tr>
    <tr>
      <th>13655</th>
      <td>2020-01-03</td>
      <td>0.000000</td>
      <td>0</td>
      <td>0.000000</td>
      <td>0</td>
      <td>United Kingdom</td>
      <td>66488991.0</td>
    </tr>
    <tr>
      <th>13656</th>
      <td>2020-01-02</td>
      <td>0.000000</td>
      <td>0</td>
      <td>0.000000</td>
      <td>0</td>
      <td>United Kingdom</td>
      <td>66488991.0</td>
    </tr>
    <tr>
      <th>13657</th>
      <td>2020-01-01</td>
      <td>0.000000</td>
      <td>0</td>
      <td>0.000000</td>
      <td>0</td>
      <td>United Kingdom</td>
      <td>66488991.0</td>
    </tr>
    <tr>
      <th>13658</th>
      <td>2019-12-31</td>
      <td>0.000000</td>
      <td>0</td>
      <td>0.000000</td>
      <td>0</td>
      <td>United Kingdom</td>
      <td>66488991.0</td>
    </tr>
  </tbody>
</table>
<p>1410 rows × 7 columns</p>
</div>



### Resultados do QGIS

Com os ficheiros csv criados acima e um GeoPackage com o mapa mundo, podemos no QGIS criar uma Virtual Layer que resulta do Inner Join entre o geopackage e o csv com os casos e as mortes. 

Desse modo, podemos apresentar os dados obtidos num mapa e fornecer uma melhor visualização dos mesmos.

Uma maneira de o fazer é através da apresentação dos dados ao longo do tempo em forma de animação. Com recurso ao plugin TimeManager do QGIS é muito fácil fazer isso e abaixo apresentamos 2 gifs extraídos com esse plugin.

### Infetados dia a dia

Nesta animação conseguimos ver a comparação do número de infetados que são publicados todos os dias para os países que selecionamos acima. 

Estes números como foi dito acima são divididos pela população de cada país para que a comparação seja justa.

<img src="cases.gif" width="500" align="left">

### Mortos dia a dia

Nesta animação podemos ver a comparação do número de mortos que são publicados todos os dias para os países que selecionamos acima. 

Da mesma maneira que os infetados, estes números também são normalizados com população de cada país.

<img src="deaths.gif" width="500" align="left">

### Total de Mortos

Na animação seguinte testemunhamos a acumulação da percentagem de mortes em cada país que selecionamos. 

Esta visualização permite realmente comparar os países na sua totalidade de fatalidades.

<img src="death_toll.gif" width="500" align="left">
