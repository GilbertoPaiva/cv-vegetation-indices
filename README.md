# Processamento Digital de Imagens — Índices de Vegetação

Pipeline de visão computacional em Python para análise de cobertura vegetal em imagens RGB, implementando os índices espectrais **VARI** e **ExG** com segmentação, filtragem comparativa e detecção de bordas.

> Projeto A3 — Computação Gráfica | Ciência da Computação

---

## Pipeline completo

![Pipeline completo](outputs/grid_pipeline_completo.png)

*Grid 2×5 com todos os estágios: original → grayscale → filtros (Gaussiano, Mediana, Bilateral) → Canny → VARI → ExG → segmentação Otsu → segmentação ExG com contornos*

---

## Índices de vegetação implementados

### VARI — Visible Atmospherically Resistant Index

```
VARI = (G − R) / (G + R − B)
```

Desenvolvido por Gitelson et al. (2002). Range teórico −1 a +1: valores positivos indicam vegetação viva, negativos indicam solo, estruturas ou água. Resistente a variações de iluminação atmosférica.

**Implementação:** máscara de luminância exclui pixels de sombra (`(R+G+B)/3 < 0.1`) onde o denominador colapsa para zero, eliminando outliers numéricos sem significado físico.

### ExG — Excess Green Index

```
ExG = 2G − R − B
```

Desenvolvido por Woebbecke et al. (1995). Threshold natural em zero: `ExG > 0` → presença de vegetação. Mais simples que o VARI, altamente eficaz para segmentação direta.

---

## Técnicas implementadas

| Técnica | Detalhes |
|---|---|
| Conversão de espaço de cores | BGR → RGB, escala de cinza |
| Separação de canais | R, G, B em `float32` normalizado [0, 1] |
| Cálculo de índices espectrais | VARI com máscara de luminância, ExG |
| Histogramas | Canais R, G, B individuais + distribuição ExG |
| Filtro Gaussiano | σ = 1.5, PSNR = 25.39 dB |
| Filtro de Mediana | kernel 5×5, PSNR = 26.12 dB |
| Filtro Bilateral | d = 9, σColor = 75, PSNR = 32.01 dB |
| Detecção de bordas (Canny) | limiares 50/130 sobre imagem pré-suavizada |
| Segmentação (Otsu) | limiarização automática por grayscale |
| Segmentação (ExG) | threshold físico fixo em 0.0 |
| Morfologia matemática | `MORPH_OPEN` + `MORPH_CLOSE` (kernel 7×7, 2 iterações) |
| Análise multi-imagem | pipeline encapsulado aplicado a 3 imagens |
| Métricas | PSNR, SNR, percentual de área segmentada |

---

## Resultados

### Visualização comparativa dos índices

![Comparativo VARI e ExG](outputs/comparativo_indices.png)

### Comparação de filtros com PSNR

![Comparação de filtros](outputs/comparacao_filtros.png)

| Filtro | PSNR (dB) | Custo relativo |
|---|---|---|
| Gaussiano σ=1.5 | 25.39 | 1× |
| Mediana 5×5 | 26.12 | 1.1× |
| Bilateral d=9 | 32.01 | 1.9× |

O **bilateral** apresenta maior PSNR porque preserva bordas — afasta menos a imagem do original mesmo suavizando ruído. O **Gaussiano** tem menor PSNR por borrar indiscriminadamente.

### Segmentação: Otsu (grayscale) vs. ExG

![Comparação de segmentação](outputs/comparacao_segmentacao.png)

| Método | Threshold | Cobertura | Regiões |
|---|---|---|---|
| Otsu (grayscale) | 0.53 | 13% | 259 |
| ExG > 0 (fixo) | 0.0 | 82% | 150 |

Otsu segmenta por **brilho** — não distingue verde de outras superfícies igualmente iluminadas. ExG segmenta por **assinatura espectral de crominância**, detectando especificamente a dominância de verde sobre vermelho e azul.

### Análise multi-imagem

![Análise multi-imagem](outputs/analise_multi_imagem.png)

| Imagem | ExG (veg%) | VARI médio | Interpretação |
|---|---|---|---|
| WhatsApp 17:57 | 8.86% | −0.1365 | baixa cobertura vegetal |
| WhatsApp 18:00 | 97.08% | −0.0016 | divergência ExG/VARI — possível falso positivo |
| paisagem-verde | 85.20% | +0.2453 | ambos os índices concordam — resultado mais confiável |

### Histogramas

| ExG | Canais R, G, B |
|---|---|
| ![Histograma ExG](outputs/histograma_exg.png) | ![Histograma RGB](outputs/histograma_rgb.png) |

---

## Estrutura do projeto

```
cv-vegetation-indices/
├── images/                  # imagens originais capturadas
├── notebooks/
│   └── analise_indices_vegetacao.ipynb   # pipeline completo
├── outputs/                 # todas as imagens geradas
│   ├── grid_pipeline_completo.png
│   ├── comparativo_indices.png
│   ├── comparacao_filtros.png
│   ├── comparacao_segmentacao.png
│   ├── analise_multi_imagem.png
│   ├── segmentacao_exg.png
│   ├── histograma_exg.png
│   └── histograma_rgb.png
└── README.md
```

---

## Como executar

**Requisitos:** macOS/Linux, Python 3.10+

```bash
# 1. Clonar o repositório
git clone https://github.com/GilbertoPaiva/cv-vegetation-indices.git
cd cv-vegetation-indices

# 2. Criar e ativar ambiente virtual
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 3. Instalar dependências
pip install opencv-python numpy matplotlib pillow scikit-image jupyterlab

# 4. Iniciar Jupyter
jupyter lab
```

Abra `notebooks/analise_indices_vegetacao.ipynb` e execute todas as células em ordem (`Kernel → Restart & Run All`).

---

## Dependências

| Pacote | Versão testada |
|---|---|
| Python | 3.11.9 |
| opencv-python | 4.13.0 |
| numpy | 2.4.4 |
| matplotlib | 3.10.9 |
| pillow | 12.2.0 |
| scikit-image | 0.26.0 |
| jupyterlab | 4.5.7 |

---

## Referências

- GITELSON, A. A. et al. *Novel algorithms for remote estimation of vegetation fraction*. Remote Sensing of Environment, 2002. — origem do VARI
- WOEBBECKE, D. M. et al. *Color indices for weed identification under various soil, residue, and lighting conditions*. Transactions of the ASAE, 1995. — origem do ExG
- GONZALEZ, R. C.; WOODS, R. E. *Processamento Digital de Imagens*. 3. ed. Pearson, 2010.
- BRADSKI, G.; KAEHLER, A. *Learning OpenCV 4*. O'Reilly Media, 2019.
- OpenCV Documentation: https://docs.opencv.org/4.x/
- NumPy Documentation: https://numpy.org/doc/stable/
- scikit-image Documentation: https://scikit-image.org/docs/stable/
