# Семинар 1 (6 сентября 2017)

Обсуждали базовые команды Unix/Linux, флаги компиляции из командной строки
и предупреждения компилятора.

### Перемещение в командной строке
| Команда | Действие                                     |
|---------|----------------------------------------------|
| cd      | Перейти в директорию                         |
| ls      | Вывести список файлов в текущей директории   |
| mkdir   | Создать поддиректорию                        |
| rm      | Удалить файл (*rm -r* удаляет директорию)    |
| touch   | Создать пустой файл                          |
| tree    | Вывести дерево директорий в красивом формате |


Ещё есть очень полезная команда *man*: если набрать *man some_command*, вы увидите
мануал по пользованию этой командой. Листать можно стрелочками, PgUp-PgDn или j/k.
На всякий случай: чтобы выйти, надо нажать *q* :)

Чтобы вернуться в родительскую директорию, нужно набрать «*cd ..*».

### Компиляция из командной строки
Возьмём простую программу.

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, world!" << std::endl;
}
```

Предположим, что программа находится в файле a.cpp; тогда для её компиляции
и запуска надо набрать

```sh
$ clang++ a.cpp
$ ./a.out
Hello, world!
```

*Здесь и далее (и во многих других местах, где вы столкнётесь с примерами работы*
*в командной строке) строки, начинающиеся с $, обозначают команды, которые вам*
*надо вводить, а все остальые строки – вывод команд.*


*a.out* – это [стандартное название](https://en.wikipedia.org/wiki/A.out) для бинарника (исполняемого файла), который
создаёт компилятор. Его можно менять с помощью опций компилятора. Опции чаще
всего имеют вид *-opt* или *-opt value*.

| Опция                   | Действие                                                                              |
|-------------------------|---------------------------------------------------------------------------------------|
| -o name                 | Сохранить бинарник в файл *name*                                                      |
| -O2                     | Включить оптимизацию 2-го уровня (самый практичный уровень для «бытового» применения) |
| -Wall                   | Включить предупреждения компилятора                                                   |
| -Wextra                 | Ещё больше предупреждений!                                                            |
| -std=c++11 (-std=c++14) | Включить поддержку 11-го стандарта (иначе *auto* работать не будет)                   |

### Зачем нужны оптимизация и предупреждения
#### Оптимизация

```cpp
#include <iostream>

int main() {
    long long s = 0;
    for (int i = 0; i < 50000; ++i) {
        for (int j = 0; j < 50000; ++j) {
            s += 1ll * i * j;
        }
    }
    std::cout << s << std::endl;
}

```

Попробуйте скомпилировать и запустить эту программу с *-O2* и без. Время выполнения уменьшится
с двух секунд практически до нуля: оптимизатор умеет сворачивать простые циклы. Естественно,
он умеет не только это, так что нет причин не использовать оптимизатор (пока вы не работаете с отладчиком).

#### Предупреждения («варнинги»)

```cpp
// выводим все элементы вектора, кроме последнего
std::vector<int> a;
for (int i = 0; i < a.size() - 1; ++i) {
    std::cout << a[i] << std::endl;
}

------

// компилировать c g++ -O2, на clang++ не воспроизводится
for (int i = 0; i < 5; ++i) {
    std::cout << i * 1000000000 << std::endl;
}

------

// a+b
int a, b;
std::cin >> a >> a;
std::cout << a+b << std::endl;
```

Попробуйте скомпилировать эти примеры с включёнными опциями *-Wall* и *-Werror* и без,
и вы увидите, сколько полезного компилятор может вам сказать в порой очень неочевидных
местах.

### Ещё немного команд командной строки
#### Просмотр больших файлов или длинного вывода команд
| Команда               | Действие                                  |
|-----------------------|-------------------------------------------|
| cat                   | Вывести файл на экран целиком             |
| head -n 20 / head -20 | Вывести первые 20 строк файла/потока      |
| tail -n 20 / tail -20 | Вывести последние 20 строк файла/потока   |
| less                  | Позволяет «листать» файл/поток вверх-вниз |

Все эти команды можно использовать двумя способами.
```sh
$ ./a.out | head
... first lines of output
$ head large_file
... first lines of file
```

Чтобы выйти из less, надо нажать *q*, как и в случае *man*. На самом деле команда *man* просто
находит файл с мануалом и открывает его через *less*, так что в этом нет ничего удивительного.


#### Всякое разное
| Команда               | Действие                                                   |
|-----------------------|------------------------------------------------------------|
| grep "some_word"      | Найти все строки, содержащие some_word                     |
| grep "some_word" -R . | Найти все файлы в текущей директории, содержащие some_word |
| sort                  | Сортирует все строки входа лексикографически               |
| wc                    | Считает количество символов, слов и строк входа            |


Полезные ключи:

* *grep -i*: игнорировать регистр при поиске
* *grep -w*: искать заданный текст только как слово, а не как подстроку
* *sort -n*: сортировать вход как числа, а не как строки
* *wc -l*: вывести только количество строк

Все эти команды также могут работать как со стандартным вводом, так и принимать
название файла как параметр.

### Бонус: удаление файла *-r*

Как вы помните, команда *-r* – это флаг команде *rm*, который говорит, что надо удалить
директорию целиком («r» значит «recursive»). Соответственно, удалить файл «-r» так просто
не получится. Тем не менее, это сделать можно.

* *rm ./-r*: удалить файл «-r», лежащий в текущей директории.
* *rm -*- *-r*: многие утилиты имеют специальную опцию «-*-*», которая говорит, что всё, что идёт
    далее – это не опции, а аргументы (в данном случае имена файлов для удаления).
* Наконец, через файловый менеджер :)
