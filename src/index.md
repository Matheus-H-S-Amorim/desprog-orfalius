# Transformada de Hough

## O Problema

Imagine que você tem uma foto de uma estrada. Após aplicar um detector de bordas, como o algoritmo de Canny, você obtém uma imagem binária onde alguns pixels estão "acesos", marcando os contornos dos objetos.

![Exemplo de imagem de estrada após detector de bordas](image.png)

O resultado desse pré-processamento é simplesmente um **conjunto de pontos** com coordenadas $(x, y)$. O computador não sabe que esses pontos formam faixas, bordas ou qualquer forma geométrica, ele enxerga apenas uma nuvem de coordenadas.

!!! Aviso
A entrada da Transformada de Hough **não é uma imagem**. É um conjunto de pontos $(x, y)$, por exemplo, as coordenadas dos pixels acesos após o pré-processamento com Canny. A transformada começa depois disso.
!!!

O nosso objetivo é: dado esse conjunto de pontos, **encontrar as retas que melhor explicam onde eles estão**.

---

## A Abordagem Ingênua

Antes de ver a solução, vamos entender por que o problema não é trivial.

A ideia mais direta seria: para cada par de pontos, traçar a reta que passa por eles e verificar quantos outros pontos estão sobre essa reta. A reta com mais pontos seria a detectada.

??? Checkpoint
Suponha que você tem $N$ pontos. Quantos pares de pontos existem? Qual seria a complexidade dessa abordagem?
::: Gabarito
O número de pares é $\binom{N}{2} = \frac{N(N-1)}{2}$, o que dá complexidade $O(N^2)$.

Para imagens reais, $N$ pode ser da ordem de dezenas de milhares de pixels de borda, isso tornaria a abordagem inviável na prática.
:::
???

Além do custo alto, há outro problema sério: na vida real, os pontos **nunca são perfeitamente colineares**. Ruído, iluminação irregular e oclusões fazem com que os pontos se desviem levemente da reta ideal. Um par de pontos levemente fora do lugar já produziria uma reta completamente diferente.

É para resolver exatamente esses dois problemas: custo e robustez a ruído, que existe a Transformada de Hough.

---

## A Ideia Central: Mudando de Perspectiva

### Começando pela equação clássica

Você provavelmente conhece a equação de uma reta:

$$y = ax + b$$

onde $a$ é o coeficiente angular (inclinação) e $b$ é o coeficiente linear (intercepto com o eixo $y$).

Agora, considere um único ponto fixo, digamos $P = (2, 3)$. Quantas retas passam por esse ponto?

??? Checkpoint
Dado o ponto $P = (2, 3)$, escreva a equação que relaciona $a$ e $b$ para **qualquer** reta $y = ax + b$ que passe por $P$.
::: Gabarito
Substituindo $x = 2$ e $y = 3$ na equação da reta:

$$3 = a \cdot 2 + b \implies b = -2a + 3$$

Isso é uma reta no **espaço de parâmetros** $(a, b)$! Cada valor de $a$ determina um valor de $b$, e o par $(a, b)$ representa uma reta que passa por $P$.
:::
???

Essa observação é o coração da Transformada de Hough. Vamos formalizar:

- No **espaço da imagem**, um ponto $(x_0, y_0)$ fixo e os parâmetros $(a, b)$ variando definem infinitas retas passando por ele.
- No **espaço de parâmetros** $(a, b)$, esse mesmo ponto $(x_0, y_0)$ vira **uma reta**: $b = -x_0 \cdot a + y_0$.

A transformação inverteu os papéis: o que era um ponto virou uma reta, e o que era uma reta (definida por $a$ e $b$) virou um ponto.

??? Checkpoint
Agora considere **dois** pontos: $P_1 = (1, 1)$ e $P_2 = (3, 3)$.

Escreva a equação da reta no espaço de parâmetros correspondente a cada ponto. Em seguida, encontre o ponto de interseção $(a^*, b^*)$ das duas retas no espaço de parâmetros.

O que esse ponto de interseção representa no espaço da imagem?
::: Gabarito
Para $P_1 = (1, 1)$: $b = -1 \cdot a + 1$

Para $P_2 = (3, 3)$: $b = -3 \cdot a + 3$

Igualando: $-a + 1 = -3a + 3 \implies 2a = 2 \implies a = 1$, portanto $b = 0$.

O ponto de interseção é $(a^*, b^*) = (1, 0)$.

No espaço da imagem, isso corresponde à reta $y = 1 \cdot x + 0$, ou seja, $y = x$, que é exatamente a reta que passa pelos dois pontos!

**Conclusão:** dois pontos colineares no espaço da imagem correspondem a duas retas que se cruzam num único ponto no espaço de parâmetros. Esse ponto de cruzamento *é* a reta detectada.
:::
???

---

## Generalizando: Vários Pontos

Se tivermos vários pontos colineares, cada um gera uma reta no espaço de parâmetros, e todas essas retas se cruzam no mesmo ponto $(a^*, b^*)$, os parâmetros da reta que os contém.

E os pontos de ruído? Eles também geram retas no espaço de parâmetros, mas essas retas se cruzam em pontos espalhados, sem concentração. O sinal verdadeiro se destaca pelo acúmulo de cruzamentos.

??? Checkpoint
Considere os pontos abaixo, sendo 4 aproximadamente colineares e 2 de ruído:

- $A = (0, 1)$, $B = (2, 3)$, $C = (4, 5)$, $D = (6, 7)$, sobre a reta $y = x + 1$
- $R_1 = (1, 5)$, $R_2 = (5, 1)$, ruído

No espaço de parâmetros, as retas de $A$, $B$, $C$ e $D$ deveriam todas se cruzar perto de que ponto $(a^*, b^*)$?

E as retas de $R_1$ e $R_2$ cruzariam nesse mesmo ponto?
::: Gabarito
A reta que contém $A$, $B$, $C$, $D$ é $y = x + 1$, então $a^* = 1$ e $b^* = 1$. As quatro retas no espaço de parâmetros se cruzam em $(1, 1)$.

As retas de $R_1$ e $R_2$ também se cruzam em algum ponto, mas em um ponto diferente, longe de $(1, 1)$. No acumulador, o ponto $(1, 1)$ recebe 4 votos, enquanto o cruzamento de $R_1$ e $R_2$ recebe apenas 2.

A reta real é identificada pelo pico com mais votos.
:::
???

---

## O Problema da Equação Clássica

A equação $y = ax + b$ funciona bem para a maioria das retas, mas tem um ponto cego: **retas verticais**.

Uma reta vertical tem a forma $x = c$ (ou seja uma constante). Não existe valor de $a$ finito que descreva isso na equação $y = ax + b$, o coeficiente angular seria infinito.

Para resolver isso, usamos uma representação diferente, chamada **forma polar**:

$$r = x \cos\theta + y \sin\theta$$

onde:
- $r$ é a distância perpendicular da reta à origem
- $\theta$ é o ângulo que a normal à reta faz com o eixo $x$

Com essa parametrização, qualquer reta, incluindo as verticais, pode ser representada por um par finito $(r, \theta)$, com $r \geq 0$ e $0° \leq \theta < 180°$.

!!! Aviso
A forma polar não é mais difícil de usar na prática, só troca a equação. A lógica do algoritmo permanece exatamente a mesma: cada ponto $(x_0, y_0)$ vira uma curva no espaço $(r, \theta)$, e pontos colineares geram curvas que se cruzam no mesmo ponto.
!!!

??? Checkpoint
Na forma polar, um ponto fixo $(x_0, y_0)$ gera, no espaço $(r, \theta)$, a curva $r = x_0 \cos\theta + y_0 \sin\theta$.

Essa curva é uma sinusoide em $\theta$. Por que ela não é mais uma reta reta (como no espaço $(a, b)$)?
::: Gabarito
Na equação $y = ax + b$, quando fixamos $(x_0, y_0)$, obtemos $b = -x_0 a + y_0$, uma relação **linear** entre $a$ e $b$, por isso é uma reta no espaço de parâmetros.

Na forma polar, a relação $r = x_0 \cos\theta + y_0 \sin\theta$ envolve funções trigonométricas de $\theta$, por isso a curva é uma **sinusoide** e não uma reta. A ideia continua a mesma, pontos colineares geram sinusoides que se cruzam num ponto, mas a geometria das curvas muda.
:::
???

---

## O Acumulador: Votação na Prática

Na prática, não calculamos interseções analíticas entre curvas. Em vez disso, discretizamos o espaço de parâmetros numa grade e usamos **votação**.

O algoritmo cria uma matriz, chamada **acumulador**, onde cada célula $(r_i, \theta_j)$ representa uma reta candidata. O processo é:

1. Para cada ponto de borda $(x, y)$:
   - Para cada ângulo $\theta_j$ discretizado (por exemplo, de $0°$ a $179°$):
     - Calcule $r = x \cos\theta_j + y \sin\theta_j$
     - Incremente o contador da célula correspondente no acumulador
2. Encontre os picos (células com mais votos) no acumulador, cada pico é uma reta detectada.

??? Checkpoint
Por que é necessário usar um acumulador com votação em vez de simplesmente calcular a interseção exata das curvas?

Pense no caso em que os pontos têm ruído, eles não estão perfeitamente sobre a reta.
::: Gabarito
Se os pontos fossem perfeitamente colineares, as curvas se cruzariam num único ponto exato, e não precisaríamos do acumulador.

Na prática, porém, pontos com ruído têm coordenadas levemente erradas. As curvas correspondentes não se cruzam num ponto único, mas numa região densa de cruzamentos próximos. O acumulador acumula esses votos ao longo de uma vizinhança, e o pico emerge dessa densidade.

Isso é justamente o que torna o algoritmo robusto: ele não exige perfeição dos dados. Uma maioria de pontos colineares "aproximados" já é suficiente para produzir um pico claro.
:::
???

!!! Aviso
O tamanho do acumulador afeta tanto a precisão quanto o custo. Uma grade fina detecta retas com mais exatidão, mas ocupa mais memória e leva mais tempo para ser percorrida.
!!!

---

## Eficiência

Vamos analisar o custo do algoritmo.

Seja $N$ o número de pontos de borda e $M$ o número de ângulos discretizados.

Para cada um dos $N$ pontos, o algoritmo percorre todos os $M$ ângulos e incrementa uma célula do acumulador. Isso é um laço duplo simples.

??? Checkpoint
Com base na descrição acima, qual é a complexidade do algoritmo de detecção de linhas?
::: Gabarito
$$O(N \times M)$$

Para uma resolução angular fixa (por exemplo, $M = 180$ ângulos de $1°$ cada), o custo é **linear no número de pontos de borda**, muito melhor que a abordagem ingênua $O(N^2)$.
:::
???

Após o preenchimento do acumulador, ainda é necessário encontrar os picos. Se o acumulador tem $R \times M$ células (onde $R$ é o número de valores discretos de $r$), essa busca tem custo $O(R \times M)$.

O custo total é portanto $O(N \times M + R \times M)$. Como $M$ é fixo e $N$ domina para imagens com muitas bordas, dizemos que o algoritmo é **linear no número de pontos de borda** para resolução angular fixa.

---

## Resumo

A Transformada de Hough detecta retas em conjuntos de pontos por meio de três ideias encadeadas:

1. **Dualidade ponto–reta:** um ponto no espaço da imagem vira uma curva no espaço de parâmetros, e uma reta no espaço da imagem vira um ponto no espaço de parâmetros.
2. **Votação:** cada ponto de borda vota de forma independente em todos os pares $(r, \theta)$ compatíveis com ele. Pontos colineares concentram votos numa mesma região do acumulador.
3. **Detecção de picos:** os picos do acumulador correspondem às retas presentes na imagem.

O resultado é um algoritmo $O(N \times M)$, eficiente e robusto a ruído e oclusões.

---

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


