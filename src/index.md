# Transformada de Hough

## Motivação: o que você vê nessa imagem?

![Imagem real de uma estrada](vertical-shot-road-with-magnificent-mountains-blue-sky-captured-california.jpg)

??? Checkpoint
Observe a imagem acima com cuidado antes de continuar.

Sem pensar em algoritmos: quais elementos da cena podem ser aproximados por **retas**?

Tente identificar pelo menos três regiões lineares distintas. Tente esboçar mentalmente (ou no papel) onde essas retas estariam.

- Há retas paralelas? Convergentes?
- Alguma está parcialmente ocluída?
- Como você distinguiu a reta de outros elementos da cena?
???

Carros autônomos precisam fazer exatamente o que você acabou de fazer — só que automaticamente, em milissegundos, mesmo com sombras, chuva e marcações desgastadas. Esse é o problema que a **Transformada de Hough** resolve.

## Da imagem ao conjunto de pontos

Antes da Transformada de Hough, um detector de bordas — como o algoritmo de Canny — percorre a imagem e encontra regiões de variação brusca de intensidade. O resultado é um **conjunto de pontos** $(x, y)$ que marcam os contornos da cena.

![Da imagem original aos pontos de borda](road_to_edges.png)

A Transformada de Hough **não recebe a imagem original**. Ela recebe apenas esses pontos de borda.

??? Checkpoint
Observe as três etapas mostradas na figura: imagem original → detector de bordas → conjunto de pontos.

**a)** O que foi preservado nessa sequência? O que foi perdido?

**b)** Olhando só os pontos acesos, você ainda consegue perceber as retas da estrada? Por quê?

**c)** Se um trecho da faixa estiver apagado, o que acontece com o conjunto de pontos naquela região?
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

Mas há um segundo problema, mais sutil. Mesmo que o custo fosse aceitável, a abordagem quebraria na prática porque pontos de borda reais nunca estão perfeitamente sobre uma reta.

![Pontos ideais e pontos com ruído](ruido_reta_duplo_painel_ruido_irregular.png)

??? Checkpoint
Observe a figura acima. O painel da esquerda mostra pontos sem ruído; o da direita, com ruído.

**Antes de ler qualquer explicação:** escolha dois pontos quaisquer no painel da direita e trace mentalmente a reta que passa por eles. Agora escolha outros dois pontos do mesmo conjunto.

- As duas retas que você traçou são parecidas ou bem diferentes?
- Por que dois pontos com um pequeno desvio podem gerar uma reta bem diferente da reta ideal?
- O que isso implica para uma abordagem baseada em pares de pontos?
???

Esses dois problemas — custo quadrático e sensibilidade ao ruído — motivam a Transformada de Hough.

## A ideia central: mudando de perspectiva

Toda reta pode ser escrita como $y = ax + b$, com parâmetros $a$ (inclinação) e $b$ (intercepto).

### Atividade: retas que passam por um ponto

Antes de qualquer fórmula, faça o seguinte exercício.

??? Checkpoint
Considere o ponto $P = (2, 3)$ no plano $xy$.

**No papel (ou mentalmente), esboce pelo menos quatro retas diferentes que passam por $P$.**

Para cada reta que você esboçar, estime os valores de $a$ e $b$ e anote-os como um par $(a, b)$. Por exemplo: uma reta horizontal tem $a = 0$; uma reta com inclinação 1 tem $a = 1$.

Você obteve quatro pares $(a, b)$ diferentes. O que eles têm em comum?
???

??? Checkpoint
Agora, formalize a observação anterior.

Se a reta $y = ax + b$ passa por $P = (2, 3)$, o que você pode dizer sobre $a$ e $b$?

Escreva uma equação que relacione $a$ e $b$.
::: Gabarito
Substituindo $x = 2$ e $y = 3$:

$$3 = a \cdot 2 + b \implies b = -2a + 3$$

Essa é uma **reta no espaço de parâmetros** $(a, b)$. Cada par $(a, b)$ sobre essa reta representa uma reta da imagem que passa por $P = (2, 3)$.

Compare com os quatro pares que você anotou no checkpoint anterior: todos devem satisfazer essa relação.
:::
???

Esse é o insight fundamental: **um ponto no espaço da imagem vira uma reta no espaço de parâmetros**. Isso se chama *dualidade ponto–reta*.

### O que acontece com dois pontos?

![Interseção no espaço de parâmetros](espaco_parametros.png)

??? Checkpoint
Observe a figura.

Sem fazer contas:

- Onde as duas retas do espaço de parâmetros parecem se cruzar?
- O que você acha que esse cruzamento representa?

???

??? Checkpoint
Considere dois pontos: $P_1 = (1, 1)$ e $P_2 = (3, 3)$.

**Antes de calcular:** esboce no papel os dois pontos e trace a reta que passa por ambos. Qual é a inclinação visual dessa reta? Qual parece ser o intercepto?

Agora escreva a equação da reta no espaço de parâmetros correspondente a cada ponto:

- Para $P_1$: qual é a relação entre $a$ e $b$?
- Para $P_2$: qual é a relação entre $a$ e $b$?

**Esboce as duas retas no espaço $(a, b)$.** Elas se cruzam? Onde?

O que esse ponto de interseção representa no espaço da imagem?
::: Gabarito
Para $P_1 = (1, 1)$: $b = -a + 1$

Para $P_2 = (3, 3)$: $b = -3a + 3$

Igualando:

$$-a + 1 = -3a + 3 \implies 2a = 2 \implies a = 1$$

Logo, $b = 0$.

O ponto de interseção é $(a^*, b^*) = (1, 0)$, que corresponde à reta $y = x$.

Compare com o esboço que você fez antes de calcular: $P_1$ e $P_2$ estão sobre $y = x$, com inclinação 1 e intercepto 0.

**Conclusão:** dois pontos colineares no espaço da imagem geram duas retas que se cruzam num único ponto no espaço de parâmetros. Esse ponto *é* a reta detectada.
:::
???

## Generalizando: vários pontos com ruído

Com muitos pontos, cada um gera uma reta no espaço de parâmetros. Se todos fossem perfeitamente colineares, todas as retas se cruzariam num único ponto exato. Mas, na prática, pontos reais têm ruído: as retas não se cruzam em um ponto único, e sim em uma região densa de cruzamentos próximos.

![Pontos com ruído e acumulador](pontos_ruido_acumulador.png)

??? Checkpoint
Observe a figura acima. O painel da esquerda mostra os pontos de borda (com ruído). O painel da direita mostra o acumulador resultante.

**Antes de ler a explicação abaixo:**

- Onde está a região mais brilhante do acumulador? O que ela representa?
- Por que existe um pico bem definido mesmo que os pontos não sejam perfeitamente colineares?
- O que você esperaria ver no acumulador se houvesse duas retas distintas na imagem?
???

Para lidar com o ruído, o algoritmo discretiza o espaço de parâmetros numa grade e usa **votação**. Cada ponto incrementa todas as células $(a, b)$ correspondentes às retas que passam por ele. O resultado é chamado de **acumulador**. Ao final, os picos do acumulador são as retas detectadas.

??? Checkpoint
Considere os pontos abaixo, sendo 4 aproximadamente colineares e 2 de ruído puro:

- $A = (0, 1.0)$, $B = (2, 3.1)$, $C = (4, 4.9)$, $D = (6, 7.2)$
- $R_1 = (1, 5)$, $R_2 = (5, 1)$

No acumulador, a região em torno de qual ponto $(a^*, b^*)$ deve acumular mais votos? Por quê os pontos $R_1$ e $R_2$ não destroem o resultado?
::: Gabarito
A reta ideal que contém $A$–$D$ é $y = x + 1$, com $a^* = 1$ e $b^* = 1$.

Como os pontos têm ruído, as retas no espaço de parâmetros não se cruzam num ponto único; elas se cruzam numa região densa em torno de $(1, 1)$. O acumulador soma esses votos próximos, e o pico emerge dessa concentração.

$R_1$ e $R_2$ também geram retas no espaço de parâmetros, mas essas retas se cruzam com as demais em posições espalhadas, sem formar concentração. O efeito deles no acumulador é difuso, não competitivo com o pico principal.
:::
???

## O problema da equação clássica

Até aqui usamos $y = ax + b$ para representar retas. Essa parametrização tem um ponto cego: não consegue representar **retas verticais**.

Uma reta vertical tem a forma $x = c$. Para expressá-la como $y = ax + b$, precisaríamos de $a \to \infty$, um valor inválido para indexar o acumulador.

A solução é usar a **forma polar**:

$$r = x \cos\theta + y \sin\theta$$

onde $r$ é a distância perpendicular da reta à origem e $\theta$ é o ângulo da normal à reta com o eixo $x$.

![Forma normal de uma reta: r é a distância perpendicular à origem e θ o ângulo da normal com o eixo x](reta_forma_normal.png)

![Curvas no espaço polar](polar_transform.png)

??? Checkpoint
Observe a figura das curvas no espaço $(r, \theta)$ acima antes de continuar.

- Quantas curvas você vê no espaço $(r, \theta)$?
- As curvas parecem retas ou têm outra forma? O que isso sugere sobre a relação entre $r$ e $\theta$?
- Onde as curvas parecem se cruzar? O que você acha que esse cruzamento representa?
???

??? Checkpoint
Na representação $(a, b)$, fixar um ponto $(x_0, y_0)$ gerava uma **reta** no espaço de parâmetros. Na forma polar, o mesmo ponto gera a curva

$$r = x_0 \cos\theta + y_0 \sin\theta$$

Por que essa curva é uma **sinusoide** e não uma reta?
::: Gabarito
Na equação $b = -x_0 a + y_0$, os parâmetros aparecem de forma linear, por isso a curva é uma reta no espaço $(a, b)$.

Na equação $r = x_0 \cos\theta + y_0 \sin\theta$, o parâmetro $\theta$ aparece dentro de funções trigonométricas, por isso a curva é uma sinusoide no espaço $(r, \theta)$.

A ideia continua a mesma: pontos colineares geram curvas que se cruzam num ponto. A geometria das curvas muda, mas o princípio de votação é idêntico.
:::
???

## O acumulador: votação na prática

O acumulador é uma matriz onde cada linha representa um valor discreto de $r$ e cada coluna representa um ângulo $\theta_j$.

O processo para cada ponto de borda $(x, y)$ é:

- para cada ângulo $\theta_j$, calcule $r = x \cos\theta_j + y \sin\theta_j$;
- incremente a célula correspondente no acumulador.

![Pico no acumulador polar](acumulador_polar.png)

??? Checkpoint
Observe o acumulador polar acima.

- Localize visualmente a região de maior intensidade (o pico). Quais são os valores aproximados de $r$ e $\theta$ nessa região?
- O que esses valores significam para a reta na imagem original?
- Por que o pico não é um único pixel brilhante, mas uma pequena região?

Só depois de responder: por que é necessário usar votação em vez de simplesmente calcular a interseção exata das curvas?
::: Gabarito
Se os pontos fossem perfeitamente colineares, as curvas se cruzariam num único ponto exato.

Na prática, pontos com ruído geram curvas levemente deslocadas. Elas não se cruzam num ponto único, mas numa região densa de cruzamentos próximos. O acumulador soma esses votos célula a célula, e o pico emerge mesmo com os votos espalhados numa pequena vizinhança.

Isso torna o algoritmo robusto: ele não exige perfeição dos dados.
:::
???

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

A mesma lógica se estende para círculos. Um círculo é definido por três parâmetros: centro $(a, b)$ e raio $r$. Atenção: aqui $r$ passa a denotar o **raio** do círculo, e não mais a distância perpendicular da forma polar.

![Ideia da extensão para círculos](circles_extension.png)

??? Checkpoint
Observe a figura acima.

Um círculo de raio $r$ centrado em $(a, b)$ satisfaz $(x - a)^2 + (y - b)^2 = r^2$.

Se você fixar um ponto de borda $(x_0, y_0)$ e um raio $r$ conhecido, quais são os possíveis centros $(a, b)$ compatíveis com esse ponto?

Esboce no papel o conjunto de possíveis centros para um único ponto de borda. Que figura geométrica eles formam?
::: Gabarito
Os possíveis centros formam um **círculo** de raio $r$ centrado em $(x_0, y_0)$ no espaço $(a, b)$.

Isso é análogo ao caso das retas: um ponto de borda no espaço da imagem gera uma curva no espaço de parâmetros. Agora a curva é um círculo, não uma sinusoide.

Vários pontos de borda colineares geravam sinusoides que se cruzavam num ponto. Vários pontos de borda sobre um mesmo círculo geram círculos (no espaço de parâmetros) que se cruzam num ponto — o centro do círculo detectado.
:::
???

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