# Проект - Бот поддержки Wallarm

## Версия - `0.1.2`

## Использованная база данных: 	`MySQL`

## Переменные окружения:

* `TELEGRAM_TOKEN` - токен бота
* `DBADDR` - адрес базы данных
* `DBNAME` - название базы данных
* `DBUSER` - имя пользователя
* `DBPASS` - пароль пользователя
* `LOGLEVEL` - уровень логирования (`info`, `warn`, `error`)

## Способы деплоя:

### Докер с вынесенной базой:

1. Нужен сервер mysql который торчит во вне (с адресом `DBADDR`) в котором есть:
    * База, имя которой будет в переменной окружения (`DBNAME`)
    * Пользователь, с правами на работу с базой
    * Токен от зарегистрированного бота в телеграм
2. Выгруженный докер-контейнер, например: `rikitikitavi1989/wallarm-support-bot`
3. `env.list` c упомянутыми выше переменными окружения.

Запуск осуществляется командой:

```bash
sudo docker run --rm --add-host=mysql_server: DBADDR --env-file ./env.list rikitikitavi1989/wallarm-support-bot
```


### Minikube или Kubernetes

1. Изучить и поправить при необзодимости yaml файлики в папрке helm
2. Выкачать в helm/wlrm_support_bot/charts/mysql офиц чарт mysql `helm fetch`
3. `helm install helm/wlrm_support_bot`

#### Быстро удалить из kubetnetes
1. Удаляем секреты 

```bash
for i in  `kubectl get secrets | grep -E '(mysql|support)' | awk '{print $1}' ` ; do kubectl delete secret $i ; done
```
2. Удаляем сервисы

```bash
for i in  `kubectl get services | grep -E '(mysql|support)'| awk '{print $1}' ` ; do kubectl delete service $i ; done
```
3. Удаляем деплойменты


```bash
for i in  `kubectl get deployments | grep -E '(mysql|support)'| awk '{print $1}' ` ; do kubectl delete deployment $i ; done	
```

## Фунцкционал:

* Cбор статистики по:
	* Сообщениям
	* Тикетам
	* Пользователям
	* Чатам
* Два языка (русский и английский)
* Ограничение прав по чатам:
	* `client_chat` - доступ к статистике по себе (проверка состояния тикетов)
	* `internal` - доступ к статистике по всем клиентам (отчет за указанный день), рассылка сообщений
	* `admin` - `internal` + полный доступ к статистике (чаты, пользователи и проч), получение технических уведомлений
	* `testing` - `admin` + тестовые сообщения
* Ограничение прав по типам пользователя:
	*  `client` - доступ к статистике о себе и смене языка бота
	*  `wallarm` - доступ к отправке сообщений и отчетам
	*  `admin` - `wallarm` + доступ к изменению свойств чатов, тикетов, пользователей

## Описание команд:

### Клиентские команды (доступ не ограничивается):

* `/my_tickets` - данные по состоянию тикетов, информация по которым есть в базе данных
* `/language (ru|en)` - переключение языка чата
* `/start` - приветственное сообщение бота
* `/help` - сообщение с описанием возможностей бота

### Команды для партнеров:

* `/client_tickets` - информация по тикетам клиентов партнера

### Команды для сотрудников организации:

* `/report DD.MM.YYYY` - статистика за выбранный день по чатам
* `/give_tickets` - информация по всем активным тикетам
* `/(clean_chat|chat_clean)` - очистка текущего чата
* `/say_to_(wallarm|admin|en|ru) text` - написать сообщение во внутренние|админские|англоязычные|русскоязычные чаты

### Команды для администраторов (поддержки)

* `/give_(users|chats|tickets)` - информация по активным пользователям, чатам, тикетам
* `/give_(users|chats)_disabled` - информация по отключтенным пользователям, чатам
* `/give_tickets_resolved` - информация по закрытым тикетам
* `/set_(chat|user|ticket)` - задание определенных параметров для чатов, пользователей, тикетов:
  * Для пользователей, параметры: `[phone, username, name, type, enabled]`, пример: `/set_user userid type admin`
  * Для чатов, параметры: `[clientid, partnerid, client, language, type, enabled]`
  * Для тикетов, параметры: `[status, clientid, partnerid, resolved]`
* `/delete_(chat|user) id` - отключение чатов, пользователей
* `/close_ticket id` - закрыть тикет
* `/add_(chat|ticket) id` или `/(chat|ticket)_add id` - добавить вручную новый чат, тикет


## Идеи для следующих версиий:
* Ввести уведомление по не отвеченным чатам в течение 15 минут (отдельный поток)
* Переделать работу с правами пользователей:
  * Ввести переменные, которые будут работать без подключения к базе данных
  * Сделать обновление этих переменных, запускаемое при изменении уровня пользователя (чата) или раз в 15 минут
  * Убрать запрос сверки с базой по правам при каждом сообщении
* Расширить список атакующих векторов для атак, вывести функционал проверки в открытую часть
* Добавить функционал отправки файлов ботом
* Добавить выгрузку отчетов в csv
* Интеграция с jira (формирование отчетов по открытым/закрытым тикетам на SRV, слежение за тикетами SRV по клиентам)
* В больших глобальных планах создание общей системы поддержки, для этого:
  * Написание ботов с тем же функционалом, по возможности использующих ту же базу:
    * slack
    * skype
  * Написание админки на rails со следующим функционалом:
    * Управление данными, которые собирают боты
    * Планировщик задач
    * Календарь рабочих смен
    * План-факт график рутинной работы
    * Создание отчетности и выгрузка в csv
  * Работа с базовым функционалом внешних роутов api


