# Поймай баг, если сможешь

В этом задании Вам предстоит поймать и исправить пачку багов, которые нашли при тестировании приведенного ниже кода.

```cpp
#include <memory>
#include <string>
#include <iostream>
#include <cassert>
#include <thread>
#include <mutex>
#include <numeric>
#include <vector>

enum ActionItem : int {
    kStartServer,
    kStopServer,
    kUpdateConfiguration,
    kClearCache,
};

template<class T>
class queue {
private:
    struct node {
        node(const T& value_, std::unique_ptr<node> next_)
            : value{value_}
            , next{std::move(next_)}
        {}
        T value;
        std::unique_ptr<node> next;
    };

    std::unique_ptr<node> head;
    node* tail;
    std::unique_ptr<std::mutex> mutex_ptr;

public:
    queue()
        : head{std::make_unique<node>(T{}, nullptr)}
        , tail{head.get()}
        , mutex_ptr{std::make_unique<std::mutex>()}
    {}

    queue(const queue&) = default;
    queue(queue&&) = default;
    queue& operator=(const queue&) = default;
    queue& operator=(queue&&) = default;

    void push(const T& value) {
        std::lock_guard<std::mutex> _{*mutex_ptr};
        tail->value = value;
        tail->next = std::make_unique<node>(T{}, nullptr);
        tail = tail->next.get();
    }

    T front() {
        std::lock_guard<std::mutex> _{*mutex_ptr};
        return head->value;
    }

    bool empty() const {
        std::lock_guard<std::mutex> _{*mutex_ptr};
        return head.get() == tail;
    }

    void pop() {
        std::lock_guard<std::mutex> {*mutex_ptr};
        head = std::move(head->next);
    }
};

using ActionQueue = queue<ActionItem>;

std::string to_string(int action) {
    switch (action) {
        case kStartServer:
            return "Start server";
        case kStopServer:
            return "Stop server";
        case kUpdateConfiguration:
            return "Update configuration";
        default:
            return nullptr;
    }
}
```

## Bug tracker report

1. После добавления инструмента для импорта данных из других форматов приложение стало падать при завершении. Падение воспроизводится на прогонах тестов на ночных сборках, но не воспроизводится на релизной сборке. Изменился сценарий тестов, ранее тестировщики использовали небольшое количество предзаготовленых данных, теперь используют результаты импорта реальной системы.

2. После имплементации механизма drag'n'drop в одной из таблиц в ui продукта приложение стало падать, при добавлении новых данных в колонку, после перетаскивания всех данных из этой колонки в другую.

3. После начала тестирования продукта в мультиклиентном окружении приложение стало случайно падать при одновременной работе двух и более клиентов

4. После добавления нового режима работы сервера приложение стало падать

## Ваша миссия

Для каждого бага предложите регрессионный тест и исправления в коде.
За лучшие решения Вас ожидают призы на стенде Лаборатории Касперского

Ответы присылайте по адресу Pavel.Filonov@kaspersky.com



