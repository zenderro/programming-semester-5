# Добрый день, группа 312!
<style type="text/css">
div.sourceCode {
  font-size: 1.2em;
}
section.slide > pre {
  font-size: 0.8em;
}
section.slide > p {
  text-align: left;
  text-indent: 2em;
}
.reveal div.owl {
  background-size: 65%;
}
.owl p, .owl h1 {

  -webkit-text-stroke-width: 1.1pt;
-webkit-text-stroke-color: #fff;
  font-weight: 700;
}

.green-box {
  background-color: #afa;
}
.red-box {
  background-color: #faa;
}
.yellow-box {
  background-color: #ff0;
}

</style>

## План
- приведение к верхнему треугольному виду методом Гаусса
- LU-разложение
- метод Жордана
- выбор главного элемента

Материалы к занятиям: https://zenderro.github.io/programming-semester-5/

email: [andrey.zenzinov@math.msu.ru](mailto:andrey.zenzinov@math.msu.ru)

# Ссылки на первое задание

Решение СЛУ: https://classroom.github.com/a/Db2n69ex

Обращение матрицы: https://classroom.github.com/a/xXHlW2-k

# Приведение матрицы к треугольному виду (метод Гаусса)

$$
\begin{array}{cllll|l}
1 & c_{1,2} & \dots & c_{1,n-1} & c_{1,n} & y_1 \\
  &  a^{(1)}_{2,2}  & \dots         & a^{(1)}_{2,n-1} & a^{(1)}_{2,n} & b^{(1)}_2 \\
  & \vdots & \vdots & \vdots & \vdots & \vdots \\
  & a^{(1)}_{n-1, 2} & \dots & a^{(1)}_{n-1, n-1} & a^{(1)}_{n-1, n} & b^{(1)}_{n-1} \\
  & a^{(1)}_{n, 2} & \dots & a^{(1)}_{n, n-1} & a^{(1)}_{n,n} &b^{(1)}_n
\end{array}
$$
В каждом столбце обнуляем часть под диагональю с использованием элементарного преобразования «вычесть строку с коэффициентом» (метод применим только в случае $\det \, A_k ≠ 0$). Сложность — $2/3 n^3 + O(n^2) $
$$ \mathbf{a}_k = \dfrac{\mathbf{a}_k}{a_{k,k}}, \qquad y_k = \dfrac{b_k}{a_{k,k}}; \qquad \mathbf{a}_i = \mathbf{a}_i - \dfrac{a_{i,k}}{a_{k,k}} \mathbf{a}_k, \qquad b_i = b_i - \dfrac{a_{i,k}}{a_{k,k}} y_k $$

# Реализация метода Гаусса
$$
\begin{array}{ccllll|l}
& \bbox[#afa,3pt]{i} &  \bbox[#afa,3pt]{⟶} \\
& 1 & c_{1,2} & \dots & c_{1,n-1} & c_{1,n} & y_1 \\
\bbox[#ff0,3pt] j &   &  a^{(1)}_{2,2}  & \dots         & a^{(1)}_{2,n-1} & a^{(1)}_{2,n} & b^{(1)}_2 \\
\bbox[#ff0,3pt] ↓&  & \vdots & \vdots & \vdots & \vdots & \vdots \\
&  & a^{(1)}_{n-1, 2} & \dots & a^{(1)}_{n-1, n-1} & a^{(1)}_{n-1, n} & b^{(1)}_{n-1} \\
&  & a^{(1)}_{n, 2} & \dots & a^{(1)}_{n, n-1} & a^{(1)}_{n,n} &b^{(1)}_n \\
& & \bbox[#faa,3pt]k & \bbox[#faa,3pt]⟶
\end{array}
$$

<pre>
for <span class=green-box>i := 0…(n-1)</span>
  for k := (i+1)…(n-1) A[i, k] := A[i, k] / A[i, i]
  A[i, i] := 1
  for <span class=yellow-box>j := (i+1)…(n-1)</span>
    for <span class=red-box>k := (i+1)…(n-1)</span>
      A[j, k] := A[j, k] - A[i, k] * A[j, i]
    for <span class=red-box>k := 0…(m-1)</span> # размер правой части (1 или n)
      B[j, k] := B[j, k] - B[i, k] * A[j, i]
</pre>


# LU-разложение

На самом деле, предыдущие преобразования могут быть представлены в форме умножения на матрицы особого вида.

1. Умножение текущей строки на элемент, обратный к диагонали.

<pre>
  for k := (i+1)…(n-1) A[i, k] := A[i, k] / A[i, i]
  A[i, i] := 1
</pre>

$A'^{(i)} = D_i · A^{(i-1)} $ , где $D_i = \mathrm{diag} [ 1, 1, 1, …, {(a^{(i-1)}_{i,i})^{-1}}, 1, …, 1] $ (единичная с обратным к элементу $a_{i,i} $ на $i $-м месте)

# LU-разложение (II)

2. Обнуление столбца под текущим диагональным элементом.

<pre>
    for k := (i+1)…(n-1) A[j, k] := A[j, k] - A[i, k] * A[j, i]
</pre>

$A^{(i)} = L_i · A'^{(i)} $, где $L_i $: (единичная со столбцом обратных по сложению элементов под $i $-м диагональным элементом)

$$
\begin{pmatrix}
1  \\
 & \ddots & \\
 & & 1 \\
 & & -a^{(i-1)}_{i+1, i} & 1 \\
 & & \vdots & & \ddots \\
 & & -a^{(i-1)}_{n, i} & & & 1
\end{pmatrix}
$$

# LU-разложение (III)

Таким образом, применение прямого хода метода Гаусса можно представить как:
$$ L_n · D_n · … · L_1 · D_1 · A = U $$

где $U $ — верхнетреугольная матрица с единицами на диагонали.
Тогда $A = LU $, где $L = (L_n · D_n · … · L_1 · D_1)^{-1} $ — нижнетреугольная матрица (т.к. нижнетреугольные матрицы замкнуты по умножению).
С использованием такого разложения можно далее решать линейные системы с правой частью $A $ и произвольной левой частью за $O(n^2) $ операций:
$$L · U ·x = b$$
1. Обратным ходом метода Гаусса решаем систему с треугольной матрицей
$$ L · y = b $$
2. Обратным ходом метода Гаусса решаем вторую систему с треугольной матрицей:
$$ U · x = y $$

# Реализация LU-разложения
Формулы построены таким образом, что обе матрицы $L, U $ можно (и нужно для выполнения требований) хранить на месте исходной матрицы $A $ (единичная диагональ $U $ не хранится).

1. $l_{i,1} = a_{i,1} \qquad (i = 1…n) \qquad \qquad u_{1,k} = a_{1,k} \big/ l_{1,1} \qquad (k = 2…n) $
2. $\bbox[#faa,3px]{l_{i, k}} = \bbox[#faa,3px]{a_{i,k}} - \sum\limits_{j=1}^{k-1} \bbox[#aff,3px]{l_{i,j}} · \bbox[#faf,3px]{ u_{j,k}}
\qquad (1 < k ⩽ i, \quad i,k = 2…n) $
$\bbox[#afa,3px]{u_{i,k}} = \left(\bbox[#afa,3px]{a_{i, k}} - \sum\limits_{j=1}^{i-1} \bbox[#aff,3px]{l_{i,j}} · \bbox[#ffa,3px]{u_{j,k}} \right) \big/ \, \bbox[#faa,3px]{l_{i,i}}
\qquad (k > i > 1, \quad i, k = 2 … n) $

$$
\begin{pmatrix}
{l_{1,1}} & u_{1,2} & \bbox[#faf,3px]{u_{1, 3}} & \bbox[#ffa,3px]{u_{1, 4}} & … & & & u_{1,n} \\
{l_{2, 1}} & {l_{2, 2}} & \bbox[#faf,3px]{u_{2, 3}} & \bbox[#ffa,3px]{u_{2, 4}} & … & & & u_{2, n} \\
\bbox[#aff,3px]{l_{3, 1}} & \bbox[#aff,3px]{l_{3, 2}} & \bbox[#faa,3px]{l} & \bbox[#afa,3px]{u} & \bbox[#afa,3px]⟶ & & & a_{2, n} \\
\bbox[#faa,3px]↪︎ & \vdots & \vdots & & \ddots & & & \vdots \\
l_{n, 1} & l_{n, 2} & a_{n,3}  & & … & & & a_{n, n}
\end{pmatrix}
$$

# Обратный ход метода Гаусса
$$
\begin{array}{ccccc|c}
c_{1,1} & c_{1,2} & \dots & c_{1,n-1} & {c_{1,n}} & y_1 \\
  &  \bbox[#ffa,3px]{c_{2,2}}  & \bbox[#faf,3px]\dots         & \bbox[#faf,3px]{c_{2,n-1}} & \bbox[#faf,3px]{c_{2,n}} & \bbox[#fea,3px]{y_2} \\
  & &\ddots & \vdots & {\vdots} & \bbox[#aef,3px]{\vdots} \\
  & & & c_{n-1,n-1} & {c_{n-1, n}} & \bbox[#aef,3px]{y_{n-1}} \\
  & & & & c_{n,n} & \bbox[#aef,3px]{y_n}
\end{array}
$$

$ x_n = y_n \big/ c_{n,n} \qquad \qquad \bbox[#fea,3px]{x_i} = \left( \bbox[#fea,3px]{y_i }- \sum_{j=i+1}^n \bbox[#faf,3px]{c_{i,j}} ·  \bbox[#aef,3px]{x_j} \right) \big/ \bbox[#ffa,3px]{c_{i,i}} $

Сложность — $\dfrac{n·(n-1)}{2} ≈ O(n^2)$

# Реализация обратного хода


Во **внутреннем** цикле придерживаемся упорядоченного доступа к памяти.

1. Для решения линейной системы (правая часть — одна):

<pre>
for i:=(n-1) … 0
  for j:= (i+1) … n
    b[i] := b[i] - A[i, j] · b[j]
  b[i] := b[i] / A[i,i]
</pre>

2. Для обращения матрицы (правых частей — $n$, сложность — $O(n^3) $):

<pre>
for i := (n-1) … 0
  for j := (i+1) … n
    for k := 0 … (n-1)
      B[i, k] := B[i, k] - A[i, j] * B[j, k]
  for k := 0 … (n-1)
    B[i, k] := B[i, k] / A[i, i]
</pre>

# Метод Гаусса-Жордана
$$
\begin{array}{cllll|l}
1 & \bbox[#afa,5px]{0} & \dots & c_{1,n-1} & c_{1,n} & y_1 \\
  &  1  & \dots         & a^{(2)}_{2,n-1} & a^{(2)}_{2,n} & b^{(2)}_2 \\
  &  & \vdots & \vdots & \vdots & \vdots \\
  &  & \dots & a^{(2)}_{n-1, n-1} & a^{(2)}_{n-1, n} & b^{(2)}_{n-1} \\
  &  & \dots & a^{(2)}_{n, n-1} & a^{(2)}_{n,n} &b^{(2)}_n
\end{array}
$$

Единственное отличие — обнуляются элементы столбца не только под, но и над диагональю.

<pre>
for <span class=green-box>i := 0…(n-1)</span>
  MultiplyRow(A[i, i…(n-1)], 1 / A[i, i])
  for j :=  <span class=yellow-box><strong>0…(i-1),</strong></span> (i+1)…(n-1)</span>
    A[j, i…(n-1)] := SubtractRow(A[j, i…(n-1)], A[i, i…(n-1)], A[j, i])
    B[j, 0…(n-1)] := SubtractRow(B[j, 0…(n-1)], B[i, 0…(n-1)], A[j, i])
</pre>

# Выбор главного элемента — мотивация
В чистом виде метод Гаусса и разновидности (LU-разложение, метод Жордана) неприменим к матрицам, у которых есть нулевой главный минор

Например, метод не сработает если $a_{1,1} = 0$

Для исправления ситуации можно использовать элементарные преобразования вида «перестановка строк» или «перестановка столбцов» — переставим на нужное место строку (столбец), где нужный элемент отличен от нуля.

Как выбирать какую именно строку или столбец переставлять?

По итогам первого шага (деление строки на диагональный элемент) погрешность изменяется обратно пропорционально модулю диагонального элемента:
$$  ε^{(1)}_{i,j} = ε_{i,j} - ε_{1,j} · \dfrac{a_{i,1}}{a_{1,1}} $$

# Выбор главного элемента
⇒ выгоднее выбирать наибольший по модулю элемент

Переставляем строки — «с выбором главного элемента по столбцу»
Переставляем столбцы — «с выбором главного элемента по строке»
И строки и столбцы — «с выбором главного элемента по всей матрице»

Асимптотика — по столбцу или по строке сохраняется ($2 \big/ 3 n^3 + O(n^2)$), по всей матрице — $n^3 + O(n^2)$). Перестановка элементов в памяти требует лишние $O(n^2)$ операций.

Возможен вариант (используется в реализациях с распределенной памятью), при котором перестановка не выполняется, а используется массив индексов перестановок.

Все методы с выбором главного элемента применимы к любым невырожденным матрицам.

# Выбор главного элемента — реализация

Преобразования правой части:
- перестановка строк — переставляются строки (просто меняем местами уравнения)
- перестановка столбцов —  перенумеруются переменные (правая часть не меняется, но требуется постобработка)

Если мы делаем перестановку столбцов, нужно запоминать все транспозиции ($τ_i$ — транспозиция $i ↔ $ <номер столбца, в котором находится главный элемент $i$-го шага>):
$$ σ = τ_1 · τ_2 · … · τ_n $$

Для получения переменных в правильной последовательности нужно делать обратные транспозиции, начиная с последней ($τ_n^{-1} · τ_{n-1}^{-1}· …$) — фактически, перемешать строки правой части согласно обратной перестановке столбцов левой.

# Выбор главного элемента
## Реализация с индексами перестановок
<pre>
Rows := [0, 1, 2, …, (n-1)]
Cols := [0, 1, 2, …, (n-1)]
Get(A, i, j) := A[Rows[i], Cols[j]]
for i := 0…(n-1)
  max_col, max_row :=i, max_abs := |Get(A, i, i)|
  for j := i…(n-1), k:= i…(n-1)
    if |Get(A, j, k)| > max_abs
      max_abs, max_col, max_row := |Get(A, j, k)|, j, k
  swap(Rows, i, j)
  swap(Cols, j, k)
  MultiplyRow(Get(A, i, i…(n-1)), <strong>1 / Get(A, i, i)</strong>)
  for j := <strong>0…(i-1),</strong> (i+1)…(n-1)
    B[j, 0…(n-1)] := SubtractRow(Get(B, j, 0…(n-1)), Get(B, i, 0…(n-1)), Get(A, j, i))
    A[j, i…(n-1)] := SubtractRow(Get(A, j, i…(n-1)), Get(A, i, i…(n-1)), <strong>Get(A, j, i)</strong>)

for i:=0…(n-1)
  SwapRows(B, i, Cols[i])
</pre>

# Отладчик

Сценарии использования отладчика:

1. Программа падает:
    - при выполнении какой строки?
    - из какой функции была вызвана текущая?
    - какие значения переменных?

2. Программа работает не так, как надо:
    - чему равны значения переменных?
    - в какой момент вызывается тот или иной фрагмент кода (срабатывает условие, выход из цикла и т.п.)?
    - в каком месте программа уходит в бесконечный цикл?

# Вызов отладчика GDB

Для отладки программу нужно скомпилировать с отладочной информацией (ключ `-g`) без оптимизации (`g++` — аналогично):
```bash
gcc -g myprog.c main.c
```
Запуск отладчика:

  - программа без аргументов
```bash
gdb ./a.out
# Вместо ./a.out
```

# Вызов отладчика GDB
  - аргументы при запуске
```bash
gdb --args ./a.out -f 2 -n 1000
# Вместо ./a.out -f 2 -n 1000
```
После запуска отображается командная строка gdb.

Аргументы можно изменить в командной строке gdb:
```bash
set args -f 1 -n 500
show args    # Показать текущие аргументы
```
# Команды отладчика GDB
- `run` (`r`) — запустить программу; в случае исключения выполнение останавливается на строке, на которой исключение произошло:
```
(gdb) r
Program received signal SIGFPE, Arithmetic exception.
0x08048681 in divint(int, int) (a=3, b=0) at crash.c:21
21        return a / b;
```

- `quit` (`q`) — выйти из отладчика (если выполнение программы было прервано, отладчик запросит подтверждение)
- `list` (`l`) — напечатать «окрестность» текущей строки  исходного кода
- `print <expression>` (`p <expr>`) — вывести значение выражения (можно использовать арифметические операторы, получение элемента массива, по-моему даже вызовы функций)

# Команды отладчика GDB (II)
- `where` (`backtrace`, `bt`) — показать стек вызовов функций:
```
(gdb) where
#0  0x08048681 in divint(int, int) (a=3, b=0) at crash.c:21
#1  0x08048654 in main () at crash.c:13
```
- `frame <n>` (`up`) — перейти на указанный уровень в стеке вызовов (чтобы посмотреть значения переменных в вызывающей функции)

- при нажатии Ctrl+C выполнение программы приостанавливается

# Команды отладчика GDB (III)
- `break <location>` (`b <loc>`) — установить точку останова в указанной локации:
    * *function* — на входе в функцию *function* в текущем файле
    * *filename:line* — на строке *line* в файле *filename*
    * *filename:function* — на входе в функцию *function* в файле *filename*
- `delete` — удалить все точки останова
- `clear <location>` — удалить точку останова в указанной локации
- `next` (`n`) — выполнить до следующей строки программы
- `step` (`s`) — выполнить одну строку, «проваливаясь» в вызываемые функции
- `continue` (`c`) — продолжить выполнение программы (до следующей точки останова или исключения)

Более подробно про использование GDB с примерами - в статье [С.А.Афонина](http://serg.tk/1/debug/gdb.html)

# Valgrind

[Valgrind](http://valgrind.org) — средство динамического (= во время выполнения) анализа программ.

Позволяет анализировать, в частности, следующие показатели:

- **ошибки доступа к памяти** (доступ по неинициализированному указателю, выход за границы массива, использование неинициализированных переменных, копирование «внахлёст», утечки памяти)
- частота не-попаданий в кэш (L1, L2)
- источники взаимных блокировок в многопоточной программе

# Запуск Valgrind

```bash
# Вместо ./a.out --fun 1 --size 2
valgrind ./a.out --fun 1 --size 2
```
```
==19182== Invalid write of size 4
==19182==    at 0x804838F: f (example.c:6)
==19182==    by 0x80483AB: main (example.c:11)
==19182==  Address 0x1BA45050 is 0 bytes after a block of size 40 alloc'd
==19182==    at 0x1B8FF5CD: malloc (vg_replace_malloc.c:130)
==19182==    by 0x8048385: f (example.c:5)
==19182==    by 0x80483AB: main (example.c:11)
# ...
==19182== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
==19182==    at 0x1B8FF5CD: malloc (vg_replace_malloc.c:130)
==19182==    by 0x8048385: f (a.c:5)
==19182==    by 0x80483AB: main (a.c:11)
```
