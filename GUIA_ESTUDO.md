# Guia de Estudo — A3 Computação Gráfica
### Processamento Digital de Imagens com Python

Este guia é para **aprender de verdade** o que o projeto faz e chegar seguro na apresentação.
Leia na ordem. Cada seção tem: **o que é**, **por que usamos** e **como explicar**.

> **A história em uma frase:** comecei tentando medir *quanto de vegetação* existe numa imagem;
> a cada problema tomei uma decisão; no fim percebi que tinha construído uma **metodologia geral
> de detecção** que serve para vegetação, manchas de óleo e até contar estrelas.

---

## 0. As ideias-base (entenda isto e o resto fica fácil)

### O que é uma imagem digital?
Uma grade de **pixels**. Cada pixel colorido guarda 3 números: **R** (vermelho), **G** (verde),
**B** (azul), de 0 a 255. O computador não "vê" verde — vê números. Todo o projeto é transformar
esses números numa resposta (ex.: "63% da imagem é vegetação").

### Os 5 blocos que se repetem o projeto inteiro (a "receita")
Decore esta sequência — ela é a espinha de tudo:

1. **Assinatura** — o que torna meu alvo diferente do fundo? (cor verde? cor quente? brilho?)
2. **Limiar absoluto** — uma regra de corte com significado físico (ex.: "verde dominante > 0").
3. **Morfologia** — uma "faxina" na máscara (tira pontinhos, une regiões, tapa buracos).
4. **Componentes conexos** — agrupa pixels vizinhos em regiões (para medir ou contar).
5. **Medida** — % de cobertura, % de área, ou contagem de objetos.

A grande sacada do projeto: **essa receita não depende de ser verde**. Só muda a *assinatura*.

---

## 1. Aquisição — "a imagem é parte do algoritmo"

**O que fiz:** comecei com **foto de celular**, mas o ângulo torto (oblíquo) e a baixa resolução
**estragavam** os índices. Troquei por **imagens de drone de alta resolução** (visada reta de
cima = *nadir*), baixadas de bancos abertos — modalidade que a própria proposta permite (seção
5.1), com a origem documentada.

- **Por que importa:** numa foto de cima (nadir), cada pixel é **uma superfície só** (só campo,
  só estrada). Na foto torta, um pixel mistura várias coisas → confunde o algoritmo.
- **Como explicar:** *"A escolha da imagem é parte do algoritmo: visada reta e alta resolução
  melhoram o resultado tanto quanto qualquer filtro."*

**Metadados (mostrar na imagem principal):** 5464×3640 px (19,9 MP), 3 canais, `uint8` (0–255),
~3 MB. Média de cor R=97, G=118, B=57 → **o verde já domina nos números**, antes de qualquer conta.

---

## 2. Pré-processamento

- **Redimensionar:** a imagem de 19,9 MP é pesada; reduzo para 1600 px de largura para trabalhar
  rápido (a análise roda na versão menor; os metadados guardam a resolução original).
- **Escala de cinza** (`0.299R + 0.587G + 0.114B`): **descoberta-chave** → na versão cinza o campo
  verde, a mata e o solo ficam *parecidos*. Ou seja, **brilho não identifica vegetação** — é a cor
  que define. Isso motiva os índices das etapas seguintes.
- **Normalizar [0,1]:** dividir por 255 para as contas dos índices não estourarem.

---

## 3. Histograma e equalização

- **Histograma** = gráfico de quantos pixels têm cada intensidade. Mostra que o verde se desloca
  para valores mais altos; o índice ExG se concentra acima de 0 (cena vegetada).
- **Equalização de histograma** = técnica de **realce de contraste**: espalha os tons por toda a
  faixa. Equalizei só a **luminância** (para não distorcer a cor). O contraste sobe (desvio-padrão
  do brilho 49 → 72).
- **Conexão esperta:** equalização é um realce *global* (uma curva para a imagem toda). Seu primo
  *local*, o **top-hat**, reaparece lá no fim para revelar estrelas fracas. Mesma família, escalas
  diferentes.

---

## 4. Índices de vegetação (a "assinatura" do verde)

Folha saudável reflete verde e absorve vermelho. Os índices exploram isso:

- **ExG = 2G − R − B** ("excesso de verde"). Simples. `ExG > 0` → vegetação.
- **VARI = (G − R) / (G + R − B)**. Normalizado de −1 a +1; resiste a variações de iluminação.
- **ExGR = 3G − 2.4R − B**. Desconta também o vermelho (ótimo para tirar solo).

**Problema dos *outliers* no VARI:** em pixels de **sombra**, o denominador quase zera e a conta
*explode* (valores absurdos, tipo 144 mil). Solução: **máscara de luminância** — ignoro pixels
muito escuros. A média do VARI quase não muda → confirma que eram só ruído de conta.

- **Outlier (decorar):** *"valor fora da curva, sem sentido físico, vindo de erro numérico — eu
  corto com clipping e máscara."*

---

## 5. Filtros de suavização (com PSNR/SNR)

Três filtros que reduzem ruído:
- **Gaussiano:** borra tudo por igual.
- **Mediana:** troca o pixel pela mediana da vizinhança; bom contra ruído "sal e pimenta".
- **Bilateral:** suaviza **preservando as bordas** → o mais fiel.

**PSNR e SNR** medem, em decibéis, o quão fiel a imagem filtrada ficou à original (**maior =
melhor**). Resultado: **Bilateral 34,1 dB** ganha (preserva bordas). Como a imagem aérea tem pouco
ruído, os três ficam próximos (32,8–34,1 dB).

---

## 6. Detecção de bordas — Canny

Acha os **contornos**. Usa **histerese** (dois limiares t1/t2): borda forte é aceita; borda fraca
só é aceita se estiver ligada a uma forte.
- 30/90 (baixo) → pega até textura, fica ruidoso (1,36% de bordas).
- **50/130 (adotado)** → equilíbrio (0,71%).
- 150/250 (alto) → só contornos fortes, fragmenta (0,04%).

**Atenção (pergunta provável):** o Canny dá uma imagem **preta com linhas brancas** — preto = "sem
borda", **não é erro**.

---

## 7. A GRANDE DESCOBERTA — Otsu (brilho) × ExG (cor)

- **Otsu:** escolhe um limiar **automático** pelo histograma, mas só pelo **brilho**.
- **Resultado surpreendente:** o Otsu segmentou a **estrada de terra clara** (17%, em 3 regiões) —
  exatamente o que **NÃO** é vegetação! O **ExG** (cor) pegou o **campo verde** (83%, 14 regiões).
- **Lição nº 1 (a mais importante):** **cor (crominância) vence brilho (luminância)**. O brilho
  pode até segmentar o alvo errado.

> Esta é a hora de pausar na apresentação: *"o algoritmo por brilho marcou a estrada; o por cor
> marcou o campo."* É o clímax da parte de vegetação.

---

## 8. O caso difícil e o refino — lavoura com solo

- **Problema:** o `ExG > 0` ingênuo dizia **97%** de vegetação numa lavoura **cheia de solo
  marrom** (falso positivo). O **VARI discordava** (média negativa, −0,07) → **alarme**.
- **Lição nº 2:** *"quando os dois índices divergem, há um erro escondido."*
- **Solução (segmentação aprimorada):**
  - **ExGR** descarta o solo avermelhado (solo tem ExGR < 0).
  - **Trava HSV** (matiz verde + saturação ≥ 40) rejeita céu/cinza.
  - **Detalhe decisivo da morfologia:** a plantação são **fileiras finas** de verde separadas por
    sulcos de terra. Se eu *abrir* a máscara primeiro, **apago as fileiras** (foi o erro: dava só
    15%). Invertendo — **fechar primeiro** (une as fileiras num bloco) e depois abrir suave —
    recupero a cultura: **40,2%** real.
- **Resultado:** lavoura 97% → **40,2%** (solo fora, cultura preservada); campo → **63,8%**.

- **HSV (decorar):** *"outro jeito de descrever cor — Matiz, Saturação, Valor. A saturação separa
  cor viva de cinza: céu e solo claro têm saturação baixa, então a trava os elimina."*

---

## 9. A VIRADA — não é sobre verde, é a metodologia

Aqui o projeto cresce. A mesma receita (assinatura → limiar → morfologia → componentes → medida)
serve para outros alvos. Provei em dois domínios reais:

### 9a. Petróleo — mancha de óleo (imagem NASA/MODIS, Deepwater Horizon, 2010)
- **Mesma lição cor-vence-brilho, de novo!** Primeiro tentei por **brilho** (a mancha é clara no
  reflexo do sol) — mas pegava **nuvens** (também claras). A assinatura **real** do óleo é a **cor
  quente (tan)**: vermelho bem maior que azul (**R − B ≈ 20–25**); água e nuvem são neutras (≈ 0).
- Trocando brilho por cor quente → mancha vira **uma região coerente de 17,4%**.
- **Aplicações reais:** monitorar derrames, ver vegetação invadindo faixa de dutos, detectar
  vazamento pela vegetação estressada perto do duto.

### 9b. Astronomia — contar estrelas (imagem Hubble do aglomerado M13)
- Assinatura agora = **brilho** (estrela = ponto claro no céu preto).
- O limiar global pegava poucas (7.736). A versão final reusa **duas peças do projeto**: o
  **top-hat** (realce *local* — primo da equalização) revela as estrelas fracas, e o **mesmo Otsu**
  da vegetação escolhe o corte automático. O **mesmo** `connectedComponentsWithStats` do filtro de
  área agora **conta**: **14.616 estrelas** (quase o dobro).
- Aqui a medida não é área, é **contagem de objetos**.

---

## 10. Síntese (o slide que amarra tudo)

| | Vegetação | Petróleo | Astronomia |
|---|---|---|---|
| Assinatura | verde (ExG/ExGR) | cor quente (R−B) | brilho |
| Limiar | absoluto / HSV | absoluto (R−B≥14) | Otsu automático |
| Mede | % cobertura | % área da mancha | contagem |
| Resultado | 63,8% / 40,2% | 17,4% | 14.616 estrelas |
| Limite honesto | cultura seca | sedimento vs. óleo | núcleo denso |

**A meta-lição honesta:** *toda detecção por limiar tem um regime onde sinal e fundo se
confundem* — o núcleo do aglomerado, o sedimento na água, a cultura seca na lavoura. É a mesma
limitação aparecendo em três caras.

---

## 11. As 4 conclusões (o que dizer no fim)

1. **A assinatura certa vence o brilho** (vegetação e óleo provaram isso).
2. **Testes absolutos > relativos** (e a ordem da morfologia importa).
3. **A imagem é parte do algoritmo** (celular → drone).
4. **A contribuição é a metodologia, não o alvo** (vegetação, óleo, estrelas).

---

## 12. Perguntas prováveis (e a resposta curta)

- **"Por que não usou foto sua / capturada por você?"** → A proposta (5.1) permite **download
  documentado** de fontes abertas; documentei origem, licença e critérios de seleção de cada
  imagem (tabela no relatório).
- **"Por que tem painéis pretos?"** → Canny e máscaras binárias são preto-e-branco por natureza
  (preto = "sem borda" / "fora da máscara"). Não é erro.
- **"VARI ou ExG?"** → ExG é mais simples (corte em 0); VARI resiste à iluminação mas estoura na
  sombra (resolvido com máscara). **Quando concordam, confio; quando divergem, é alerta.**
- **"O número de estrelas é exato?"** → Não — é um **limite inferior**; no núcleo denso elas se
  fundem. Honesto e esperado.
- **"Isso serve pra quê na vida real?"** → Agricultura de precisão, desmatamento, resposta a
  derrames de óleo, e até astronomia. A metodologia é reaproveitável.

---

## 13. Glossário relâmpago

| Termo | Em 1 frase |
|---|---|
| Pixel | Menor ponto da imagem; guarda R, G, B. |
| Normalizar | Dividir por 255 para os valores virarem fração 0–1. |
| Índice (ExG/VARI) | Conta com R,G,B que dá alto onde há planta. |
| Outlier | Valor absurdo por erro de conta; a gente corta. |
| Limiar (threshold) | Número de corte ("acima disso é o alvo"). |
| Otsu | Método que escolhe o limiar **automático**, só pelo brilho. |
| HSV | Cor descrita por Matiz, Saturação, Valor. |
| Trava HSV | Regra que só aceita cor realmente viva (não cinza). |
| Morfologia (abrir/fechar) | Faxina na máscara: tira pontinhos, tapa buracos. |
| Componentes conexos | Agrupa pixels vizinhos em regiões (medir/contar). |
| Top-hat | Realce **local** de pontos claros pequenos (revela estrelas fracas). |
| PSNR/SNR | Quão fiel ficou a imagem filtrada (maior = melhor). |
| Nadir | Foto reta de cima; cada pixel = uma superfície. |
| Sunglint | Reflexo do sol no mar; o óleo brilha prateado ali. |

---

## 14. Como rodar (se precisar mostrar)

**Slides (a apresentação):**
```bash
cd ~/DevSpace/cv-vegetation-indices
python3 -m http.server 8765
# abrir http://127.0.0.1:8765/site/  → setas ← →, tecla D = claro/escuro
```

**Notebook (o código):**
```bash
source .venv/bin/activate && jupyter lab
# abrir notebooks/analise_indices_vegetacao.ipynb → Kernel → Restart & Run All (~1 min)
```

O relatório está em `artigo/main.pdf`. Você pode apresentar **só pelas imagens** (slides + PDF),
sem mostrar código.
