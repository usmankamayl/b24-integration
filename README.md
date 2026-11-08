# Интеграция Bitrix Framework (БУС / Б24) с внешними системами

[![CI](https://github.com/usmankamayl/b24-integration/actions/workflows/ci.yml/badge.svg)](https://github.com/usmankamayl/b24-integration/actions/workflows/ci.yml) [![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Слой интеграции для сайтов на Bitrix Framework: HTTP-клиент с таймаутами и защитой от утечки
ключей, идемпотентная отправка заказов во внешние системы, переключение окружений dev/prod,
логирование с маскированием секретов.

Код на `declare(strict_types=1)`, в одном стиле, с автотестами на фейковом транспорте:
реальные запросы для проверки не нужны.

## Что здесь доказывается

| Задача | Решение в коде |
|---|---|
| Не отправить заказ дважды при обрыве связи | `Idempotency-Key` и `externalId` в `ServiceRequestClient`, метод `reconcile()` для сверки статуса |
| Не потерять ключи при редиректе на чужой домен | `HttpClient` запрещает автопереходы и валидирует базовый URL |
| Сайт не висит при недоступной внешней системе | раздельные `timeout` и `connectTimeout`, `TransportException` с кодом ошибки |
| Секреты не попадают в лог | `Config/Logger` маскирует токены и пароли перед записью |
| Одинаковый код на dev и prod | `Config/Environment` и `config/environments.php` |

## Структура

```
php_interface/
├── classes/Training/
│   ├── Integration/HttpClient.php           # транспорт: cURL, таймауты, запрет редиректов
│   ├── Integration/ServiceRequestClient.php # бизнес-логика: идемпотентность, сверка, валидация
│   ├── Integration/TransportException.php   # типизированная ошибка транспорта
│   ├── Config/Environment.php               # выбор окружения
│   └── Config/Logger.php                    # логирование с маскированием секретов
├── config/environments.php                  # настройки dev/prod
├── init.php                                 # автозагрузка
├── events.php                               # обработчики событий ядра
└── tests/                                   # запускаемые автотесты
scripts/check_s02_1.php                      # проверочный скрипт для одного из сценариев
```

На боевой инсталляции всё это кладётся в `/local/php_interface/`.

## Запуск

Требуется PHP 8.1+ с расширением cURL.

```bash
# автотесты: 26 проверок на фейковом транспорте, без сети
php php_interface/tests/run.php

# проверка синтаксиса всех файлов
find php_interface -name '*.php' -exec php -l {} \;
```

Оба шага выполняются в CI на каждый пуш, конфигурация в `.github/workflows/ci.yml`.

## Конфигурация

Скопируйте `php_interface/config/.environment.example` в `php_interface/config/.environment`
(в репозиторий он не попадает) и заполните базовый URL, ключ и окружение.
Секретные значения читаются только из окружения, в коде их нет.
