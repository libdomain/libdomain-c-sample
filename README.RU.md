# Документация для Примера Программы по Поиску LDAP с использованием библиотеки libdomain

## Обзор

Предоставленный пример - это программа на языке C, которая демонстрирует использование библиотеки libdomain для выполнения операции поиска LDAP.
Программа устанавливает соединение с сервером LDAP, выполняет поиск в указанном сервере каталога.

## Компиляция

Для компиляции программы необходимо установить библиотеку libdomain.

```bash
apt-get install git cmake make gcc krb5-kinit libdomain-devel libconfig-devel
```

Склонируйте пример:
```bash
git clone https://github.com/libdomain/libdomain-c-sample
```

Используйте следующую команду компиляции:

```bash
cd libdomain-c-sample && mkdir build && cd build && cmake .. && make -j `nproc`
```

## Использование

Программа принимает аргументы командной строки для указания параметров соединения с сервером LDAP и параметров поиска.
Для этого примера мы предполагаем что у нас развёрнут тестовый стенд на основе Samba DC названием `dc` в домене `example.org`,
в котором LDAP сервис работает на порту 389. 

Убедитесь что используете правильные данные для подключения, такие как имя пользователя и пароль и.т.д. В случае если имя домена или сервис LDAP отличаются вам может потребоваться изменить параметры передаваемые примеру в соответствии с настройками сервиса LDAP.

Samba и Windows Active Directory могут поддерживать аутентификацию Kerberos в зависимости от конфигурации. Если сервер поддерживает GSSAPI и Kerberos, убедитесь, что вы передали параметер "--sasl". В этом случае вам также может потребоваться сгенерировать билет Kerberos для этого может потребоваться выполнить команду kinit.

```bash
kinit administrator@example.org
```
Для запуска примера выполните:

```bash
./libdomain-c-sample --host ldap://dc.example.org:389 --user administrator --pass password --bind "dc=example,dc=org" --sasl
```

### Опции командной строки

- `--host (-h)`: Указывает сервер LDAP в формате "протокол://адрес:порт", например, "ldap://example.org:389".
- `--user (-u)`: Указывает имя пользователя LDAP.
- `--pass (-w)`: Указывает пароль LDAP.
- `--bind (-b)`: Указывает DN (Distinguished Name) для привязки к LDAP, например, "dc=example,dc=org".
- `--sasl (-s)`: Включает использование SASL-привязки.

## Структура Программы

1. **Функции обработки событий:**
   - `exit_callback`: Вызывет событие завершения программы.
   - `connection_on_error`: Обрабатывает ошибки во время операций LDAP.
   - `connection_on_update`: Обрабатывает обновления состояния соединения и обрабатывает ошибки во время установки соединения.

2. **Обработка Опций:**
   - `parse_opt`: Разбирает опции командной строки с использованием библиотеки argp.

3. **Основная Функция:**
   - Сначала создаётся новый контекст talloc при помощи функции `talloc_new`.
   - Инициализируется структура `arguments` которая служит для хранения аргументов командной строки.
   - При помощи функии `argp_parse` производится обработка аргументов командной строки.
   - Происходит проверка обязательных аргументов host и bind_dn если эти аргументы не найдены программа завершается.
   - Создаётся структура для конфигурации для подключения к серверу LDAP, при помощи функции `ld_config`.
   - Инициализируется основной указатель библиотеки `handle` при помощи функции `ld_init`.
   - Устанавливаются стандартные обработчики событий `ld_install_default_handlers(handle)`.
   - Устанавливается обработчик с основной логикой программы: `ld_install_handler(handle, connection_on_update, update_interval)`.
   - Устанавливается обработчик который выключает программу через 10 секунд: `ld_install_handler(handle, exit_callback, exit_time)`.
   - Устанавливается обработчки ошибок.
   - Запуск основного цикла событий с помощью `ld_exec`.
   - Выполняется поиск.
   - Очищаются ресурсы при помощи функций: `ld_free(handle)` и `talloc_free(talloc_ctx)`.

## Обработка Ошибок

Ошибки, такие как невозможность выполнения операций LDAP, приводят к завершению программы с соответствующим сообщением об ошибке.
Обработка ошибок реализована в функции `connection_on_error`. Однако обработка ошибок соединения происходит в функции `connection_on_update`.

## Примечания

- Программа выполняет LDAP-поиск при изменении состояния соединения на `LDAP_CONNECTION_STATE_RUN`.
- Объекты для поиска задаются в переменной `LDAP_DIRECTORY_ATTRS`.

## Дополнительная Информация

- https://www.gnu.org/software/libc/manual/html_node/Argp-Example-3.html

## Информация о Версии

- Версия программы: 1.0.0

## Лицензия

Эта программа распространяется под лицензией GPLv2. Смотрите сопроводительный файл LICENSE для получения подробной информации.
