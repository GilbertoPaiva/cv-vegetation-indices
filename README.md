# Projeto A3 — Processamento de Imagens

Ambiente de desenvolvimento local para processamento de imagens com Python, configurado para Apple Silicon M1.

## Requisitos

- macOS com Apple Silicon (arm64)
- Homebrew
- pyenv

## Estrutura

```
projeto-a3/
├── images/      # imagens originais capturadas
├── outputs/     # imagens geradas pelo processamento
├── notebooks/   # jupyter notebooks
├── src/         # código Python modular
└── README.md
```

## Setup

```bash
# Ativar o ambiente virtual
source .venv/bin/activate

# Iniciar Jupyter Lab
jupyter lab
```

## Pacotes instalados

| Pacote | Versão |
|--------|--------|
| opencv-python | 4.13.0 |
| numpy | 2.4.4 |
| matplotlib | 3.10.9 |
| pillow | 12.2.0 |
| scikit-image | 0.26.0 |
| jupyter / jupyterlab | 1.1.1 / 4.5.7 |

## Python

- Versão: 3.11.9 (gerenciado via pyenv)
- Venv: `.venv/`
