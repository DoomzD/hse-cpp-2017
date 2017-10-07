# Семинар 7 (27 сентября 2017)

Обсуждали, какие бывают разные итераторы и что можно с ними делать.

### Какие бывают итераторы
Итераторы делятся на несколько типов в зависимости от того, какие операции с ними можно делать.
У типов есть иерархия: каждый очередной позволяет делать больше, чем предыдущий (кроме output_iterator, который стоит отдельно).

* input_iterator: можно только инкрементировать, нельзя переприсваивать
* forward_iterator: можно переприсваивать. Пример: forward_list
* bidirectional_iterator: можно декрементировать. Пример: map, set
* random_access_iterator: есть произвольный доступ (можно делать it += k и it1 - it2).
    Пример: vector

Многие шаблонные функции работают по-разному в зависимости от того, какой итератор им передать.

```cpp
vector<int> a;
lower_bound(a.begin(), a.end(), x); // O(log a.size())

set<int> b;
lower_bound(b.begin(), b.end(), x); // O(b.size() * log b.size())
// в этом случае правильно так:
b.lower_bound(x); // O(log b.size())
```

Есть разные вспомогательные итераторы. Например, так можно считать элементы из потока, сохранить в вектор и вывести их обратно.
```cpp
vector<int> a;
std::copy(
        std::istream_iterator<int>(std::cin),
        std::istream_iterator<int>(),
        std::back_inserter(a));

std::copy(
        a.begin(),
        a.end(),
        std::ostream_iterator<int>(std::cout));
```
