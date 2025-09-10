```mermaid
flowchart TB
  %% --------- PRE-PROCESSAMENTO (GEE) ---------
  subgraph PRE[Pre-processamento - GEE]
    A0([Inicio])
    A1[Definir ROI (Pantanal)]
    A2[Montar NDVI Landsat L5/L7/L8/L9]
    A3[Para cada ANO: 12 bandas (jan..dez) @300 m; Int16=NDVI*10000; NoData=-32768; CRS unico]
    A4[Exportar 1 GeoTIFF por ano (1985-2024)]
    A0 --> A1 --> A2 --> A3 --> A4
  end

  %% --------- POS-PROCESSAMENTO (PYTHON) ---------
  subgraph POS[Pos-processamento - Python/Colab]
    B0([Inicio])
    B1[Montar Drive e listar TIFFs]
    B2[Validar alinhamento (CRS, transform, resolucao, extent)]
    B3[Gerar janelas (tiles) em pixels (TILE_PX=32/64)]
    B4[Ler cubo temporal do tile (todos os anos -> serie mensal por pixel)]
    B5{Dados suficientes no tile?}
    B6[Remover sazonalidade (climatologia mensal -> anomalias)]
    B7[Detectar 1o break por pixel (ruptures Binseg, n_bkps=1)]
    B8[Agregacao por tile: contagem por ANO e YYYYMM; earliest/latest/modal]
    B9[Salvar CSVs]
    B10[Plotar serie mediana do tile (linha no break modal)]
    B0 --> B1 --> B2 --> B3 --> B4 --> B5
    B5 -- nao --> B9
    B5 -- sim --> B6 --> B7 --> B8 --> B9 --> B10
  end

  A4 --> B0
  C{{Opcional: BFAST (R via rpy2)}}
  B10 -.-> C
```

  B10 -.-> C
```
