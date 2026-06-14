# Artigo A3 — LaTeX (Overleaf)

Relatório técnico completo da Avaliação A3, preenchido com os resultados reais do projeto.

## Conteúdo

```
artigo/
├── main.tex        # documento LaTeX completo
├── figuras/        # 10 figuras geradas pelo notebook
└── main.pdf        # PDF compilado (15 páginas)
```

## Como abrir no Overleaf

1. Acesse [overleaf.com](https://www.overleaf.com) e faça login
2. **New Project → Upload Project**
3. Compacte a pasta `artigo/` em `.zip` e envie — OU crie um projeto em branco e suba `main.tex` + a pasta `figuras/`
4. Em **Menu → Compiler**, selecione **pdfLaTeX**
5. Clique em **Recompile**

> O documento usa apenas pacotes padrão (babel, graphicx, booktabs, amsmath, hyperref), então compila no Overleaf sem configuração extra.

## Identificação do autor

O bloco `\author{...}` no início do `main.tex` já está preenchido:

```latex
\author{
  Gilberto de Paiva Melo \\
  \small Curso de Ciência da Computação --- Computação Gráfica \\
  \small Turno: Noturno \\
  \small Avaliação A3 --- Semestre 2026.1
}
```

Não há campos de matrícula ou turma a preencher.

## Estrutura do artigo (13 páginas)

1. Resumo + palavras-chave
2. Introdução e contextualização
3. Objetivos (geral e específicos)
4. Fundamentação teórica (RGB, HSV, VARI/ExG/ExGR, filtros, Canny, Otsu, morfologia, PSNR/SNR)
5. Metodologia (ferramentas e pipeline)
6. Resultados e discussão (metadados, histogramas, índices, filtros, segmentação, multi-imagem)
7. Análise crítica
8. Conclusão
9. Referências bibliográficas

Todos os números e figuras vêm da execução real do notebook `notebooks/analise_indices_vegetacao.ipynb`.
