Algoritmo de Karatsuba
======================

Redefinindo o "Básico" e a Falha da Divisão Simples
----------------------------------------------------

Até agora, em muitas análises de algoritmos, tratamos qualquer operação aritmética — soma, subtração, multiplicação — como tendo custo $O(1)$. Isso faz sentido quando lidamos com números que cabem em um registrador de 32 ou 64 bits do processador. Mas e quando precisamos multiplicar números com **milhares ou milhões de dígitos**, como em criptografia ou simulações científicas?

Nesses casos, o computador não consegue resolver a multiplicação em um único ciclo de clock. Ele precisa operar **dígito por dígito**. Por isso, precisamos redefinir o que chamamos de operação básica.

!!! Aviso
A partir de agora, a **operação básica** é a manipulação de um único dígito. Isso muda tudo na nossa análise de complexidade.
!!!

Com essa nova definição, as consequências imediatas são:

* **Somar** dois números de $n$ dígitos requer percorrer cada dígito uma vez: custo $O(n)$.
* **Multiplicar** dois números de $n$ dígitos pelo método escolar (cada dígito de um pelo cada dígito do outro) requer $n \times n$ operações: custo $O(n^2)$.

Nosso objetivo é encontrar um algoritmo de multiplicação melhor do que $O(n^2)$. A primeira ideia natural de um cientista da computação é aplicar a técnica de **Dividir para Conquistar**.

A Tentativa: Dividir para Conquistar
-------------------------------------

A ideia é simples: se o número é grande demais, quebre-o ao meio e resolva partes menores. Dado dois números $x$ e $y$ de $n$ dígitos, podemos separá-los em uma metade alta e uma metade baixa:

$$x = A_1 \cdot 10^{n/2} + A_0$$

$$y = B_1 \cdot 10^{n/2} + B_0$$

onde $A_1$ e $B_1$ são as metades altas (dígitos mais significativos) e $A_0$ e $B_0$ são as metades baixas (dígitos menos significativos), cada uma com $n/2$ dígitos.

Por exemplo, para $x = 1234$ e $n = 4$: $A_1 = 12$ e $A_0 = 34$, de forma que $1234 = 12 \cdot 10^{2} + 34$.

Agora, basta multiplicar $x$ por $y$ aplicando a propriedade distributiva:

$$x \cdot y = (A_1 \cdot 10^{n/2} + A_0) \cdot (B_1 \cdot 10^{n/2} + B_0)$$

Expandindo:

$$x \cdot y = A_1 B_1 \cdot 10^{n} + (A_1 B_0 + A_0 B_1) \cdot 10^{n/2} + A_0 B_0$$

Note que multiplicar por $10^{n/2}$ ou $10^n$ é equivalente a deslocar os dígitos (um simples *shift*), o que custa apenas $O(n)$. O custo real está nas multiplicações entre os blocos.

??? Checkpoint

Olhe com atenção para a equação expandida acima:

$$x \cdot y = A_1 B_1 \cdot 10^{n} + (A_1 B_0 + A_0 B_1) \cdot 10^{n/2} + A_0 B_0$$

Quantas multiplicações entre blocos de $n/2$ dígitos precisamos realizar para calcular o resultado completo? Liste cada uma delas.

::: Gabarito
São **4 multiplicações**: $A_1 B_1$, $A_1 B_0$, $A_0 B_1$ e $A_0 B_0$.

Cada uma delas é uma multiplicação entre números de tamanho $n/2$, ou seja, subproblemas menores que o original — exatamente o que a técnica de Dividir para Conquistar promete.
:::

???

A Decepção: Ainda $O(n^2)$
---------------------------

Com 4 multiplicações de subproblemas de tamanho $n/2$ e um custo adicional de $O(n)$ para as somas e os *shifts*, a relação de recorrência desse método é:

$$T(n) = 4T(n/2) + O(n)$$

??? Checkpoint

Usando o **Teorema Mestre**, qual é a complexidade assintótica da recorrência $T(n) = 4T(n/2) + O(n)$?

Lembre-se: no Teorema Mestre, comparamos $n^{\log_b a}$ com $f(n)$. Aqui, $a = 4$, $b = 2$ e $f(n) = O(n)$.

::: Gabarito
Calculamos $\log_2 4 = 2$, portanto $n^{\log_b a} = n^2$.

Como $f(n) = O(n)$ cresce mais devagar que $n^2$, estamos no **Caso 1** do Teorema Mestre:

$$T(n) = O(n^2)$$

{red}(Nadamos, nadamos... e morremos na praia.) Dividir o número ao meio e gerar 4 subproblemas **não melhorou nada** em relação ao método escolar — a complexidade continua sendo $O(n^2)$, idêntica à multiplicação ingênua. Estamos presos nessas 4 multiplicações.

Será que existe alguma forma matemática de reduzir esse número?
:::

???