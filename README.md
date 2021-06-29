# sphinx-3-autoinstall
Скрипт автоматической установки и настройки Sphinx 3, на сервере Ubuntu 20.04

**Скрипт запускается под root!**

Практически все данные для этого скрипта были взяты из репозитория:
https://github.com/psilocyberunner/sphinxsearch-v3-install

Скрипт был написан для быстрого развертывания Sphinx 3.3.1 на сервере Ubuntu 20.04 в качетсве сервиса.

Команды выполняются из папки со скриптом:

`./sphinxsearch-v3-install.sh`

Стандартный запуск скрипта, проверяет наличие всех необходимых пакетов:
    mysql-server - для переноса данных из mysql в sphinx
    libmysqlclient-dev - для работы indexer
Стандартно идет ответ yes (при 10 секундном ожидании пакеты ставятся автоматически)

Дальше идет создание "рабочей" директории установщика (sphinxsearch-v3-install), в дальнейшем в эту директорию качается файл установки sphinx с серверов sphinx и распаковывается для дальнейших манипуляций.

Потом происходит копирование рабочих файлов sphinx в /usr/bin/, создание пользователя и группы sphinx, потом внутри систему создаются рабочие папки sphinx:
```text
/etc/sphinx - тут лежит конфиг sphinx.conf
/var/run/sphinx
/var/log/sphinx - тут лежат логи
/var/lib/sphinx
/var/lib/sphinx/data
```

Создается tmpfiles.d и sphinx.conf (либо копируется ваш конфиг файл, если он лежит в папке со скриптом установки), создается файл sphinx.service.

**Стандартный sphinx.conf настроен на rt индексы.**
```text
index default
{
	type = rt
    path = /var/lib/sphinx/data/default

    rt_mem_limit = 256M
    index_exact_words = 1
    morphology = stem_enru

    rt_field = el_name
	rt_field = el_price
	rt_field = el_catalog

	stored_fields = el_name, el_price, el_catalog

	rt_attr_uint = el_id
}

indexer
{
	mem_limit = 512M
}

searchd
{
	listen = 127.0.0.1:9312
	listen = 127.0.0.1:9306:mysql41
	listen = /var/run/sphinx/searchd.sock:sphinx

	log	= /var/log/sphinx/sphinx.log
	query_log = /var/log/sphinx/query.log

	read_timeout = 5
	client_timeout = 300

	max_children = 30

	persistent_connections_limit = 30

	pid_file = /var/run/sphinx/sphinx.pid
	binlog_path = /var/lib/sphinx/data

	seamless_rotate = 1

	preopen_indexes	= 1

	unlink_old = 1

	max_packet_size = 8M

	max_filters	= 256

	max_filter_values = 4096

	max_batch_queries = 32

	workers	= threads
}
```

И в конце спрашивается нужно ли запускать sphinx и добавлять в автозапуск, и копировать html документацию (sphinx3.html) если у вас на сервере есть директория /var/www/html (/var/www/html/sphinx3.html).

В конце удаляет рабочую папку скрипта установки (sphinxsearch-v3-install).

**Скрипт запускается под root!**

Так же в скрипте есть полное удаление sphinx из системы (поставленный этим скриптом):

`./sphinxsearch-v3-install.sh -delete`

Будет предложенно удалить так же mysql-server и libmysqlclient-dev (стандартный ответ no, который активируется сам через 10 секунд).

**Команды управления сервисом:**

`systemctl status sphinx` - статус сервиса
`systemctl start sphinx` - старт сервиса
`systemctl stop sphinx` - стоп сервиса

`systemctl enable sphinx` - вешаем сервис в автозапуск
`systemctl disable sphinx` - убираем сервис из автозапуска