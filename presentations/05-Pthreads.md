# Добрый день, группа 312!

### Вторая задача
### Параллельное программирование с общей памятью
### Библиотека pthreads
### Средства синхронизации библиотеки pthreads

andrey.zenzinov@math.msu.ru  

https://zenderro.github.io/programming-semester-5/

<style>
div.left-column {
  width: 44%;
  float: left;
}
div.right-column {
  width: 44%;
  float: right;
}
div.small {
  font-size: 0.8em;
}
div.fullwidth img {
  width: 100% !important;
}
</style>

# Вторая задача

Вторая задача — параллельная реализация алгоритма решения линейной системы (решения матрицы) с использованием библиотеки pthreads. Нужно реализовать параллельную версию вашего метода из первой задачи. При этом программа должна принимать в качестве одного из агрументов количество потоков.

При увеличении потоков на достаточно больших матрицах (2000x2000, 4000x4000) программа должна ускоряться, на 2 потоках — хотя бы в 1.6 раза, на 4 потоках — в 2.5 раза. На одном потоке программа не должна быть существенно медленнее, чем однопоточная реализация

# Литература
1. К.Ю. Богачёв. Основы параллельного программирования.
2. [Мобильное программирование приложений реального времени, Лекции 1,2](http://www.intuit.ru/studies/courses/53/53/lecture/1569)

Небольшие примеры, введения, презентации:
- http://aivt.ftk.spbstu.ru/media/files/2012/course/parallel/mt1.pdf
- https://computing.llnl.gov/tutorials/pthreads/

# Программы исполняются последовательно
Программы на языке C транслируются в машинный код — последовательность простых команд для процессора:

<div class="left-column">
```c
void foo(int x) {
    if (x == 0) {
       printf("testing\n");
    }
}
```

Процессор затем исполняет эти команды по порядку.
</div>

<div class="right-column">
```gnuassembler
.globl	_foo
_foo:        ## @foo
    testl	%edi, %edi
    je	LBB0_2
    retq
  LBB0_2:
    leaq	L_str(%rip), %rdi
    jmp	_puts                

L_str:        ## @str
.asciz	"testing"
```
</div>

# Современные процессоры имеют несколько ядер
 — т.е. они могут выполнять несколько «потоков команд» (программ) одновременно.

2nd Generation (Sandy Bridge) Intel Core Processor:

![](images/inside_intel_sandy_bridge_quad_core_processor.jpg)

# Современные процессоры имеют несколько ядер

12nd Generation (Alder Lake) Intel Core Processor:

![](images/intel-12th-gen-core-ht3.webp)

# Потоки (threads)
— способ запуска нескольких потоков исполнения параллельно, с разделяемой памятью и ресурсами.
Т.е. можно обмениваться указателями, выполнять одновременное чтение и запись, и так далее. 
Если просто запустить несколько копий программы, у каждой копии будет своё пространство памяти и они не смогут взаимодействовать.

Для использования потоков нужно дать специальную команду «создать поток» и указать, какие команды этот поток должен исполнять.

В Linux (и на Mac) потоки реализованы согласно спецификации POSIX Threads.

В командную строку при компиляции добавить: `-pthreads` или `-lpthreads`

В исходный код:

```c
#include <pthread.h>
```

<div class="small">
POSIX (Portable Operating System Interface for Unix) — серия стандартов, описывающая интерфейсы операционной системы для разработки портируемых приложений.
</div>

# Потоки и общая память
Когда программа использует потоки, можно считать, что в этой программе одновременно выполняется несколько функций (или несколько копий одной функции). При этом в общем случае про взаимную последовательность выполнения команд функций (какой поток раньше считает элемент матрицы) ничего неизвестно.

![](images/5-4-figure-1.gif)

# Создание потока

```c
pthread_t thread;
int  pthread_create(pthread_t *thread,
     const pthread_attr_t *attr,
     void *(*start_routine)(void*),
     void *arg);
void pthread_exit(void *retval);
int  pthread_join(pthread_t thread, void **retval);
```

<div class="fullwidth">
```tikz
[line width=1pt, node distance=8cm, font=\Huge]
\usetikzlibrary{positioning,shapes,arrows}
\usetikzlibrary{fit}
        \node (main) {main()};
        \coordinate [right = of main] (fork);
        \coordinate [above right = of fork] (threadStart);
        \node [right = of threadStart] (threadEnd) {pthread\_exit};
        \coordinate [right = of fork, below right = of threadEnd] (join);
        \node [right = of join] (exit) {process exit};
        \draw (main) -- (fork);
        \draw (fork) -- node[sloped, anchor=center, above] {pthread\_create} (threadStart);
        \draw (fork) -- node[above] {\textbf{Main thread} (main)} (join);
        \draw (threadStart) -- node[above] {\textbf{Child thread} (start\_routine)} (threadEnd);
        \draw [densely dashed] (threadEnd) -- node[sloped, anchor=center, above] {pthread\_join} (join);
        \draw (join) -- (exit);
```
</div>

# Создание потока
```c
int  pthread_create(pthread_t *thread,
     const pthread_attr_t *attr,
     void *(*start_routine)(void*),
     void *arg);
```

`thread`
~ (выходной аргумент) — идентификатор потока

`attr`
~ атрибуты, можно оставить `NULL`

`start_routine`
~ указатель на функцию, главную для потока
`void *child_thread_func(void *arg) `{.c}

`arg`
~ указатель на аргумент, передаваемый главной функции

# Завершение потока

`pthread_exit` вызывать необязательно, можно просто возвращать значение через `return`{.c}

`pthread_join` ожидает завершения потока-аргумента и возвращает ненулевое значение, если поток вернул результат, который будет записан по адресу `retval`

# Мьютэксы (mutex)

Мьютэкс — это базовый элемент синхронизации потоков с двумя атомарными операциями — Lock (захватить мьютэкс) и Unlock (освободить мьютэкс).

Поток, в котором была первой вызвана функция `pthread_mutex_lock`, захватывает мьютэкс. Выполнение остальных потоков при вызове `pthread_mutex_lock` блокируется до вызова `pthread_mutex_unlock` потоком, захватившим мьютэкс.

```c
pthread_mutex_t mutex;
int  pthread_mutex_init(pthread_mutex_t *mutex,
     const pthread_mutexattr_t *attr);
int  pthread_mutex_destroy(pthread_mutex_t *mutex);
```
`attr` может быть `NULL`

```c
int  pthread_mutex_lock(pthread_mutex_t *mutex);
int  pthread_mutex_unlock(pthread_mutex_t *mutex);
```

# Мьютэксы

<div class="fullwidth">
```tikz
\node (SR) {Thread Red};
\node [above = 1cm of SR] (SB) {Thread Blue};

\coordinate [right = 0.8\linewidth of SR] (ER);
\coordinate [right = 0.8\linewidth of SB] (EB);

\draw [dotted] (SR) -- (ER);
\draw [dotted] (SB) -- (EB);

\node [right = 1cm of SR, fill=white] (R1) {($L1$)};
\node [right = 2cm of SB, fill=white] (B1) {($L1$)};
\node [right = 3cm of R1, fill=white] (R2) {($U1$)};
\coordinate [right = 3cm of B1] (B2);
\node [right = 1cm of B2] (B3) {($U1$)};

\draw [line width=1em, color=red] (SR) -- (R1);
\draw [line width=1em, color=red] (R1) -- (R2);
\draw [line width=1em, color=red] (R2) -- (ER);
\draw [line width=1em, color=blue] (SB) -- (B1);
\draw [line width=1em, color=blue] (B2) -- (B3);
\draw [line width=1em, color = blue] (B3) -- (EB);
```
</div>

($Li$) `lock(`$m_i$`)` <span class="four-em"></span> ($Ui$) `unlock(`$m_i$`)`

# Взаимная блокировка (deadlock)

<div class="fullwidth">
```tikz
\node (SR) {Thread Red};
\node [above = 1cm of SR] (SB) {Thread Blue};

\coordinate [right = 0.8\linewidth of SR] (ER);
\coordinate [right = 0.8\linewidth of SB] (EB);

\draw [dotted] (SR) -- (ER);
\draw [dotted] (SB) -- (EB);

\node [right = 1cm of SR, fill=white] (R1) {($L1$)};
\node [right = 2cm of SB, fill=white] (B1) {($L2$)};
\node [right = 3cm of R1, fill=white] (R2) {($L2$)};
\node [right = 2cm of B1] (B2) {($L1$)};
\node [right = 1cm of B2] (B3) {($U2$) - won't run};

\draw [line width=1em, color=red] (SR) -- (R1);
\draw [line width=1em, color=red] (R1) -- (R2);
\draw [line width=1em, color=blue] (SB) -- (B1);
\draw [line width=1em, color=blue] (B1) -- (B2);
```
</div>

При использовании более чем одного мьютэкса возможна ситуация, когда потоки блокируют друг друга — синий поток ожидает мьютэкс $1$ для того, чтобы потом разблокировать мьютэкс $2$, который требуется красному потоку для разблокировки мьютэкса $1$.

# Условные переменные (condvar)
С помощью условных переменных потоки могут передавать сигналы о готовности данных и т.п. Потоки, вызывающие `pthread_cond_wait` блокируются до момента, когда какой-либо активный поток вызовет `pthread_cond_signal` (разблокирует один поток) или `pthread_cond_broadcast` (разблокирует все ожидающие потоки)

```c
int  pthread_cond_init(pthread_cond_t *cond,
     const pthread_condattr_t *attr);
int  pthread_cond_destroy(pthread_cond_t *cond);
```

```c
int  pthread_cond_wait(pthread_cond_t *cond,
     pthread_mutex_t *mutex);
int  pthread_cond_signal(pthread_cond_t *cond);
int  pthread_cond_broadcast(pthread_cond_t *cond);
```

`mutex` — один и тот же для всех операций с фиксированной условной переменной

# Условные переменные (condvar)

<div class="fullwidth">
```tikz
[line width=1em]
\tikzset{every node/.style={fill=white}}
\node (SR) {Thread Red};
\node [above = 1cm of SR] (SB) {Thread Blue};
\node [below = 1cm of SR] (SG) {Thread Green};

\coordinate [right = 0.8\linewidth of SR] (ER);
\coordinate [right = 0.8\linewidth of SB] (EB);
\coordinate [right = 0.8\linewidth of SG] (EG);

\draw [dotted, line width=0.5pt] (SR) -- (ER);
\draw [dotted, line width=0.5pt] (SB) -- (EB);
\draw [dotted, line width=0.5pt] (SG) -- (EG);

\node [right=1cm of SB] (BW) {(W)};
\node [right=1.5cm of SG] (GW) {(W)};
\node [right=2.5cm of SR] (RS) {(S)};

\draw [color=blue] (SB) -- (BW);
\draw [color=green] (SG) -- (GW);
\draw [color=red] (SR) -- (RS);

\coordinate [right=1cm of BW] (BC);
\node [right=1cm of BC] (BW2) {(W)};
\draw [color=blue] (BC) -- (BW2);

\node [right=2cm of RS] (RB) {(B)};
\coordinate[right=3.6cm of GW] (GR);
\coordinate[right=1.1cm of BW2] (BR);

\draw [color=blue] (BR) -- (EB);
\draw[color=red] (RS) -- (RB);
\draw [color=red] (RB) -- (ER);
\draw [color=green] (GR) -- (EG);
```
</div>

\(W) — wait, (S) — signal, (B) — broadcast

# Синхронизация с условными переменными

Допускает использование внутри циклов, учитывается не только количество вошедших, но и вышедших задач.

```c
void synchronize(int total_threads)
{
    /* Объект синхронизации типа mutex */
    static pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    /* Объект синхронизации типа condvar */
    static pthread_cond_t condvar_in = PTHREAD_COND_INITIALIZER;
    /* Объект синхронизации типа condvar */
    static pthread_cond_t condvar_out = PTHREAD_COND_INITIALIZER;
    /* Число пришедших в функцию задач */
    static volatile int threads_in = 0;
    /* Число ожидающих выхода из функции задач */
    static volatile int threads_out = 0;

    /* "захватить" mutex для работы с переменными 
        threads_in и threads_out */
    pthread_mutex_lock(&mutex);
    /* увеличить на 1 количество прибывших в эту функцию задач */
    threads_in++;

    /* проверяем количество прибывших задач */
    if (threads_in >= total_threads) {
        /* текущий поток пришёл последним */
        threads_out = 0;
        /* устанавливаем начальное значение для threads_out */
        pthread_cond_broadcast(&condvar_in);
    } else {
        /* есть ещё не пришедшие потоки */
        /* ожидаем, пока в эту функцию не придут все потоки */
        while (threads_in < total_threads) {
            /* ожидаем разрешения продолжить работу:
               освободить mutex и ждать сигнала от 
               condvar, затем "захватить" mutex опять */
            pthread_cond_wait(&condvar_in, &mutex);
        }
    }
    /* увеличить на 1 количество ожидающих выхода задач */
    threads_out++;

    if (threads_out >= total_threads) {
        threads_in = 0;
        pthread_cond_broadcast(&condvar_out);
    } else {
        while (threads_out < total_threads) {
            pthread_cond_wait(&condvar_out, &mutex);
        }
    }
    /* освободить mutex */
    pthread_mutex_unlock(&mutex);
}
```

# Барьеры (barrier)
Барьеры определяют точки синхронизации, в которых потоки ожидают, пока до данной точки дойдёт заданное количество потоков.

```c
int pthread_barrier_init(pthread_barrier_t *barrier,
    const pthread_barrierattr_t *attr, unsigned count);
int pthread_barrier_destroy(pthread_barrier_t *barrier);

int pthread_barrier_wait(pthread_barrier_t *barrier);
```
`count` — количество потоков

Барьеры — наиболее простой примитив синхронизации, я советую использовать в задачах именно его.

# Барьеры (barrier)
<div class="fullwidth">
```tikz
[line width=1em]
\tikzset{every node/.style={fill=white}}
\node (SR) {Thread Red};
\node [above = 1cm of SR] (SB) {Thread Blue};
\node [below = 1cm of SR] (SG) {Thread Green};

\coordinate [right = 0.8\linewidth of SR] (ER);
\coordinate [right = 0.8\linewidth of SB] (EB);
\coordinate [right = 0.8\linewidth of SG] (EG);

\draw [dotted, line width=0.5pt] (SR) -- (ER);
\draw [dotted, line width=0.5pt] (SB) -- (EB);
\draw [dotted, line width=0.5pt] (SG) -- (EG);

\node [right = 2cm of SB] (BB) {(B)};
\node [right = 1cm of SR] (RA) {(A)};
\node [right = 2.5cm of SG] (GA) {(A)};
\node [right = 3cm of RA] (RB) {(B)};
\node [right = 2.5cm of GA] (GB) {(B)};

\draw [color=red] (SR) -- (RA);
\draw [color=green] (SG) -- (GA);
\draw [color=blue] (SB) -- (BB);

\coordinate [right = 1.7cm of RA] (CR1);
\coordinate [right = 1.1cm of RB] (CR2);
\coordinate [right = 4cm of BB] (CB);

\draw[color = red] (CR1) -- (RB);
\draw[color = red] (CR2) -- (ER);
\draw [color=green] (GB) -- (EG);
\draw [color=blue] (CB) -- (EB);
\draw [color=green] (GA) -- (GB);

\coordinate [above = 0.5cm of CR1] (B1T);
\coordinate [below = 2.3cm of CR1] (B1B);

\coordinate [above = 2.3cm of CR2] (B2T);
\coordinate [below = 2.3cm of CR2] (B2B);

\draw [color=gray, line width=3pt] (B1T) -- (B1B);
\draw [color=gray, line width=3pt] (B2T) -- (B2B);
```
</div>

\(A) wait (barrier count = 2)
\(B) wait (barrier count = 3)

# Схема распараллеливания задач
Количество потоков $P$ передаётся как аргумент командной строки.

В `main` создаётся $P$ потоков с одной и той же функцией, в которую передаются:

- общие для всех потоков данные: размер матрицы, указатель на матрицу и обратную матрицу (или правую часть), указатели на дополнительную общую память
- данные, специфичные для каждого потока: номер потока, указатели на собственную память.

Матрица распределяется по потокам-«владельцам» циклически, по строкам или по столбцам (1-я строка — 1-й поток, …, P-я строка — P-й поток; P+1-я строка — 1-й поток, …). Это делается для того, чтобы при уменьшении части матрицы, с которой работает метод, потоки не начинали простаивать. 

Лучше и распределять по потокам, и хранить элементы матрицы по столбцам.

# Псевдокод main
```small
    P, n, ... = получить аргументы командной строки
    matrix, inverse_matrix, tmp_common, tmp_specific[P] = выделить память
    matrix, inverse_matrix = считать входные данные
    threads = new pthread_t[P]
    thread_args = new thread_args[P]
    pthread_barrier_t barrier;
    pthread_barrier_init(&barrier, NULL, P)
    for p=[0…P):
      thread_args[p] ⟵ n, P, matrix, inverse_matrix, tmp_common, barrier
      thread_args[p] ⟵ tmp_specific[p], p
      pthread_create(threads+p, NULL, Func, thread_args+p)

    for i=[0, P):
      pthread_join(threads[p], NULL);

    очищаем память, уничтожаем потоки, барьер и т.п.
```

# Псевдокод func
```{.small .tall}
void* func(void* args) {
  thread_args A = (thread_args) args;
  n, P, matrix, inverse_matrix, tmp_common, barrier, tmp_specific, p ⟵ A

  for i=[0…n): - цикл по диагональным элементам
    j0 = i
    if i%P == p: — текущий поток — владелец текущего столбца
      подготовить в tmp_common данные для обнуления i-го столбца
      обнулить i-й столбец, j0 = i+1

    pthread_barrier_wait(barrier)

    for j=[j0…n):
      применить преобразование из tmp_common к своим столбцам матрицы
    for j=[0…n): — если обращение матрицы, иначе - без цикла
      применить преобразование из tmp_common к правой части

    pthread_barrier_wait(barrier) — можно убрать, если чередовать tmp_common
        по чётным-нечётным шагам

  for i=[0…n):
    обратный ход метода Гаусса
    pthread_barrier_wait(barrier)

  return NULL;
}
```

# Пример структуры с аргументами

```c
typedef struct {
    double *matrix;
    double *vector;
    double *reuslt;
    int n;
    int thread_num;
    int total_threads;
    pthread_barrier_t *barrier;
    double *tmp_common;
    double *tmp_specific;
} ARGS;
```

# Задание

1. Решение СЛУ: https://classroom.github.com/a/LWB2i2As
2. Обращение матрицы: https://classroom.github.com/a/SKNsdn5B

Предусмотрено три режима сборки Makefile:

- `make` - "быстрая" сборка с оптимизацией, используется для измерения времени работы программы на больших размерах матрицы;
- `make debug` - отладочная сборка для отладки с использованием `gdb`;
- `make test` - сборка для тестирования с проверками на на утечку памяти и на выход за границы массива. Эта версия программы будет использоваться для запуска тестов.

Для перехода между разными режимами необходимо выполнить команду `make clean`, которая удаляет все файлы, созданные во время сборки.

# Задание

Собранный бинарный файл называется `main`.

Запуск: `./main p n m k filename`

То же самое, что и в первой задаче, но добавляется параметр `p` - количество потоков.

Добавляется новый код ошибки: `-5`, если не удалось создать поток.


# Замечания
1. Если многопоточная программа, в которой не используются циклы `while` не завершается, скорее всего, имеет место блокировка (deadlock, не все потоки доходят до барьера и т.п.)
2. Чем больше синхронизаций, тем хуже ускоряется программа.
3. Если программа при нескольких запусках на нескольких потоках возвращает недетерминированный результат (при каждом запуске разный с одними и теми же входными данными), то проблема в несинхронизированном чтении.
4. **Внимание:** самостоятельно нужно сделать обработку ошибок, чтобы, если один поток решил, что матрица вырожденная, другие тоже об этом узнавали и заканчивали выполнение. Main тоже нужно как-то уведомить — через общую память в tmp_common, например.
5. Желательно распараллелить и умножение матрицы для вычисления невязки, чтобы можно было, например, дождаться результатов на 4000x4000
6. Одновременное обращение нескольких процессов к одним и тем же элементам памяти приводит к конфликтам в оперативной памяти и кэш-памяти, что ведёт к замедлению.
