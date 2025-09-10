flowchart TB
  %% ---------- PRE-PROCESSAMENTO (GEE) ----------
  subgraph PRE[Pré-processamento (Google Earth Engine)]
    A0([Início])
    A1[Definir ROI (Pantanal)\n(Shapefile -> geometria válida)]
    A2[Montar coleções Landsat L5/L7/L8/L9\ncalcular NDVI]
    A3[Para cada ANO: empilhar 12 bandas (jan..dez)\nem 300 m, Int16 = NDVI*10000\nNoData = -32768, mesmo CRS/transform]
    A4[Exportar 1 GeoTIFF por ANO\npara uma pasta no Drive]
    A5([Saída: ~40 arquivos .tif\n1985..2024, cada um com 12 bandas])
    A0-->A1-->A2-->A3-->A4-->A5
  end

  %% ---------- PÓS-PROCESSAMENTO (PYTHON) ----------
  subgraph POS[Pós-processamento (Python/Colab, offline)]
    B0([Início])
    B1[Montar Drive\nListar TIFFs anuais]
    B2[Validar alinhamento:\nCRS, transform, resolução, extensão\n(iguais entre todos os anos)]
    B3[Gerar grade de janelas (tiles em px)\nex.: TILE_PX = 32/64]
    B4[Para cada tile:\nler todos os anos (12 bandas/ano)\n→ cubo (T,h,w)]
    B5{Tile com dados\nsuficientes? (valid_frac ≥ limiar)}
    B6[Remover sazonalidade\n(climatologia mensal)\n→ série “anomalias”]
    B7[Detectar 1º break por pixel\n(ruptures Binseg, n_bkps=1)]
    B8[Agregação por tile:\ncontar por ANO e YYYYMM,\ncalcular earliest/latest/modal]
    B9([Acumular resultados])
    B10[Salvar CSVs:\n- tiles_with_breaks\n- breaks_by_tile_year\n- breaks_by_tile_yyyymm]
    B11[Plotar séries medianas por tile\n(com linha vertical no break modal)]
    B12([Saídas prontas p/ análise e aula])
    B0-->B1-->B2-->B3-->B4-->B5
    B5 -- não --> B9
    B5 -- sim --> B6-->B7-->B8-->B9
    B9-->B10-->B11-->B12
  end

  %% opcional
  C{{Opcional depois: trocar\nruptures → BFAST (R via rpy2)}}
  B12-.->C
