```mermaid
flowchart TB
  %% --------- PRÉ-PROCESSAMENTO (GEE) ---------
  subgraph PRE[Pré-processamento (Google Earth Engine)]
    A0([Início])
    A1[Definir ROI do Pantanal<br/>(SHP válido)]
    A2[Montar NDVI Landsat L5/L7/L8/L9]
    A3[Para cada ANO: 12 bandas (jan..dez)<br/>300 m • Int16=NDVI*10000 • NoData=-32768 • CRS único]
    A4[Exportar 1 GeoTIFF por ano<br/>(1985–2024)]
    A0 --> A1 --> A2 --> A3 --> A4
  end

  %% --------- PÓS-PROCESSAMENTO (PYTHON) ---------
  subgraph POS[Pós-processamento (Python/Colab, offline)]
    B0([Início])
    B1[Montar Drive e listar TIFFs]
    B2[Validar alinhamento<br/>(CRS, transform, resolução, extensão)]
    B3[Gerar janelas (tiles) em pixels<br/>(ex.: TILE_PX = 32/64)]
    B4[Ler cubo temporal do tile<br/>(todos os anos → série mensal por pixel)]
    B5{Dados suficientes no tile?}
    B6[Remover sazonalidade<br/>(climatologia mensal → anomalias)]
    B7[Detectar 1º break por pixel<br/>(ruptures • Binseg • n_bkps=1)]
    B8[Agregação por tile:<br/>contar por ANO e YYYYMM; earliest/latest/modal]
    B9[Salvar CSVs:<br/>tiles_with_breaks • breaks_by_tile_year • breaks_by_tile_yyyymm]
    B10[Plotar série mediana do tile<br/>(dessazonalizada + linha do break modal)]
    B0 --> B1 --> B2 --> B3 --> B4 --> B5
    B5 -- não --> B9
    B5 -- sim --> B6 --> B7 --> B8 --> B9 --> B10
  end

  A4 --> B0
  C{{Opcional depois:<br/>usar BFAST (R via rpy2)<br/>no lugar do ruptures}}
  B10 -.-> C
```
