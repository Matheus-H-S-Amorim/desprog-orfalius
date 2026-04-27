# Transformada de Hough

## O Problema

Imagine que você tem uma foto de uma estrada. Após aplicar um detector de bordas ,como o algoritmo de Canny ,você obtém uma imagem binária onde alguns pixels estão "acesos", marcando os contornos dos objetos.

![Exemplo de imagem de estrada após detector de bordas](image.png)

O resultado desse pré-processamento é simplesmente um **conjunto de pontos** com coordenadas $(x, y)$. O computador não sabe que esses pontos formam faixas, bordas ou qualquer forma geométrica ,ele enxerga apenas uma nuvem de coordenadas.

!!! Aviso
A entrada da Transformada de Hough **não é uma imagem**. É um conjunto de pontos $(x, y)$ ,por exemplo, as coordenadas dos pixels acesos após o pré-processamento com Canny. A transformada começa depois disso.
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

Para imagens reais, $N$ pode ser da ordem de dezenas de milhares de pixels de borda ,isso tornaria a abordagem inviável na prática.
:::
???

Além do custo alto, há outro problema sério: na vida real, os pontos **nunca são perfeitamente colineares**. Ruído, iluminação irregular e oclusões fazem com que os pontos se desviem levemente da reta ideal. Um par de pontos levemente fora do lugar já produziria uma reta completamente diferente.

É para resolver exatamente esses dois problemas ,custo e robustez a ruído ,que existe a Transformada de Hough.

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

No espaço da imagem, isso corresponde à reta $y = 1 \cdot x + 0$, ou seja, $y = x$ ,que é exatamente a reta que passa pelos dois pontos!

**Conclusão:** dois pontos colineares no espaço da imagem correspondem a duas retas que se cruzam num único ponto no espaço de parâmetros. Esse ponto de cruzamento *é* a reta detectada.
:::
???

---

## Generalizando: Vários Pontos

Se tivermos vários pontos colineares, cada um gera uma reta no espaço de parâmetros, e todas essas retas se cruzam no mesmo ponto $(a^*, b^*)$ ,os parâmetros da reta que os contém.

E os pontos de ruído? Eles também geram retas no espaço de parâmetros, mas essas retas se cruzam em pontos espalhados, sem concentração. O sinal verdadeiro se destaca pelo acúmulo de cruzamentos.

??? Checkpoint
Considere os pontos abaixo, sendo 4 aproximadamente colineares e 2 de ruído:

- $A = (0, 1)$, $B = (2, 3)$, $C = (4, 5)$, $D = (6, 7)$ ,sobre a reta $y = x + 1$
- $R_1 = (1, 5)$, $R_2 = (5, 1)$ ,ruído

No espaço de parâmetros, as retas de $A$, $B$, $C$ e $D$ deveriam todas se cruzar perto de que ponto $(a^*, b^*)$?

E as retas de $R_1$ e $R_2$ cruzariam nesse mesmo ponto?
::: Gabarito
A reta que contém $A$, $B$, $C$, $D$ é $y = x + 1$, então $a^* = 1$ e $b^* = 1$. As quatro retas no espaço de parâmetros se cruzam em $(1, 1)$.

As retas de $R_1$ e $R_2$ também se cruzam em algum ponto ,mas em um ponto diferente, longe de $(1, 1)$. No acumulador, o ponto $(1, 1)$ recebe 4 votos, enquanto o cruzamento de $R_1$ e $R_2$ recebe apenas 2.

A reta real é identificada pelo pico com mais votos.
:::
???
