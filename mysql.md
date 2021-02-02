---
title: 'MySQL шпаргалки / Хабрахабр'
viewport: width=1024
---

<div class="post_show">

<div id="post_105954" class="post shortcuts_item">

<div class="published">

11 октября 2010 в 19:46

</div>

<div class="hubs">

[MySQL](http://habrahabr.ru/hub/mysql/ "Вы подписаны на этот хаб"){.hub
.subscribed}<span class="profiled_hub" title="Профильный хаб">\*</span>

</div>

<div class="content html_format">

Часто, когда разрабатываешь сайт, замечаешь, как на одни и те же грабли
наступают разработчики при проектировании базы данных.\
\
Сегодня я решил опубликовать свои шпаргалки, на самые часто
встречающиеся ошибки при работе с MySQL.\
\
[]()\
#### Работа с бекапами

\
\
Делаем бекап\
`mysqldump -u USER -pPASSWORD DATABASE > /path/to/file/dump.sql`\
\
Создаём структуру базы без данных\
`mysqldump --no-data - u USER -pPASSWORD DATABASE > /path/to/file/schema.sql`\
\
Если нужно сделать дамп только одной или нескольких таблиц\
`mysqldump -u USER -pPASSWORD DATABASE TABLE1 TABLE2 TABLE3 > /path/to/file/dump_table.sql`\
\
Создаём бекап и сразу его архивируем\
`mysqldump -u USER -pPASSWORD DATABASE | gzip > /path/to/outputfile.sql.gz`\
\
Создание бекапа с указанием его даты\
`` mysqldump -u USER -pPASSWORD DATABASE | gzip > `date +/path/to/outputfile.sql.%Y%m%d.%H%M%S.gz` ``\
\
Заливаем бекап в базу данных\
`mysql -u USER -pPASSWORD DATABASE < /path/to/dump.sql`\
\
Заливаем архив бекапа в базу\
`gunzip < /path/to/outputfile.sql.gz | mysql -u USER -pPASSWORD DATABASE`\
или так\
`zcat /path/to/outputfile.sql.gz | mysql -u USER -pPASSWORD DATABASE`\
\
Создаём новую базу данных\
`mysqladmin -u USER -pPASSWORD create NEWDATABASE`\
\
Удобно использовать бекап с дополнительными опциями `-Q -c -e`, т.е.\
`mysqldump -Q -c -e -u USER -pPASSWORD DATABASE > /path/to/file/dump.sql`,
где:\
-   **-Q** оборачивает имена обратными кавычками
-   **-c** делает полную вставку, включая имена колонок
-   **-e** делает расширенную вставку. Итоговый файл получается меньше и
    делается он чуть быстрее

\
\
Для просмотра списка баз данных можно использовать команду:\
`mysqlshow -u USER -pPASSWORD`\
\
А так же можно посмотреть список таблиц базы:\
`mysqlshow -u USER -pPASSWORD DATABASE`\
\
Для таблиц InnoDB надо добавлять **--single-transaction**, это
гарантирует целостность данных бекапа.\
Для таблиц MyISAN это не актуально, ибо они не поддерживают
транзакционность.\
\
[Подробнее](http://www.bonsai.com/wiki/howtos/misc/mysql_admin/)\
\
#### Общие факты

\
-   Полезно под каждую базу на боевом сервере создавать своего
    пользователя
-   Кодировка базы может быть любой, если она UTF8
-   В большинстве случаев лучше использовать движок InnoDB
-   В php лучше забыть про сильно устаревшее расширение mysql и
    по-возможности использовать pdo или mysqli
-   Новую копию MySQL всегда можно настроить и оптимизировать
-   Без особой нужды не стоит открывать MySQL наружу. Вместо этого можно
    сделать проброс портов\
    `ssh -fNL LOCAL_PORT:localhost:3306 REMOTE_USER@REMOTE_HOST`

\
\
#### Работа с данными

\
\
##### Числа

\
-   На 32-битных системах практически нет смысла ставить для типа
    INTEGER свойство UNSIGNED, так как такие большие числа в php не
    поддерживаются.\
    На 64-битных системах, php поддерживает большие числа, вплоть до
    MySQL BIGINT со знаком.
-   Связанные таблицы («Foreign keys») должны иметь полное сходство по
    структуре ключей. Т.е. если у нас на одной таблице для поля указано
    «INTEGER UNSIGNED DEFAULT 0 NOT NULL» то и на другой должно быть
    указано аналогично
-   Для хранения булевых значений, нужно использовать TINYINT(1)
-   А деньги лучше хранить в DECIMAL(10, 2), где первое число обозначает
    количество всех знаков, включая запятую, а второе — количество
    знаков после запятой. Итого, у нас получится что DECIMAL(10,2) может
    сохранить 9999999,99

\
\
##### Строки

\
-   В старых версиях (до 5.0.3) VARCHAR была ограничена 255 символами,
    но сейчас можно указывать до 65535 символов
-   Помните, что тип TEXT ограничен только 64 килобитами, поэтому что бы
    сохранять «Войну и Мир» пользуйтесь «LONGTEXT»
-   Самая правильная кодировка для вашей БД UTF8

\
\
##### Даты

\
Не забывайте, что\
-   DATE, TIME, DATETIME — выводятся в виде строк, поэтому поиск и
    сравнение дат происходит через преобразование
-   TIMESTAMP — хранится в виде UNIX\_TIMESTAMP, и можно указать
    автоматически обновлять колонку
-   Сравнивая типы данных DATETIME и TIMESTAMP, не забывайте делать
    преобразование типов, например:\
    `` SELECT * FROM table WHERE `datetime` = DATE(`timestamp`)  ``

\
\
##### Перечисления

\
-   Для перечислений правильно использовать тип ENUM
-   Правильно пишется так: ENUM('мама', 'мыла', 'раму')
-   Можно ставить значение по-умолчанию, как и для любой строки
-   В базе поле с перечислением хранится как число, поэтому скорость
    работы — потрясающе высокая
-   Количество перечислений \~ 65 тысяч

\
\
[dev.mysql.com/doc/refman/4.1/en/storage-requirements.html](http://dev.mysql.com/doc/refman/4.1/en/storage-requirements.html)\
[help.scibit.com/mascon/masconMySQL\_Field\_Types.html](http://help.scibit.com/mascon/masconMySQL_Field_Types.html)\
\
#### Отладка

\
\
-   Если запросы тормозят, то можно включить лог для медленных запросов
    в /etc/mysql/my.cnf
-   А потом оптимизировать запросы через
    [EXPLAIN](http://habrahabr.ru/blogs/mysql/31129/)
-   И наблюдать за запросами удобно через программу mytop

\
\
Пожалуйста, сообщите мне, если вы заметили неточность или есть желание
поделиться советом или шпаргалкой.
<div class="clear">

</div>

</div>

-   [mysql](http://habrahabr.ru/search/?q=%5Bmysql%5D&target_type=posts)
-   , [tips and
    tricks](http://habrahabr.ru/search/?q=%5Btips%20and%20tricks%5D&target_type=posts)
-   , [советы
    начинающим](http://habrahabr.ru/search/?q=%5B%D1%81%D0%BE%D0%B2%D0%B5%D1%82%D1%8B%20%D0%BD%D0%B0%D1%87%D0%B8%D0%BD%D0%B0%D1%8E%D1%89%D0%B8%D0%BC%5D&target_type=posts)
-   ,
    [шпаргалки](http://habrahabr.ru/search/?q=%5B%D1%88%D0%BF%D0%B0%D1%80%D0%B3%D0%B0%D0%BB%D0%BA%D0%B8%5D&target_type=posts)

<div class="infopanel_wrapper">

<div id="infopanel_post_105954" class="infopanel">

<div class="voting">

<span class="plus" title="Срок голосования истек."></span>
<div class="mark positive">

<span class="score" title="Всего 215: ↑193 и ↓22">+171</span>

</div>

<span class="minus" title="Срок голосования истек."></span>

</div>

<div class="pageviews" title="Просмотры публикации">

177744

</div>

<div class="favorite">

[](http://habrahabr.ru/post/105954/# "Добавить в избранное"){.add}

</div>

<div class="favs_count"
title="Количество пользователей, добавивших публикацию в избранное">

1155

</div>

<div class="author">

[ukko](http://habrahabr.ru/users/ukko/ "Автор текста") <span
class="rating" title="рейтинг пользователя">0,0</span>
[G+](https://plus.google.com/109991236548424964470){.googleplus_profile}

</div>

<div class="share">

<div class="twitter">

[](http://twitter.com/intent/tweet?text=MySQL+%D1%88%D0%BF%D0%B0%D1%80%D0%B3%D0%B0%D0%BB%D0%BA%D0%B8+http://habr.ru/p/105954/+via+%40habrahabr+%23habr "Опубликовать ссылку в Twitter")

</div>

<div class="vkontakte">

[](https://vk.com/share.php?url=http://habrahabr.ru/post/105954/ "Опубликовать ссылку во ВКонтакте")

</div>

<div class="facebook">

[](https://www.facebook.com/sharer/sharer.php?u=http://habrahabr.ru/post/105954/ "Опубликовать ссылку в Facebook")

</div>

<div class="googleplus">

[](https://plus.google.com/share?url=http://habrahabr.ru/post/105954/ "Опубликовать ссылку в Google Plus")

</div>

</div>

</div>

<div class="clear">

</div>

</div>

</div>

</div>
