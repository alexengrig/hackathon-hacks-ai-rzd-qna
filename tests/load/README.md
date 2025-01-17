
# Нагрузочное тестирование API с помощью k6

Этот проект содержит скрипт для нагрузочного тестирования API, который обрабатывает вопросы пользователей. 
Базовое требование к нагрузке - **не предъявляются**.
Среднее время ответа не должно превышать **1 мин**.


## Требования

- macOS (или любая другая система на базе Unix)
- [k6](https://k6.io) для нагрузочного тестирования
- Node.js для возможных расширений скриптов

## Установка

### Шаг 1: Установка k6

Установите `k6` с помощью Homebrew на macOS:

```bash
brew install k6
```

Для других операционных систем следуйте инструкциям на официальной [странице установки k6](https://k6.io/docs/getting-started/installation).

### Шаг 2: Клонирование репозитория

Склонируйте репозиторий и перейдите в директорию `tests`:

```bash
git clone https://github.com/airndlab/hackathon-hacks-ai-rzd-qna.git
cd tests
```

### Шаг 3: Подготовьте файлы с вопросами

Создайте файлы с именем `questions_subject.txt` в каталоге `tests/load/faq_questions/`. Каждый вопрос должен находиться на новой строке, например:

```txt
Какие льготы мне полагаются?
Какие льготы предоставляются членам семьи?
Как часто индексируется зарплата?
```

Этот файл будет использоваться для динамической отправки вопросов к API.

## Скрипт для нагрузочного тестирования

Основной скрипт для тестирования находится в файле `k6.js`. Он выполняет следующие действия:
- Загружает вопросы из файлов `*.txt`.
- Отправляет HTTP POST запросы к API с частотой 10 запросов в секунду.
- Проверяет, что API возвращает ответ с кодом 200 и содержит обязательные поля (`answer`).

### Настройки

Вы можете изменить этапы нагрузочного тестирования (увеличение нагрузки, основное тестирование и уменьшение нагрузки) в разделе `options` скрипта:

```js
export let options = {
  stages: [
    { duration: '1m', target: 10 }, // Плавное увеличение до 10 запросов в секунду за 1 минуту
    { duration: '5m', target: 10 }, // 5 минут тестирования с нагрузкой 10 запросов в секунду
    { duration: '1m', target: 0 },  // Плавное завершение теста
  ],
};
```

### Запуск теста

Запустите следующий команду для выполнения теста:

```bash
k6 run k6.js
```

### Описание скрипта

- **`k6.js`**: скрипт для выполнения нагрузочного тестирования с помощью k6.
- **`questions.txt`**: текстовые файл с вопросами пользователей, которые случайным образом отправляются в запросах к API.

### Проверка ответа

Скрипт проверяет, что каждый ответ содержит:
- Статус HTTP 200.
- Поля `answer`, которые не должны быть пустыми.

```js
check(res, {
  'status was 200': (r) => r.status === 200,
  'response contains id': (r) => JSON.parse(r.body).id !== '',
  'response contains answer': (r) => JSON.parse(r.body).answer !== '',
});
```

### Метрики нагрузочного теста

Во время теста k6 автоматически собирает метрики, включая:
- **Запросы в секунду (RPS)**
- **Время ответа**
- **Процент успешных и неуспешных запросов**

Подробный отчет можно увидеть в терминале после завершения теста.

### Кастомизация скрипта

Вы можете изменить URL API, данные запроса или заголовки в скрипте, чтобы адаптировать его под свои нужды:

```js
const url = 'https://api.yourservice.com/your-endpoint'; // Укажите ваш конечный URL API
```

### Вывод результатов

После завершения теста k6 покажет результаты, включая количество успешных запросов, среднее время ответа и другие важные метрики.