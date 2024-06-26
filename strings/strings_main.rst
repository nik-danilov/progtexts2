.. highlight:: cpp

Строковые алгоритмы
===================

Теорию подготовил Никита Данилов. Вопросы по этой статье можно писать мне в телеграм: https://t.me/nik_danilov.

Хэши
----

Обычно хэш объекта - это число, являющееся сокращённой информацией об этом объекте. Хэши создаются так, чтобы у двух одинаковых объектов хэши совпадали, а у разных с огромной вероятнотью различались. Благодаря этому можно быстро сравнивать объекты, сравнивая соответствующие им числа. Здесь я буду говорить только о хэшировании строк, но в целом хэшировать можно почти любые объекты.

Ещё раз - наша задача сопоставить строкам такие числа, называющиеся хэшами, чтобы у двух одинаковых строк хэши совпадали, а у разных почти всегда различались.

Пусть есть какие-то константы `B` и `MOD`, про них мы подробнее поговорим потом.

Пусть `pows` это массив степеней `B`, где `pows[i]` это `B` в степени `i`.

Тогда пусть хэш строки `s`, длины `n` это:

`( (s[0] - "a" + 1) * pows[n - 1] + (s[1] - "a" + 1) * pows[n - 2] + ... + (s[n - 1] - "a" + 1) * pows[0] ) % MOD`

Насчитаем хэши у префиксов строки следующим образом:

::
   
   hash[0] = 0; // у нулевого префикса хэш нулевой
   for (int i = 1; i <= s.size(); i++) {
       hash[i] = (hash[i - 1] * b + (s[i - 1] - 'a' + 1)) % MOD;
   }

Теперь предположим что мы хотим узнать хэш подстроки с символа `l` по символ `r`. Хэш этой подстроки будет равен `((hash[r] - hash[l] * pows[r - l]) % MOD + MOD) % MOD`.

Тогда чтобы сравнить две подстроки, мы можем просто сравнить их хэши, а это делается за `O(1)`.

А можем ли мы сравнить две строки не на равенство, а на больше-меньше? Да, можем за `O(log(N))`. Найдём у двух строк наибольший совпадающий префикс при помощи бинпоиска, а затем посмотрим на следующую букву. Где она больше, та строка и больше.

Коллизии хэшей
--------------

Теперь о выборе магических констант `B` и `МOD`. Базовые правила выбора этих констант такие:

1. Константа `B` должна быть не меньше размера "алфавита".  То есть если ваши символы это только строчные латинские буквы, значит размер алфавита равен 26.  Тогда можно взять `B = 29`.  Если при этом могут быть цифры и заглавные латинские, то размер алфавита уже 62.  Тогда можно взять `B = 67`.

2. Обычно модуль хэшей `MOD` должен быть порядка `10^9`, чаще всего используют `10^9 + 7` или `10^9 + 9`.

3. Обе константы обычно берут простыми.

А теперь важный момент. Бывает такое, что вы написали всё это правильно, но заслав задачу получаете Wrong Answer на каком-нибудь `57` тесте. Тогда есть большая вероятность, что вы столкнулись с коллизией.

Почему это произошло? Просто так неожиданно совпало, что хэши двух строк одинаковые, а строки разные. Да, вероятность этого кажется маленькой, всё-таки числа до 10^9, но такое всё-же бывает. Жюри олимпиад иногда даже специально делают "анти-хэш-тесты", для самых популярных модулей хэширования.

Что делать? Есть `2` классических варианта:

1. Просто поменять `B` и `MOD` на какие-нибудь другие. Я иногда прямо во время раунда быстренько пишу код, который ищет несколько простых порядка `10^9`, чтобы дальше их использовать. С большой вероятностью через несколько посылок ваш код зайдёт.

2. Второй вариант чуть дольше, но зато надёжнее. Можно написать двойное хэширование. Просто сравнивать хэши сразу по двум модулям. Это будет чуть дольше, но вероятность ошибки станет почти нулевая.

.. important::
    
    Чаще всего, если хэши не заходят по времени, это происходит из-за долгой операции взятия по модулю. Но иногда это на самом деле можно ускорить.
    
    Например можно заменить (hash[r] - (hash[l] * pows[r - l]) % MOD + MOD) % MOD == (hash[r2] - (hash[l2] * pows[r2 - l2]) % MOD + MOD) % MOD
    
    Нa ((hash[r] - hash[l] * pows[r - l] + hash[r] - hash[l] * pows[r - l]) % MOD + MOD) % MOD == 0
    
    Тем самым ускорить код.

Ну и есть ещё довольно классический приём, работающий на с++. Если вы написали самые обычные хэши, а они не заходят по времени, можно попробовать заменить модуль `MOD` на `2^64`. Что? Нужен же простой модуль? Да, обычно лучше брать простой модуль, но в целом иногда модуля `2^64` хватает. На самом деле брать по модулю `2^64` мы можем автоматически, просто используя переполнение long long. Тем самым мы можем просто  никогда не брать по модулю, и всё равно хэши могут работать.

Поиск палиндромов при помощи хэшей
----------------------------------

Нужно понять, какой наибольший палиндром с центром в текущей позиции строки существует. Заметим, что здесь можно применить бинпоиск, так как существует какое-то `х` такое, что для любого `у <= х`, подстрока с `i - y` по `i + y` палиндром, а для любого `y > x` это неверно. А для конкретного `у` проверить условие легко. Нужно просто изначально захэшировать не только строку, но и перевёрнутую её же, а затем проверять что хэши подстроки и её же в обратном варианте совпадают.

Таким образом, мы можем найти все палиндромы в строке, запустив этот бинпоиск от всех возможных центров палиндромов.

.. important::
    
    Важный трюк. Чтобы не разбирать случаи когда палиндромы чётной и нечётной длины отдельно, можно вставить "#" между соседними элементами строки. Тогда нужно будет проверять только палиндромы нечётной длины, так как палиндромы чётной будут с центром в одном из "#". Здесь "#" должен быть символом, который точно не встретиться в строке, поэтому если хэштеги в строке возможны, замените его на какой-то другой символ.

Префикс-функция
---------------

Назовём суффикс собственным, если он не совпадает со всей строкой.

Префикс функция строки - массив чисел, являющихся ответом на вопрос, какой максимальный собственный суффикс совпадает с префиксом, у текущего префикса строки.

Например если есть строка `flipflapflip`, её префикс функция равна:

`0 0 0 0 1 2 0 0 1 2 3 4`

Пояснение: например `prefix_function[5]` здесь равно `2`, так как у префикса `flipfl` максимальный собственный суффикс совпадающий с префиксом имеет длину `2`. Аналогично для строки `flipflapfl`, поэтому `prefix_function[9] = 2`. Но например для строки `flipflap` никакой собственный суффикс с префиксом не совпадает, поэтому `prefix_function[7] = 0`.

Зная префикс-функцию строки, можно например за `O(N)` искать все вхождения подстроки в строку. Пусть мы хотим найти вхождения строки `t` в строку `s`. Тогда сделаем строку `x = t + "#" + s`. Здесь "#" должен быть символом, который точно не встретиться в строке, поэтому если хэштеги в строке возможны, замените его на какой-то другой символ. Тогда если мы посчитаем префикс функцию за `O(N)`, а затем пройдёмся по позициям строки `x`, то вхождение `t` в `s` равносильно элементу префикс-функции, равному длине `t`. Элементов больше `t` быть не может, так как символ "#" не встречается в `s`.

КМП
---

Алгоритм Кнута-Морриса-Пратта помогает быстро вычислять префикс-функцию строки. Чтобы делать это быстрее, нужно использовать для вычисления `i` ответы для предыдущих. 

Заметим, что следующее значение не больше чем на `1` превосходит предыдущее значение, так как иначе мы можем увеличить ответ для предыдущей.

Тогда сделаем следующее. Посмотрим на значение предыдущей префикс-функции. Пусть мы хотим найти значение префикс-функции для позиции `i`, а значение для `i - 1` это `j`. Если `s[j + 1] == s[i]`, то `prefix_function[i] = prefix_function[i - 1] + 1`, то есть мы тривиально продлили предыдущую строку на один символ. Иначе нам нужен максимальный суффикс подстроки до `i - 1` меньший `j`, а это значение мы знаем, так как это `prefix_function[j]`. Тогда заменяя `i - 1` на `j` будем продолжать эти действия, пока не найдём совпадающий символ. Если мы его не нашли, значит значение функции `0`. 

::
    
    int n = s.size();
    vector<int> prefix_function(n, 0);
    for (int i = 1; i < n; i++) {
        // префикс функция точно не больше предыдущего значения + 1
        int cur = prefix_function[i - 1];
        // уменьшаем значение cur, пока новый символ не совпадёт
        while (s[i] != s[cur] && cur > 0)
            cur = prefix_function[cur - 1];
        // здесь либо s[i] == s[cur], либо cur == 0(что значит что мы не нашли совпадений)
        if (s[i] == s[cur])
            prefix_function[i] = cur + 1;
    }

Нетрудно понять, что это будет работать за `O(N)`, несмотря на вложенный цикл, ведь суммарно внутренний цикл пройдёт максимум `O(N)` итераций.

Z-функция
---------

Z-функция (также иногда N-функция) - в каком-то смылсе похожа на префикс-функцию. Теперь нас для каждой позиции интересует, какая самая длинная подстрока с началом в текущей позицией совпадает с префиксом строки.

Например для строки `flipflapflip`, z-функция будет такой:

`12 0 0 0 2 0 0 0 4 0 0 0`

А для строки `abacabadabacaba`:

`15 0 1 0 3 0 1 0 7 0 1 0 3 0 1`

При помощи z-функции также можно искать вхождения подстроки в строку. Здесь всё аналогично префикс функции. Пусть мы хотим найти вхождения строки `t` в строку `s`. Тогда сделаем строку `x = t + '#' + s`. Здесь '#' должен быть символом, который точно не встретиться в строке, поэтому если хэштеги в строке возможны, замените его на какой-то другой символ. Тогда если мы посчитаем z-функцию за `O(N)`, а затем пройдёмся по позициям строки `x`, то вхождение `t` в `s` равносильно элементу z-функции равному длине `t`. Элементов больше `t` быть не может, так как символ '#' не встречается в `s`.

Вычисление z-функции
--------------------

Z-функцию, также как и префикс-функцию, можно насчитывать за `O(N)`. Будем снова использовать уже известную информацию. Будем хранить самую правую из текущих z-фукций (оба конца, и левый и правый), `l` и `r`. Теперь когда мы хотим посчитать текущую z-функцию, будем использовать уже известную информацию. `z[l] = r - l, z[i] = min(r - i + 1, z[i - l])`, так как строки с `0` по `r - l` и с `l` по `r` совпадают. Если после этого `z[i] = r - i + 1`, будем просто двигать `r` пока символы совпадают. Тогда суммарно мы сделаем не более `O(N)` операций.

::
    
    int n = s.size();
    vector<int> z(n, 0);
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        // если правая граница всё ещё не левее i
        if (i <= r)
            // мы можем попробовать его инициализировать z[i - l],
            // но не дальше правой границы: там мы уже ничего не знаем
            z[i] = min(r - i + 1, z[i - l]);
        // дальше каждое успешное увеличение сдвинет блок на единицу
        while (i + z[i] < n && s[z[i]] == s[i + z[i]])
            z[i]++;
        // проверим, правее ли мы текущего r, и если да, то обновим границы самого правого блока
        if (i + z[i] - 1 > r) {
            r = i + z[i] - 1;
            l = i;
        }
    }

Алгоритм Манакера
-----------------

Сразу скажу, что можно проделать трюк, который мы уже проделывали с хэшами и вставить между соседними символами "#", чтобы все существующие палиндромы в строке стали нечётной длины.

И снова искать мы будем массив, длины `N`, где для позиции `i` будет значение, равное максимальной длине палиндрома с центром в элементе `i`.

Если искать его тривиально, это будет работать за `O(N * N)`, поэтому будем делать что-то напоминающее z-функцию. Также будем сохранять самую правую границу. И теперь брать палиндром, не превышающий рамки известного, и соответсвенно если он превышает, будем его сокращать. То есть если есть палиндром с позиции `l` по `r` и текущая позиция `i`, по `manaker[i] = min(r - i, manaker[l + r - i - 1])`, а далее аналогично z-функции, если `manaker[i] = r - i`, будем двигать по единице, пока не найдём неравные элементы.

::
    
    int n = s.size();
    vector<int> manaker(n, 0);
    int l = -1, r = -1;
    for (int i = 0; i < n - 1; i++) {
        if (i < r) { // используем известное
            manaker[i] = min(r - i, manaker[l + r - i - 1]);
        }
        while (i - manaker[i] >= 0 && i + manaker[i] + 1 < n && s[i - manaker[i]] == s[i + manaker[i] + 1]) { // продолжаем двигать
            manaker[i]++;
        }
        if (i + manaker[i] > r) {
            l = i - manaker[i] + 1, r = i + manaker[i]; // обновляем самую правую границу при необходимости
        }
    }


