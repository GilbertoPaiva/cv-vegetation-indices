# Roteiro de Apresentação — A3 Computação Gráfica (breve)

## 1. Abertura (30s)

> "O olho humano vê o verde fácil, mas o computador só vê números. Meu projeto pega uma
> imagem aérea RGB comum e responde objetivamente: **quais pixels são vegetação e qual o
> percentual de cobertura** — sem sensor especializado."

**Imagens:** 2 fotos de drone de alta resolução baixadas de bancos abertos (modalidade
prevista na seção 5.1 da proposta): a **principal** (`campo-fazenda`, 19,9 MP — campo,
estrada e céu, onde rodo o pipeline inteiro) e o **caso difícil** (`lavoura-solo`, 11,9 MP
— solo exposto que engana o índice). Testei celular no início, mas o ângulo oblíquo
degradava os índices — está registrado no artigo.

## 2. Pipeline em 6 passos (2 min) — seção 5.3 do artigo

1. **Carregar** (`cv2.imread`, BGR→RGB) e inspecionar metadados: 5464×3640, uint8, média RGB [97, 118, 57] → verde domina
2. **Pré-processar**: redimensionar 19,9 MP → 1600px (−91% de dados), escala de cinza, normalizar [0,1]
3. **Histogramas** R/G/B e do índice ExG
4. **Índices**: VARI = (G−R)/(G+R−B) com máscara de sombra; ExG = 2G−R−B (vegetação se > 0)
5. **Filtros** (Gaussiano/Mediana/Bilateral, medidos por PSNR) e **Canny** (3 pares de limiares)
6. **Segmentar**: Otsu vs. ExG + morfologia; depois a versão aprimorada ExGR + trava HSV

## 3. Os 4 números para decorar

| Resultado | Número | Frase pronta |
|---|---|---|
| **Otsu vs. ExG** | 17% vs. 83% | "O Otsu, por brilho, marcou a **estrada de terra** — exatamente o que NÃO é vegetação. O ExG, por cor, marcou o campo. Cor vence brilho." |
| **Filtros (PSNR)** | Bilateral 34,1 dB | "O bilateral é o mais fiel porque suaviza **preservando bordas**." |
| **Resolução do caso difícil** | lavoura: 97% → 40,2% | "O ExG dizia 97% numa lavoura cheia de solo — e o VARI negativo (−0,07) denunciava. ExGR + trava HSV + fechamento morfológico resolve: 40,2% de vegetação real (a cultura verde preservada, o solo fora). Histórico mantido na figura." |
| **Generalização (petróleo)** | mancha: 7,1% → **17,4%** | "Primeiro detectei só por brilho — mas pegava nuvens. A assinatura real do óleo é a **cor quente** (R−B≈20): trocando brilho por cor, a mancha da **Deepwater Horizon** (imagem NASA) vira uma região coerente de 17,4%. Mesma lição da vegetação." |
| **Generalização (estrelas)** | **14.616 estrelas** | "A ideia original: telescópio. Na imagem **Hubble** do M13, a assinatura é o brilho. O top-hat morfológico pega até as estrelas fracas e o **mesmo** connectedComponents que filtrava área agora **conta** 14.616 estrelas. A contribuição é a metodologia." |

## 4. Perguntas prováveis — respostas curtas

- **"Por que painéis pretos?"** → Canny e máscaras binárias são preto-e-branco por natureza: preto = "sem borda"/"não segmentado".
- **"Por que imagens da internet?"** → A proposta (5.1) permite download documentado; documentei busca, seleção (3 critérios) e download no artigo.
- **"VARI vs. ExG?"** → ExG é mais simples (limiar 0); VARI é normalizado e resiste à iluminação, mas explode na sombra (resolvido com máscara de luminância). **Quando concordam, confio; quando divergem, é alerta de falso positivo.**
- **"Por que HSV?"** → A Saturação separa cor viva de cinza: céu e solo claro têm saturação baixa, então a trava S≥40 os elimina.
- **"Limiares do Canny?"** → 30/90 pega textura demais (1,4% de bordas); 150/250 fragmenta (0,04%); 50/130 equilibra (0,7%).

## 5. Glossário rápido

- **Outlier** — valor absurdo por erro de conta (divisão por ~zero na sombra); cortamos com clip + máscara
- **Destaque por dessaturação** — a região detectada fica em COR e o resto vira cinza: o que está colorido é exatamente o que o algoritmo classificou (nada é pintado por cima)
- **Trava HSV** — regra que só aceita pixel de cor realmente verde e saturada (no petróleo, invertida: dessaturado e claro)
- **Morfologia (abrir/fechar)** — faxina na máscara: tira pontinhos, tapa buraquinhos
- **Nadir** — foto vertical de cima (drone): cada pixel = uma superfície só
- **Sunglint** — reflexo do sol no mar; o óleo alisa a água e brilha prateado nessa região

## 6. Se pedirem para executar

```bash
cd ~/DevSpace/cv-vegetation-indices
source .venv/bin/activate
jupyter lab
# abrir notebooks/analise_indices_vegetacao.ipynb
# Kernel → Restart Kernel and Run All Cells  (~50 s)
```

Alternativa segura: o notebook já está salvo **com todas as saídas**; basta abrir e rolar.
O PDF do artigo está em `artigo/main.pdf` (21 páginas) — a seção 5.3 tem o pipeline
numerado para narrar.

## 7. Slides da apresentação

```bash
cd ~/DevSpace/cv-vegetation-indices
python3 -m http.server 8765
# abrir http://127.0.0.1:8765/site/  (19 slides; ← → navega; tecla D alterna claro/escuro)
```

## 8. Fechamento (15s)

> "Três conclusões: **cor vence brilho** (na vegetação e no óleo); **índices que divergem
> denunciam erros** — e nós resolvemos o caso difícil; e **a contribuição é a metodologia**:
> os mesmos blocos mediram % de vegetação, delimitaram a mancha de óleo da Deepwater Horizon
> (NASA) e contaram 14.616 estrelas no aglomerado M13 (Hubble)."
