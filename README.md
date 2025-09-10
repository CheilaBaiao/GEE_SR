```mermaid
flowchart TB
  A([Inicio]) --> B[Pre-processamento - GEE: ROI -> NDVI -> 12 bandas/ano -> GeoTIFF]
  B --> C[Pos-processamento - Python: montar Drive, validar alinhamento, tiles]
  C --> D[Remover sazonalidade (anomalias mensais)]
  D --> E[Detectar 1o break por pixel (ruptures, n_bkps=1)]
  E --> F[Agregacoes por tile (por ANO e YYYYMM)]
  F --> G[Gerar CSVs e plots]
  G -.-> H{{Opcional: BFAST (R via rpy2)}}
```

