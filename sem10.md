# Семинар 10 (5 октября 2017)

Чуть-чуть говорили про битмаски. Писали свой итератор с `iterator_traits`, обсуждали связанную с этим технику tag dispatch и то, как вызывать разные алгоритмы в зависимости от типа переданных итераторов.

```cpp
// быстрая реализация, рассчитанная на random access iterator
// (то есть позволяющий делать iter += k)
template<typename Iter>
bool findFast(Iter begin, Iter end, int x) {
    return std::binary_search(begin, end, x);
}

// медленная реализация, рассчитанная на bidirectional iterator
// (на самом деле достаточно forward iterator, поскольку мы его только
// инкрементируем)
template<typename Iter>
bool findSlow(Iter begin, Iter end, int x) {
    while (begin != end) {
        if (*begin++ == x) {
            return true;
        }
    }
    return false;
}

// Две функции, в качестве дополнительного параметра принимающие специальный
// тип, так называемый тег
template<typename Iter>
bool findImpl(Iter begin, Iter end, int x, std::random_access_iterator_tag) {
    return findFast(begin, end, x);
}

template<typename Iter>
bool findImpl(Iter begin, Iter end, int x, std::bidirectional_iterator_tag) {
    return findSlow(begin, end, x);
}

// Здесь мы смотрим на тег итератора (это тип, доступ к которому можно получить
// через iterator_traits) и передаём объект такого типа во вспомогательную
// функцию, позволяя определить, какой алгоритм надо вызывать
template<typename Iter>
bool myFind(Iter begin, Iter end, int x) {
    return findImpl(begin, end, x,
            typename std::iterator_traits<Iter>::iterator_category{});

}
```

Если мы хотим реализовать функцию для произвольного типа-тега, можно использовать троеточие:

```cpp
template<typename Iter>
bool findImpl(Iter begin, Iter end, int x, ...) {
    return findVerySlow(begin, end, x);
}
```

Ещё один вариант -- сделать необходимую функцию статическим методом шаблонного класса и сделать необходимые частичные специализации.

```cpp
template<typename Iter, typename Tag>
struct Caller {
    static bool findImpl(Iter begin, Iter end, int x) {
        return findSlow(begin, end, x);
    }
};

template<typename Iter>
struct Caller<Iter, std::random_access_iterator_tag> {
    static bool findImpl(Iter begin, Iter end, int x) {
        return findFast(begin, end, x);
    }
};

template<typename Iter>
bool myFind(Iter begin, Iter end, int x) {
    return Caller<Iter, typename std::iterator_traits<Iter>::iterator_category>::findImpl
        (begin, end, x);

}
```

Когда мы реализуем свой итератор, мы хотим, чтобы класс iterator_traits от него тоже был определён. Для этого нужно определить его явно.

```cpp
namespace std {

template<>
struct iterator_traits<RangeIterator> {
    // эти две строчки делают одно и то же, нужно использовать только одну
    using iterator_category = std::bidirectional_iterator_tag;
    typedef std::bidirectional_iterator_tag iterator_category;
};

} // namespace std
```

Здесь мы заходим внутрь неймспейса std (поскольку ``iterator_traits`` находится там, а специализировать класс нужно в том же неймспейсе, в котором он объявлен) и объявляем специализацию класса ``iterator_traits`` с параметром RangeIterator (это итератор, который мы писали на семинаре). После этого нужно определить в теле класса необходимые типы (через typedef/using).

Есть ещё несколько способов сделать так, чтобы ``iterator_traits`` заработал с вашим классом: например, унаследовать ваш итератор от std::iterator или реализовать тайпдефы `iterator_category`, `value_type`, `pointer`, `reference` и `difference_type` в теле самого итератора.

[Здесь](https://pastebin.com/vjCQSvV9) можно посмотреть пример: итератор-обёртку над T*.
