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

Recorrência: $T(n) = 3T(n/2) + O(n)$.
:::
???

---

## Complexidade do Karatsuba

Pelo Teorema Mestre:

- $a = 3$, $b = 2$, $\log_2 3 \approx 1.585$
- $f(n) = O(n)$ cresce mais devagar que $n^{\log_2 3}$

Portanto:

$$T(n) = O(n^{\log_2 3}) \approx O(n^{1.585})$$

**Isso é significativamente melhor que $O(n^2)$!**

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
   - $P_1 = 1 \cdot 3 = 3$
   - $P_2 = 2 \cdot 4 = 8$
   - $P_3 = (1+2) \cdot (3+4) = 3 \cdot 7 = 21$

3. **Termo do meio:**
   - $P_3 - P_1 - P_2 = 21 - 3 - 8 = 10$

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