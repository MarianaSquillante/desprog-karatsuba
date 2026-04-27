# Algoritmo de Karatsuba

## Redefinindo o "Básico" e a Falha da Divisão Simples

Até agora, tratamos qualquer operação aritmética — soma, subtração, multiplicação — como tendo custo $O(1)$. Isso faz sentido para números que cabem em um registrador de 32 ou 64 bits. Mas e quando precisamos multiplicar números com **milhares ou milhões de dígitos**, como em criptografia ou simulações científicas?

Nesses casos, o computador opera **dígito por dígito**. Por isso, precisamos redefinir a operação básica.

!!! Aviso
A partir de agora, a **operação básica** é a manipulação de um único dígito. Isso muda tudo na nossa análise de complexidade.
!!!

Consequências imediatas:

- **Somar** dois números de $n$ dígitos: custo $O(n)$.
- **Multiplicar** pelo método escolar: custo $O(n^2)$.

Nosso objetivo: encontrar um algoritmo melhor do que $O(n^2)$.

---

## A Tentativa Ingênua: Dividir para Conquistar

Dado dois números $x$ e $y$ de $n$ dígitos, separe-os em metade alta e metade baixa:

$$x = A_1 \cdot 10^{n/2} + A_0$$
$$y = B_1 \cdot 10^{n/2} + B_0$$

**Exemplo:** $x = 1234$, $n=4$ → $A_1 = 12$, $A_0 = 34$, pois $1234 = 12 \cdot 10^2 + 34$.

Expandindo o produto:

$$x \cdot y = (A_1 \cdot 10^{n/2} + A_0)(B_1 \cdot 10^{n/2} + B_0)$$
$$= A_1 B_1 \cdot 10^{n} + (A_1 B_0 + A_0 B_1) \cdot 10^{n/2} + A_0 B_0$$

Multiplicar por $10^{n/2}$ ou $10^n$ é um simples deslocamento (*shift*), custo $O(n)$. O custo real está nas multiplicações entre blocos.

??? Checkpoint
Quantas multiplicações entre blocos de $n/2$ dígitos precisamos realizar?
::: Gabarito
São **4 multiplicações**: $A_1 B_1$, $A_1 B_0$, $A_0 B_1$ e $A_0 B_0$.
:::
???

---

A Decepção: Ainda $O(n^2)$
---------------------------

Com 4 multiplicações de subproblemas de tamanho $n/2$ e um custo adicional de $O(n)$ para as somas e os *shifts*, a relação de recorrência desse método é:

$$T(n) = 4T(n/2) + O(n)$$

??? Checkpoint

Com base na recorrência acima e nas complexidades que vimos até agora, o que você consegue concluir sobre a eficiência desse método?

::: Gabarito
Calculamos $\log_2 4 = 2$, portanto $n^{\log_b a} = n^2$.

Como $f(n) = O(n)$ cresce mais devagar que $n^2$, a recorrência resolve para:

$$T(n) = O(n^2)$$

{red}(Nadamos, nadamos... e morremos na praia.) Dividir o número ao meio e gerar 4 subproblemas **não melhorou nada** — a complexidade continua sendo $O(n^2)$. Estamos presos nessas 4 multiplicações.

Será que existe alguma forma matemática de reduzir esse número?
:::

???

## A Sacada Genial de Karatsuba

Não precisamos de $A_1B_0$ e $A_0B_1$ separados. **Só precisamos da SOMA deles:** $(A_1B_0 + A_0B_1)$.

Vamos obter essa soma por outro caminho. Considere:

$$(A_1 + A_0) \cdot (B_1 + B_0)$$

Expandindo:

$$(A_1 + A_0)(B_1 + B_0) = A_1B_1 + A_1B_0 + A_0B_1 + A_0B_0$$

Os termos $A_1B_1$ e $A_0B_0$ já seriam calculados de qualquer forma. Portanto:

$$A_1B_0 + A_0B_1 = (A_1 + A_0)(B_1 + B_0) - A_1B_1 - A_0B_0$$

---

## O Algoritmo Karatsuba

Definimos três produtos recursivos:

- $P_1 = A_1 \cdot B_1$
- $P_2 = A_0 \cdot B_0$
- $P_3 = (A_1 + A_0) \cdot (B_1 + B_0)$

Então o resultado final é:

$$x \cdot y = P_1 \cdot 10^{n} + (P_3 - P_1 - P_2) \cdot 10^{n/2} + P_2$$

??? Checkpoint
Quantas multiplicações de números de tamanho $n/2$ são necessárias agora?
::: Gabarito
**Apenas 3 multiplicações!** $P_1$, $P_2$ e $P_3$.
:::
???

---

## Comparação Visual

| Abordagem | Multiplicações | Complexidade |
|-----------|:-------------:|:------------:|
| Método escolar | 4 | $O(n^2)$ |
| **Karatsuba** | **3** | $\mathbf{O(n^{1.585})}$ |

**A ideia-chave:** Trocamos 1 multiplicação (cara) por 2 somas e 1 subtração (baratas). Para números grandes, o saldo é extremamente positivo.

---

## Exemplo Interativo: Faça Você Mesmo

Pegue $A = 12$ e $B = 34$ (base 10, $n=2$):

1. **Divida:**
   - $A_1 = 1$, $A_0 = 2$
   - $B_1 = 3$, $B_0 = 4$

2. **Calcule os três produtos:**
   - $Z_1 = 1 \cdot 3 = 3$
   - $Z_2 = 2 \cdot 4 = 8$
   - $Z_3 = (1+2) \cdot (3+4) = 3 \cdot 7 = 21$

3. **Termo do meio:**
   - $Z_3 - Z_1 - Z_2 = 21 - 3 - 8 = 10$

4. **Monte o resultado** ($10^{n}=100$, $10^{n/2}=10$):
   - $3 \cdot 100 + 10 \cdot 10 + 8 = 300 + 100 + 8 = 408$

Conferindo: $12 \times 34 = 408$ ✓

---

## Resumo da Mágica

O que Karatsuba fez não foi mágica, foi **álgebra estratégica**. Ele percebeu que, calculando o produto da soma das partes, ganhava de brinde os termos cruzados que precisava. O preço foi apenas umas subtrações extras — muito mais baratas que uma multiplicação completa.

---

## Para Saber Mais

- O algoritmo de Karatsuba foi publicado em 1962 por Anatoly Karatsuba.
- Foi o primeiro algoritmo de multiplicação a superar a barreira $O(n^2)$.
- Hoje, algoritmos ainda mais rápidos existem, como o FFT (Toom-Cook, Schönhage–Strassen), mas Karatsuba é um excelente equilíbrio para números de tamanho moderado (centenas a milhares de dígitos).

## A Recursão

Agora que entendemos como o algoritmo reduz o número de multiplicações para 3, precisamos analisar como isso impacta o tempo de execução.

A resposta está em uma mudança de perspectiva fundamental: **essas três multiplicações não são resolvidas diretamente** como uma conta de escola. Em vez disso, cada uma delas é uma **chamada recursiva** à própria função de Karatsuba.

Essas multiplicações são resolvidas de forma recursiva: o algoritmo chama a si mesmo para resolver subproblemas menores.

!!! Aviso
A base de todo processo recursivo é saber a hora de parar. Quando um número é tão pequeno que tem apenas **1 dígito**, a multiplicação se torna a nossa operação básica e tem custo $O(1)$. Essa é a condição de parada do nosso algoritmo.
!!!

O algoritmo realiza três chamadas recursivas sobre subproblemas de tamanho $n/2$.

Agora que temos clareza sobre **quais** são e **como** essas três multiplicações se conectam, vamos focar no que realmente importa para o nosso objetivo: **provar que essa estratégia é, de fato, mais rápida.**

Antes de montar a árvore, precisamos separar bem duas coisas que acontecem dentro do algoritmo:

1.  **O custo das chamadas recursivas:** As três multiplicações que o algoritmo chama para resolver os subproblemas menores. Esse custo é representado por $T(n/2)$.
2.  **O custo do trabalho extra:** Tudo que o algoritmo faz *além* das chamadas recursivas. Estamos falando das somas ($A_1 + A_0$ e $B_1 + B_0$), das subtrações para recompor o resultado e dos deslocamentos (multiplicar por $10^n$ ou $10^{n/2}$).

Como as somas e os deslocamentos não envolvem nenhum laço aninhado ou operação mais cara, podemos analisar seu custo pensando na quantidade de dígitos.

Se os números têm $n$ dígitos, operações como soma, subtração e deslocamento percorrem todos os dígitos do número. Como lidamos com $n$ dígitos, cada uma dessas operações custa $O(n)$.

Como realizamos apenas um número constante dessas operações em cada nível, o custo total desse trabalho extra cresce proporcionalmente a $n$.

Por isso, representamos esse custo como $O(n)$.

!!! Aviso
Esse termo $O(n)$ **não representa uma nova chamada recursiva**.

Ele corresponde ao trabalho feito **dentro de cada nível** do algoritmo (somas, subtrações e deslocamentos).
!!!

---

## Somando Tudo

Com isso, o custo do algoritmo em um único nível é: $3$ chamadas recursivas (cada uma com metade do tamanho) mais esse trabalho extra de $O(n)$.

??? Checkpoint
Com base no que acabamos de ver sobre a estrutura recursiva, monte a relação de recorrência para o algoritmo de Karatsuba.
Escreva a equação $T(n) = \dots$
::: Gabarito
A relação de recorrência que descreve o tempo de execução do algoritmo é:

$$T(n) = 3T\left(\frac{n}{2}\right) + O(n)$$

Onde $3$ representa o número de chamadas, $\frac{n}{2}$ é o tamanho da entrada que cada chamada resolve, e $O(n)$ é o custo de "remontar" a solução com somas e deslocamentos.
:::
???

---

## Analisando a Complexidade

Agora que entendemos a estrutura recursiva do algoritmo, queremos responder à pergunta central:

**qual é o custo total do Karatsuba?**

Vamos usar a **árvore de recursão**, pois ela nos permite visualizar o custo em cada nível da execução.

Observe a árvore abaixo. Cada nó representa uma chamada recursiva do algoritmo, e cada nível mostra como o problema é dividido em subproblemas menores.

![Árvore de recursão do algoritmo de Karatsuba](karatsuba_arvore.png)

??? Checkpoint
Observe a árvore acima. Em cada chamada, quantos subproblemas são gerados? E qual é o tamanho de cada um?
::: Gabarito
O algoritmo gera 3 subproblemas, cada um com tamanho $n/2$.
???
Agora vamos analisar o custo por nível da árvore.

??? Checkpoint
No nível 0 existe apenas 1 problema de tamanho $n$.
Qual é o custo extra desse nível?
::: Gabarito
$$c\cdot n$$

O custo é proporcional a $n$ porque, como vimos, as operações realizadas (somas, subtrações e deslocamentos) percorrem todos os dígitos dos números.

Multiplicamos por uma constante $c$ porque estamos realizando **um número fixo de operações** desse tipo, independentemente de $n$.

Ou seja, o tempo total é algo como:

$$(\text{número de operações}) \times (\text{custo de cada operação}) = c \cdot n$$
:::
???

??? Checkpoint
No nível 1 existem 3 problemas de tamanho $n/2$.
Qual é o custo total desse nível?
::: Gabarito
$$3\cdot c\cdot \frac{n}{2}=c\cdot n\cdot \frac{3}{2}$$
:::
???

??? Checkpoint
No nível 2 existem $3^2=9$ problemas de tamanho $n/4$.
Qual é o custo total?
::: Gabarito
$$9\cdot c\cdot \frac{n}{4}=c\cdot n\cdot \left(\frac{3}{2}\right)^2$$
:::
???

Consegue perceber o padrão?

??? Checkpoint
Escreva a fórmula do custo total no nível $i$.
::: Gabarito
$$c\cdot n\cdot \left(\frac{3}{2}\right)^i$$
:::
???

---

## Altura da Árvore

Para somar todos os níveis, precisamos saber quantos níveis existem.

??? Checkpoint
Se a cada nível o tamanho do problema é dividido por 2, qual é o tamanho do subproblema no nível $k$?
::: Gabarito
$$\frac{n}{2^k}$$
:::
???

??? Checkpoint
A árvore para quando o tamanho chega em 1.
Resolva:
$$\frac{n}{2^k}=1$$
::: Gabarito
$$k=\log_2 n$$
Essa é a altura da árvore.
:::
???

---

## Somando os Andares

Agora podemos somar o custo de todos os níveis:

??? Checkpoint
Monte a soma dos custos dos níveis da árvore.
::: Gabarito
$$S=c\cdot n\left[1+\frac{3}{2}+\left(\frac{3}{2}\right)^2+\dots+\left(\frac{3}{2}\right)^{\log_2 n}\right]$$
:::
???

??? Checkpoint
A soma obtida é uma progressão geométrica.

$$1+\frac{3}{2}+\left(\frac{3}{2}\right)^2+\dots+\left(\frac{3}{2}\right)^{\log_2 n}$$

Identifique:

*   o primeiro termo $a_1$
*   a razão $q$
*   a quantidade de termos $x$
::: Gabarito
*   $a_1=1$
*   $q=\frac{3}{2}$
*   $x=\log_2 n+1$
:::
???

??? Checkpoint
Agora calcule a soma total dos custos. Para isso, use a fórmula da progressão geométrica.

$$S_n=a_1\cdot \frac{q^x-1}{q-1}$$
::: Gabarito
$$S=c\cdot n\cdot \left(\frac{\left(\frac{3}{2}\right)^{\log_2 n+1}-1}{\frac{3}{2}-1}\right)$$

Simplificando:

$$S=2c\cdot n\left[\left(\frac{3}{2}\right)^{\log_2 n+1}-1\right]$$
:::
???

??? Checkpoint
Reescreva a potência separando o expoente $\log_2 n+1$.
Partimos de:

$$S=2c\cdot n\left[\left(\frac{3}{2}\right)^{\log_2 n+1}-1\right]$$
::: Gabarito
$$\left(\frac{3}{2}\right)^{\log_2 n+1} = \left(\frac{3}{2}\right)\left(\frac{3}{2}\right)^{\log_2 n}$$

Portanto:

$$S=2c\cdot n\left[\frac{3}{2}\left(\frac{3}{2}\right)^{\log_2 n}-1\right]$$
:::
???

??? Checkpoint
Simplifique a potência:

$$\left(\frac{3}{2}\right)^{\log_2 n}$$
::: Gabarito
$$\left(\frac{3}{2}\right)^{\log_2 n} = \frac{3^{\log_2 n}}{2^{\log_2 n}}$$

Como:

$$2^{\log_2 n}=n$$
e
$$3^{\log_2 n}=n^{\log_2 3}$$

temos:

$$\left(\frac{3}{2}\right)^{\log_2 n} = \frac{n^{\log_2 3}}{n}$$
:::
???

Substituindo esse resultado na expressão do custo total $S$, vemos que o termo principal da soma cresce proporcionalmente a $n^{\log_2 3}$.

Como estamos interessados na ordem de crescimento, ignoramos constantes e termos menores.

Portanto, a complexidade do algoritmo de Karatsuba é:

$$T(n)=O(n^{\log_2 3})\approx O(n^{1.58})$$