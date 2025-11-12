# Aplicação dos Índices Espectrais NDVI e NDWI Utilizando Python
Artigo voltado para registrar como extrair Índices NDVI e NDWI de uma imagem Raster do satélite CBERS

## Importando os módulos necessários
rasterio: leitura e manipulação de arquivos raster

matplotlib.pyplot: gerar gráficos e mapas.

função show do rasterio.plot: permite utilizar argumentos que facilitam a apresentação de informações.

```
import rasterio 
import matplotlib.pyplot as plt
from rasterio.plot import show
```

## Definindo funções

### Normalização dos dados
As bandas de um satélite podem ter diferentes escalas de brilho, por isso é importante normalizar os dados, dessa forma todos os dados são convertidos para valores entre 0 e 1.

```
def normalize(array):
    array_min, array_max = array.min(), array.max()
    return((array - array_min)/(array_max - array_min))
```

### Função para leitura e armazenamento dos dados em variáveis.
Esta função cria um gerenciador de contexto, que cria as variáveis e armazenam as informações de cada banda espectral do satélite em variáveis distintas (band1, band2, band3 e band4). Além disso, cria a variável profile que guarda metadados importantes como resolução espacial, projeção e transformação geográfica (necessários para o plot).

OBS: O "with" é um gerenciador de contexto que garante que após a realização das operações necessárias o arquivo raster seja fechado automaticamente, evitando uso de recursos computacionais desnecessários e otimizando o código.

```
def cbers_raster(raster):
    with rasterio.open(raster, 'r') as dst:
        band1 = dst.read(1)
        band2 = dst.read(2)
        band3 = dst.read(3)
        band4 = dst.read(4)
        profile = dst.profile
    return band1, band2, band3, band4, profile
```

### Função que aplica os índices espectrais:

Primeiro a função recebe as variáveis criadas para armazenamento das informações das bandas espectrais, fazem a normalização e aplicam os índices NDVI e NDWI.

```
def cbers_index():
    band1, band2, band3, band4, profile = cbers_raster(raster)
    ir, r, g, b = normalize(band4), normalize(band3), normalize(band2), normalize(band1)
    ndvi = (ir - r) / (ir + r)
    ndwi = (g - ir) / (g + ir)
    return ndvi, ndwi, profile
```

#### NDVI (Normalized Difference Vegetation Index) → mede o vigor da vegetação.

Valores altos → vegetação densa e saudável🌳

Valores baixos → áreas degradadas, solo exposto ou urbano 🏙️

#### NDWI (Normalized Difference Water Index) → identifica corpos d’água e áreas úmidas 💧

Valores altos → presença de água

Valores baixos → áreas secas

### Função para plotagem

Cria uma figura com duas visualizações lado a lado: NDVI e NDWI.

Define colormaps temáticos: “Greens” para vegetação e “Blues” para água.

Utiliza o sistema de coordenadas da imagem original (via transform) para manter a georreferência.

```
def plot_cbers():
    ndvi, ndwi, profile = cbers_index()
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(21,7))
    show(ndvi, cmap='Greens', transform=profile['transform'], ax=ax1, title='NDVI INDEX')
    show(ndwi, cmap='Blues', transform=profile['transform'], ax=ax2, title='NDWI INDEX')
```

## Executando o algorítmo
Para executar o algorítmo, cria-se uma variável raster para armazenar a imagem raster do CBERS, a qual é possível obter no catálogo de imagens do INPE em : https://www.dgi.inpe.br/catalogo/explore.
E executar a função "plot_cbers()".

```
raster = 'cbers.tif'
plot_cbers()
```

## Resultados
O resultado é um mapa temático duplo, mostrando lado a lado a distribuição da vegetação (NDVI) e da água (NDWI), permitindo visualizar correlações entre umidade/saúde da vegetação.
As cores refletem os valores dos índices: verde intenso para vegetação e azul para água.

<img width="1464" height="599" alt="image" src="https://github.com/user-attachments/assets/4206d16d-41c5-4ddf-abd4-51ee2f10953e" />

Com algumas linhas de código, é possível processar grandes volumes de dados e gerar insights ambientais valiosos, auxiliando na gestão de recursos naturais.



