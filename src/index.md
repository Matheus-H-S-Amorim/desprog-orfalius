# Transformada de Hough

++++++++++++++++++++++++++++++++++++++++++ Desafios de Programação

**Transformada de Hough**

Detecção de retas em imagens

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ Motivação: detecção de faixas

Carros autônomos precisam saber onde estão as faixas da estrada.

![Exemplo de imagem de estrada após detector de bordas](image.png)

Como ensinar o computador a encontrar as retas nessa imagem?

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ A entrada não é uma imagem

Antes da Transformada de Hough, um detector de bordas (como o Canny) extrai os contornos da cena. O resultado é um **conjunto de pontos** $(x, y)$.

A Transformada de Hough recebe **apenas esses pontos** não a imagem original.

O objetivo é encontrar as retas que melhor explicam onde esses pontos estão.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ Por que não é trivial?

**Abordagem ingênua:** para cada par de pontos, traçar a reta e contar quantos outros pontos estão sobre ela.

* Com $N$ pontos há $\binom{N}{2}$ pares → $O(N^2)$. Para dezenas de milhares de pixels, inviável.

* Na vida real os pontos **nunca são perfeitamente colineares** ruído, iluminação e oclusões fazem os pontos desviarem da reta ideal. Um desvio de 0.1 pixel já produz uma reta completamente diferente.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ A ideia central: mudando de perspectiva

Toda reta pode ser escrita como $y = ax + b$, com parâmetros $a$ (inclinação) e $b$ (intercepto).

**Observação-chave:** se fixarmos um ponto $(x_0, y_0)$ e perguntarmos "quais retas passam por ele?", obtemos:

$$b = -x_0 \cdot a + y_0$$

Isso é **uma reta no espaço de parâmetros** $(a, b)$!

* Um **ponto** no espaço da imagem → uma **reta** no espaço de parâmetros.
* Uma **reta** no espaço da imagem → um **ponto** no espaço de parâmetros.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ Pontos colineares → pico no acumulador

Se dois pontos $P_1$ e $P_2$ estão sobre a mesma reta, as duas retas no espaço de parâmetros se **cruzam num único ponto** $(a^*, b^*)$ os parâmetros da reta detectada.

Com vários pontos, cada um gera uma reta no espaço de parâmetros. Pontos colineares concentram cruzamentos na mesma região; pontos de ruído espalhados cruzam em regiões diferentes, sem concentração.

O algoritmo conta os cruzamentos numa grade (**acumulador**) e encontra o pico a reta com mais votos.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ Por que usar votação e não cruzamentos exatos?

Na prática os pontos têm ruído: um pixel que deveria estar em $(2, 3)$ pode estar em $(2.1, 3.2)$.

As retas no espaço de parâmetros não se cruzam num ponto único, mas numa **região densa** de cruzamentos próximos.

O acumulador soma esses votos numa grade discreta e o pico emerge mesmo com os votos ligeiramente espalhados o algoritmo não exige perfeição dos dados.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ Problema da equação clássica

$y = ax + b$ não consegue representar **retas verticais**: para $x = c$, o coeficiente $a$ seria infinito.

**Solução:** usar a forma polar:

$$r = x \cos\theta + y \sin\theta$$

onde $r$ é a distância perpendicular da reta à origem e $\theta$ é o ângulo da normal.

Com essa parametrização qualquer reta inclusive verticais é representada por um par finito $(r, \theta)$, com $r \geq 0$ e $0° \leq \theta < 180°$.

A lógica continua a mesma: cada ponto vira uma curva (agora uma sinusoide) no espaço $(r, \theta)$, e pontos colineares geram sinusoides que se cruzam no mesmo ponto.

++++++++++++++++++++++++++++++++++++++++++

++++++++++++++++++++++++++++++++++++++++++ O algoritmo

Para cada ponto de borda $(x, y)$:

1. Para cada ângulo $\theta_j$ de $0°$ a $179°$:
   * Calcule $r = x \cos\theta_j + y \sin\theta_j$
   * Incremente a célula $(r, \theta_j)$ no acumulador

Ao final, os **picos do acumulador** são as retas detectadas.

**Custo:** $O(N \times M)$, onde $N$ é o número de pontos de borda e $M$ o número de ângulos discretizados muito melhor que $O(N^2)$.

++++++++++++++++++++++++++++++++++++++++++


## A Abordagem Ingênua

Vamos entender por que o problema não é simples.

A ideia mais direta seria: para cada par de pontos, traçar a reta que passa por eles e contar quantos outros pontos estão sobre ela. A reta com mais pontos seria a detectada.

??? Checkpoint
Suponha que você tem $N$ pontos. Quantos pares de pontos existem? Qual seria a complexidade dessa abordagem?
::: Gabarito
O número de pares é $\binom{N}{2} = \frac{N(N-1)}{2}$, o que dá complexidade $O(N^2)$.

Para imagens reais, $N$ pode ser da ordem de dezenas de milhares de pixels de borda testar todos os pares é inviável.
:::
???

Mas há um segundo problema, mais sutil. Mesmo que o custo fosse aceitável, a abordagem quebraria na prática porque pontos de borda reais nunca estão perfeitamente sobre uma reta. Ruído, variações de iluminação e oclusões fazem cada ponto desviar levemente da posição ideal.

??? Checkpoint
Suponha que dois pontos deveriam estar sobre a reta $y = x$, mas por ruído estão em $P_1 = (1, 1.3)$ e $P_2 = (3, 2.8)$. Qual reta passa por esses dois pontos? Ela é a mesma que $y = x$?

**Dica:** use a fórmula do coeficiente angular $a = \frac{y_2 - y_1}{x_2 - x_1}$.
::: Gabarito
$$a = \frac{2.8 - 1.3}{3 - 1} = \frac{1.5}{2} = 0.75$$

$$b = y_1 - a \cdot x_1 = 1.3 - 0.75 \cdot 1 = 0.55$$

A reta é $y = 0.75x + 0.55$ bem diferente de $y = x$. Um terceiro ponto em $(5, 5)$, que está perfeitamente sobre $y = x$, ficaria a $|5 - (0.75 \cdot 5 + 0.55)| = |5 - 4.3| = 0.7$ unidades dessa reta e seria descartado como "não pertencente".

Pequenos desvios nos pontos usados para definir a reta produzem retas completamente diferentes. A abordagem ingênua não tolera ruído.
:::
???

Esses dois problemas custo quadrático e sensibilidade ao ruído motivam a Transformada de Hough.

## A Ideia Central: Mudando de Perspectiva

Os slides apresentaram a observação-chave: um ponto no espaço da imagem vira uma reta no espaço de parâmetros. Vamos verificar isso na prática.

??? Checkpoint
Dado o ponto $P = (2, 3)$, escreva a equação que relaciona $a$ e $b$ para **qualquer** reta $y = ax + b$ que passe por $P$.
::: Gabarito
Substituindo $x = 2$ e $y = 3$:

$$3 = a \cdot 2 + b \implies b = -2a + 3$$

Isso é uma reta no **espaço de parâmetros** $(a, b)$. Cada par $(a, b)$ sobre essa reta representa uma reta da imagem que passa por $P$.
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

Igualando: $-a + 1 = -3a + 3 \implies 2a = 2 \implies a = 1$, portanto $b = 0$.

O ponto de interseção é $(a^*, b^*) = (1, 0)$, que corresponde à reta $y = x$ exatamente a reta que passa pelos dois pontos.

**Conclusão:** dois pontos colineares no espaço da imagem geram duas retas que se cruzam num ponto no espaço de parâmetros. Esse ponto de cruzamento *é* a reta detectada.
:::
???

![Espaço da imagem vs espaço de parâmetros](espaco_parametros.png)

## Generalizando: Vários Pontos com Ruído

Com muitos pontos, cada um gera uma reta no espaço de parâmetros. Se todos fossem perfeitamente colineares, todas as retas se cruzariam num único ponto exato. Mas vimos que pontos reais têm ruído as retas não se cruzam num ponto único, e sim numa região densa de cruzamentos próximos.

Para lidar com isso, o algoritmo discretiza o espaço de parâmetros numa grade e usa **votação**: cada ponto incrementa a célula $(a, b)$ correspondente a cada reta que passa por ele. O resultado é chamado de **acumulador**. Ao final, os picos do acumulador as células com mais votos são as retas detectadas.

??? Checkpoint
Considere os pontos abaixo, sendo 4 aproximadamente colineares (com ruído) e 2 de ruído puro:

- $A = (0, 1.0)$, $B = (2, 3.1)$, $C = (4, 4.9)$, $D = (6, 7.2)$ aproximadamente sobre $y = x + 1$, mas com desvios
- $R_1 = (1, 5)$, $R_2 = (5, 1)$ ruído puro

No acumulador, a região em torno de qual ponto $(a^*, b^*)$ deve acumular mais votos? Os pontos $R_1$ e $R_2$ contribuem para esse mesmo pico?

**Dica:** não tente calcular os cruzamentos exatos pense qualitativamente. Como o ruído nos pontos afeta onde as retas se cruzam no espaço de parâmetros?
::: Gabarito
A reta ideal que contém A–D é $y = x + 1$, com $a^* = 1$ e $b^* = 1$. Como os pontos têm ruído, as retas no espaço de parâmetros não se cruzam num ponto único elas se cruzam numa **região densa** em torno de $(1, 1)$. O acumulador soma todos esses votos próximos, e o pico emerge dessa concentração: 4 votos (um de cada ponto A–D).

As retas de $R_1$ e $R_2$ cruzam em uma região diferente, longe de $(1, 1)$, acumulando apenas 2 votos. A reta real se destaca pelo pico mais alto, mesmo com ruído.
:::
???

![Pontos com ruído no espaço da imagem e o pico correspondente no acumulador](pontos_ruido_acumulador.png)

## O Problema da Equação Clássica

Até aqui usamos $y = ax + b$ para representar retas. Essa parametrização tem um ponto cego: não consegue representar **retas verticais**.

Uma reta vertical tem a forma $x = c$. Para expressá-la como $y = ax + b$, precisaríamos de $a \to \infty$ um valor inválido para indexar o acumulador.

A solução é usar a **forma polar**:

$$r = x \cos\theta + y \sin\theta$$

onde $r$ é a distância perpendicular da reta à origem e $\theta$ é o ângulo da normal à reta com o eixo $x$. Com essa parametrização, qualquer reta incluindo verticais é representada por um par finito $(r, \theta)$, com $r \geq 0$ e $0° \leq \theta < 180°$.

A lógica do algoritmo não muda. A única diferença é a forma das curvas no espaço de parâmetros.

??? Checkpoint
Na representação $(a, b)$, fixar um ponto $(x_0, y_0)$ gerava uma **reta** no espaço de parâmetros. Na forma polar, o mesmo ponto gera a curva $r = x_0 \cos\theta + y_0 \sin\theta$.

Por que essa curva é uma **sinusoide** e não uma reta?

**Dica:** Compare as duas relações $b = -x_0 a + y_0$ e $r = x_0 \cos\theta + y_0 \sin\theta$. O que é matematicamente diferente entre elas?
::: Gabarito
Na equação $b = -x_0 a + y_0$, os parâmetros $a$ e $b$ aparecem de forma **linear** por isso a curva é uma reta no espaço $(a, b)$.

Na equação $r = x_0 \cos\theta + y_0 \sin\theta$, o parâmetro $\theta$ aparece dentro de funções trigonométricas por isso a curva é uma **sinusoide** no espaço $(r, \theta)$.

A ideia continua a mesma: pontos colineares geram sinusoides que se cruzam num ponto. A geometria das curvas mudou, mas o princípio de votação é idêntico.
:::
???

## O Acumulador: Votação na Prática

O acumulador é uma matriz onde cada linha representa um valor discreto de $r$ e cada coluna representa um ângulo $\theta_j$ (por exemplo, de $0°$ a $179°$, em passos de $1°$). Cada célula $(r_i, \theta_j)$ conta quantos pontos de borda votaram nessa reta candidata.

O processo para cada ponto de borda $(x, y)$ é:

- Para cada ângulo $\theta_j$, calcule $r = x \cos\theta_j + y \sin\theta_j$ e incremente a célula correspondente.

Ao final, os picos do acumulador são as retas detectadas.

??? Checkpoint
Por que é necessário usar um acumulador com votação em vez de simplesmente calcular a interseção exata das sinusoides?

Pense no caso em que os pontos têm ruído eles não estão perfeitamente sobre a reta.
::: Gabarito
Se os pontos fossem perfeitamente colineares, as sinusoides se cruzariam num único ponto exato, e não precisaríamos do acumulador.

Na prática, pontos com ruído geram sinusoides levemente deslocadas. Elas não se cruzam num ponto único, mas numa região densa de cruzamentos próximos. O acumulador soma esses votos célula a célula o pico emerge mesmo que os votos estejam espalhados numa pequena vizinhança.

Isso é justamente o que torna o algoritmo robusto: ele não exige perfeição dos dados.
:::
???

![Acumulador polar: o pico brilhante corresponde à reta detectada](acumulador_polar.png)

!!! Aviso
O tamanho do acumulador afeta tanto a precisão quanto o custo. Uma grade fina detecta retas com mais exatidão, mas ocupa mais memória e leva mais tempo para ser percorrida.
!!!

## Eficiência

O algoritmo tem dois laços encadeados: para cada um dos $N$ pontos de borda, percorre todos os $M$ ângulos discretizados e incrementa uma célula do acumulador.

??? Checkpoint
Com base nessa descrição, qual é a complexidade do preenchimento do acumulador?
::: Gabarito
São $N$ pontos, cada um disparando $M$ incrementos um laço duplo de custo $O(N \times M)$.

Para $M = 180$ ângulos fixos, o custo é **linear em $N$** muito melhor que $O(N^2)$ da abordagem ingênua.
:::
???

Após o preenchimento, ainda é necessário varrer o acumulador para encontrar os picos. Se o acumulador tem $R \times M$ células (onde $R$ é o número de valores discretos de $r$), essa busca tem custo $O(R \times M)$.

??? Checkpoint
O custo total do algoritmo é $O(N \times M + R \times M)$. Para imagens com muitas bordas, qual dos dois termos domina? Por quê?
::: Gabarito
$O(N \times M)$ domina quando $N \gg R$, o que é comum em imagens reais com muitos pixels de borda.

Como $M$ e $R$ são constantes definidas pela resolução do acumulador, o custo efetivo é **linear em $N$**.
:::
???


## Resumo

A Transformada de Hough detecta retas em conjuntos de pontos por meio de três ideias encadeadas:

1. **Dualidade ponto–reta:** um ponto no espaço da imagem vira uma curva no espaço de parâmetros, e uma reta no espaço da imagem vira um ponto no espaço de parâmetros.
2. **Votação:** cada ponto de borda vota de forma independente em todos os pares $(r, \theta)$ compatíveis com ele. Pontos colineares concentram votos numa mesma região do acumulador.
3. **Detecção de picos:** os picos do acumulador correspondem às retas presentes na imagem.

O resultado é um algoritmo $O(N \times M)$, eficiente e robusto a ruído e oclusões.


## Desafio: Detecção de Círculos

A mesma lógica se estende para círculos. Um círculo é definido por três parâmetros: centro $(a, b)$ e raio $r$.

Se o raio já é conhecido, cada ponto de borda vota em todos os possíveis centros que estariam à distância $r$ dele, o acumulador vira 2D $(a, b)$ e o custo é $O(N \times W \times H)$, onde $W \times H$ é o tamanho da imagem.

Se o raio é desconhecido, o acumulador vira 3D $(a, b, r)$ e o custo sobe para $O(N \times W \times H \times R)$.

??? Desafio
Suponha que você sabe que o raio dos círculos que quer detectar está entre 10 e 50 pixels, e a imagem tem resolução $100 \times 100$. O número de pontos de borda é $N = 500$.

Estime o número de operações no pior caso para o acumulador 3D. Isso é viável?
::: Gabarito
$R = 50 - 10 = 40$ valores de raio. O custo seria:

$$500 \times 100 \times 100 \times 40 = 200{,}000{,}000$$

Duzentos milhões de operações. Para imagens maiores ou ranges de raio mais amplos, isso cresce rapidamente. Na prática, limitar o range de raios e usar técnicas como o gradiente de borda para votar apenas na direção correta reduzem o custo significativamente.
:::
???
