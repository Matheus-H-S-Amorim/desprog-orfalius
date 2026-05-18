# Transformada de Hough

++++++++++++++++++++++++++++++++++++++++++
## Desafios de Programação

**Transformada de Hough**

Detecção de retas em imagens

++++++++++++++++++++++++++++++++++++++++++

## Motivação: detecção de faixas

Carros autônomos precisam descobrir automaticamente onde estão as faixas da estrada.

![Imagem de estrada com contornos destacados](edge_points.png)

??? Checkpoint
Observe a imagem acima.

Quais partes da cena podem ser aproximadas por retas?

Tente identificar, visualmente:
- as faixas da pista;
- bordas da estrada;
- estruturas lineares ao fundo.

O que fica mais difícil quando a imagem tem ruído ou partes ocluídas?
???

## A entrada não é uma imagem

Antes da Transformada de Hough, um detector de bordas, como o Canny, extrai os contornos da cena. O resultado é um **conjunto de pontos** $(x, y)$.

A Transformada de Hough recebe **apenas esses pontos**, não a imagem original.

??? Checkpoint
Olhe para a imagem de bordas acima.

Se você só pudesse enxergar os pontos acesos, o que mudaria na forma de pensar o problema?
???

## Por que não é trivial?

A abordagem mais direta seria: para cada par de pontos, traçar a reta que passa por eles e contar quantos outros pontos estão sobre ela. A reta com mais pontos seria a detectada.

??? Checkpoint
Suponha que você tem $N$ pontos. Quantos pares de pontos existem? Qual seria a complexidade dessa abordagem?
::: Gabarito
O número de pares é $\binom{N}{2} = \frac{N(N-1)}{2}$, o que dá complexidade $O(N^2)$.

Para imagens reais, $N$ pode ser da ordem de dezenas de milhares de pixels de borda. Testar todos os pares é inviável.
:::
???

Mas há um segundo problema, mais sutil. Mesmo que o custo fosse aceitável, a abordagem quebraria na prática porque pontos de borda reais nunca estão perfeitamente sobre uma reta. Ruído, variações de iluminação e oclusões fazem cada ponto desviar levemente da posição ideal.

![Pontos ideais e pontos com ruído](ruido_reta.png)

??? Checkpoint
Observe a figura acima.

Por que dois pontos com um pequeno erro podem gerar uma reta bem diferente da reta ideal?
???

Esses dois problemas — custo quadrático e sensibilidade ao ruído — motivam a Transformada de Hough.

## A ideia central: mudando de perspectiva

Toda reta pode ser escrita como $y = ax + b$, com parâmetros $a$ (inclinação) e $b$ (intercepto).

A observação-chave é: se fixarmos um ponto $(x_0, y_0)$ e perguntarmos "quais retas passam por ele?", obtemos:

$$b = -x_0 \cdot a + y_0$$

Isso é **uma reta no espaço de parâmetros** $(a, b)$.

![Espaço da imagem e espaço de parâmetros](image_vs_params.png)

??? Checkpoint
Dado o ponto $P = (2, 3)$, escreva a relação entre $a$ e $b$ para qualquer reta $y = ax + b$ que passe por $P$.
::: Gabarito
Substituindo $x = 2$ e $y = 3$:

$$3 = a \cdot 2 + b \implies b = -2a + 3$$

Essa é uma reta no espaço de parâmetros. Cada par $(a, b)$ sobre essa reta representa uma reta da imagem que passa por $P$.
:::
???

Agora vem a parte mais importante: o que acontece quando temos dois pontos?

??? Checkpoint
Considere dois pontos: $P_1 = (1, 1)$ e $P_2 = (3, 3)$.

Escreva a equação da reta no espaço de parâmetros correspondente a cada ponto. Em seguida, encontre o ponto de interseção $(a^*, b^*)$ das duas retas.

O que esse ponto representa no espaço da imagem?
::: Gabarito
Para $P_1 = (1, 1)$: $b = -a + 1$

Para $P_2 = (3, 3)$: $b = -3a + 3$

Igualando:

$$-a + 1 = -3a + 3 \implies 2a = 2 \implies a = 1$$

Logo, $b = 0$.

O ponto de interseção é $(a^*, b^*) = (1, 0)$, que corresponde à reta $y = x$.

**Conclusão:** dois pontos colineares no espaço da imagem geram duas retas que se cruzam num ponto no espaço de parâmetros. Esse ponto de cruzamento *é* a reta detectada.
:::
???

## Generalizando: vários pontos com ruído

Com muitos pontos, cada um gera uma reta no espaço de parâmetros. Se todos fossem perfeitamente colineares, todas as retas se cruzariam num único ponto exato. Mas, na prática, pontos reais têm ruído: as retas não se cruzam em um ponto único, e sim em uma região densa de cruzamentos próximos.

![Pontos com ruído e acumulador](pontos_ruido_acumulador.png)

Para lidar com isso, o algoritmo discretiza o espaço de parâmetros numa grade e usa **votação**. Cada ponto incrementa a célula $(a, b)$ correspondente a cada reta que passa por ele. O resultado é chamado de **acumulador**. Ao final, os picos do acumulador são as retas detectadas.

??? Checkpoint
Observe a imagem acima.

- Por que o pico aparece mesmo com ruído?
- Por que os pontos fora da reta não destroem o resultado?
- O que aconteceria se a grade do acumulador fosse muito grossa?
???

??? Checkpoint
Considere os pontos abaixo, sendo 4 aproximadamente colineares e 2 de ruído puro:

- $A = (0, 1.0)$, $B = (2, 3.1)$, $C = (4, 4.9)$, $D = (6, 7.2)$
- $R_1 = (1, 5)$, $R_2 = (5, 1)$

No acumulador, a região em torno de qual ponto $(a^*, b^*)$ deve acumular mais votos?
::: Gabarito
A reta ideal que contém $A$–$D$ é $y = x + 1$, com $a^* = 1$ e $b^* = 1$.

Como os pontos têm ruído, as retas no espaço de parâmetros não se cruzam num ponto único; elas se cruzam numa região densa em torno de $(1, 1)$. O acumulador soma esses votos próximos, e o pico emerge dessa concentração.
:::
???

## O problema da equação clássica

Até aqui usamos $y = ax + b$ para representar retas. Essa parametrização tem um ponto cego: não consegue representar **retas verticais**.

Uma reta vertical tem a forma $x = c$. Para expressá-la como $y = ax + b$, precisaríamos de $a \to \infty$, um valor inválido para indexar o acumulador.

A solução é usar a **forma polar**:

$$r = x \cos\theta + y \sin\theta$$

onde $r$ é a distância perpendicular da reta à origem e $\theta$ é o ângulo da normal à reta com o eixo $x$.

![Curvas no espaço polar](polar_transform.png)

??? Checkpoint
Na representação $(a, b)$, fixar um ponto $(x_0, y_0)$ gerava uma reta no espaço de parâmetros. Na forma polar, o mesmo ponto gera a curva

$$r = x_0 \cos\theta + y_0 \sin\theta$$

Por que essa curva é uma sinusoide e não uma reta?
???
::: Gabarito
Na equação $b = -x_0 a + y_0$, os parâmetros aparecem de forma linear, por isso a curva é uma reta no espaço $(a, b)$.

Na equação $r = x_0 \cos\theta + y_0 \sin\theta$, o parâmetro $\theta$ aparece dentro de funções trigonométricas, por isso a curva é uma sinusoide no espaço $(r, \theta)$.

A ideia continua a mesma: pontos colineares geram curvas que se cruzam num ponto. A geometria das curvas muda, mas o princípio de votação é idêntico.
:::

## O acumulador: votação na prática

O acumulador é uma matriz onde cada linha representa um valor discreto de $r$ e cada coluna representa um ângulo $\theta_j$.

O processo para cada ponto de borda $(x, y)$ é:

- para cada ângulo $\theta_j$, calcule $r = x \cos\theta_j + y \sin\theta_j$;
- incremente a célula correspondente no acumulador.

![Pico no acumulador polar](acumulador_polar.png)

Ao final, os picos do acumulador são as retas detectadas.

??? Checkpoint
Por que é necessário usar um acumulador com votação em vez de simplesmente calcular a interseção exata das curvas?
???
::: Gabarito
Se os pontos fossem perfeitamente colineares, as curvas se cruzariam num único ponto exato.

Na prática, pontos com ruído geram curvas levemente deslocadas. Elas não se cruzam num ponto único, mas numa região densa de cruzamentos próximos. O acumulador soma esses votos célula a célula, e o pico emerge mesmo com os votos espalhados numa pequena vizinhança.

Isso torna o algoritmo robusto: ele não exige perfeição dos dados.
:::

!!! Aviso
O tamanho do acumulador afeta tanto a precisão quanto o custo. Uma grade fina detecta retas com mais exatidão, mas ocupa mais memória e leva mais tempo para ser percorrida.
!!!

## Eficiência

O algoritmo tem dois laços encadeados: para cada um dos $N$ pontos de borda, percorre todos os $M$ ângulos discretizados e incrementa uma célula do acumulador.

??? Checkpoint
Com base nessa descrição, qual é a complexidade do preenchimento do acumulador?
::: Gabarito
São $N$ pontos, cada um disparando $M$ incrementos: custo $O(N \times M)$.

Para $M = 180$ ângulos fixos, o custo é aproximadamente linear em $N$, muito melhor que $O(N^2)$.
:::
???

Após o preenchimento, ainda é necessário varrer o acumulador para encontrar os picos. Se o acumulador tem $R \times M$ células, essa busca tem custo $O(R \times M)$.

??? Checkpoint
O custo total do algoritmo é $O(N \times M + R \times M)$. Para imagens com muitas bordas, qual termo tende a dominar? Por quê?
::: Gabarito
$O(N \times M)$ tende a dominar quando $N$ é grande, o que é comum em imagens reais.

Como $M$ e $R$ são definidos pela resolução do acumulador, o custo efetivo é linear em $N$ para uma discretização fixa.
:::
???

## Resumo

A Transformada de Hough detecta retas em conjuntos de pontos por meio de três ideias encadeadas:

1. **Dualidade ponto–reta:** um ponto no espaço da imagem vira uma curva no espaço de parâmetros, e uma reta no espaço da imagem vira um ponto no espaço de parâmetros.
2. **Votação:** cada ponto de borda vota de forma independente em todos os pares $(r, \theta)$ compatíveis com ele. Pontos colineares concentram votos numa mesma região do acumulador.
3. **Detecção de picos:** os picos do acumulador correspondem às retas presentes na imagem.

O resultado é um algoritmo $O(N \times M)$, eficiente e robusto a ruído e oclusões.

## Desafio: detecção de círculos

A mesma lógica se estende para círculos. Um círculo é definido por três parâmetros: centro $(a, b)$ e raio $r$.

![Ideia da extensão para círculos](circulos_plot.png)

Se o raio já é conhecido, cada ponto de borda vota em todos os possíveis centros que estariam à distância $r$ dele. O acumulador vira 2D $(a, b)$.

Se o raio é desconhecido, o acumulador vira 3D $(a, b, r)$ e o custo sobe rapidamente.

??? Desafio
Suponha que você sabe que o raio dos círculos que quer detectar está entre 10 e 50 pixels, e a imagem tem resolução $100 \times 100$. O número de pontos de borda é $N = 500$.

Estime o número de operações no pior caso para o acumulador 3D. Isso é viável?
::: Gabarito
$R = 50 - 10 = 40$ valores de raio. O custo seria:

$$500 \times 100 \times 100 \times 40 = 200{,}000{,}000$$

Duzentos milhões de operações. Para imagens maiores ou ranges de raio mais amplos, isso cresce rapidamente. Na prática, limitar o range de raios e usar o gradiente de borda reduz bastante o custo.
:::
???