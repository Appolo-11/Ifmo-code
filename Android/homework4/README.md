# Домшнее задание №4 - кэш на SQLite

Цель этого задания -- реализовать кэш на основе базы данных SQLite в существующем приложении, которое показывает расписание поездов РЖД между Санкт-Петербургом и Москвой на любую выбранную дату. В рамках этого задания надо будет создать свой класс на основе [SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) для доступа к БД, реализовать создание и апгрейд таблицы с расписанием, запись и чтение данных.

Приложение состоит из двух экранов:
- Экран выбора маршрута
- Экран расписания

## Экран выбора маршрута

Этот экран показывает станцию отправления и станцию прибытия. Возможность выбора станции отсутсвует -- для демонстрации зафиксирован маршрут Санкт-Петербург - Москва. По клику на кнопку "Расписание V1" или "Расписание V2" происходит переход на экран расписания для выбранного маршрута. Эти две кнопки отличаются версией модели данных, которые должны использоваться в кэше (об этом ниже). 

<img src="https://github.com/IFMO-Android-2016/homework4/blob/master/screenshots/select_route.png" width="360px"/>

Код экрана выбора маршрута написан в классе [SelectRouteActivity](https://github.com/IFMO-Android-2016/homework4/blob/master/app/src/main/java/ru/ifmo/droid2016/rzddemo/SelectRouteActivity.java).

## Экран расписания

При открытии экрана внизу показывается сегодняшняя дата и начинается загрузка расписания для выбранного маршрута и даты, при этом показывается индикатор прогресса. За загрузку данных отвечает класс [TimetableLoader](https://github.com/IFMO-Android-2016/homework4/blob/master/app/src/main/java/ru/ifmo/droid2016/rzddemo/loader/TimetableLoader.java) -- он выполняет запросы к сервису РЖД и получает расписание для выбранных маршрута и даты. После окончания загрузки показывается список поездов или сообщение об ошибке.

<table border="0">
<tr>
<td><img src="https://github.com/IFMO-Android-2016/homework4/blob/master/screenshots/loading_progress.png" width="360px"/></td>
<td><img src="https://github.com/IFMO-Android-2016/homework4/blob/master/screenshots/timetable.png" width="360px"/></td>
</tr>
</table>

В нижней части экрана есть кнопки, позволяющие перейти к следующей или предыдущей дате -- по нажатию на них дата сменяется и загрузка расписания начинается заново для новой выбранной даты.

Код экрана расписания написан в классе [TimetableActivity](https://github.com/IFMO-Android-2016/homework4/blob/master/app/src/main/java/ru/ifmo/droid2016/rzddemo/TimetableActivity.java)

## Кэш расписаний

Кэш расписаний, который предстоит реализовать, позволяет сохранить полученное из сети расписание в базу данных и в следующий раз, когда для экрана расписания понадобится загрузить данные для какого-то маршрута и даты -- прочитать эти данные из базы данны и не выполнять сетевой запрос. Кэш позволяет ускорить загрузку данных и уменьшить количество сетевых запросов (что в свою очередь может сэкономить трафик пользователю и уменьшить нагрузку на сервера).

В качестве ключа для кэша используется комбинация из трех значений: ID станции отправления, ID станции прибытия и дата. В рамках задания мы предполагаем, что расписание между двумя станциями на один день никогда не меняется, поэтому его можно прочитать из кэша и показать пользователю, не выполняя нового запроса. 

В коде приложения уже есть класс [TimetableCache](https://github.com/IFMO-Android-2016/homework4/blob/master/app/src/main/java/ru/ifmo/droid2016/rzddemo/cache/TimetableCache.java), который определяет методы для записи и чтения данных -- их вам предстоит реализовать. Загрузчик расписания `TimetableLoader` уже содержит код, который вызывает методы `TimetableCache` для чтения и записи данных, но так как кэш не реализован, загрузчик ничего не получает из кэша и поэтому всегда выполняет новый запрос. Когда задание будет выполнено, загрузчик автоматически начнет использовать кэш, и при открытии расписания для ранее просмотренной даты, расписание будет сразу загружено из кэша без выполнения сетевого запроса. Для это вам не нужно менять код никаких существующих классов кроме `TimetableCache` (но, возможно, понадобится создать новые дополнительные классы).

## Модель данных

В разработке реальных приложений часто бывают ситуации, когда сначала выпускается первая версия приложения, которая поддерживает некоторую начальную модель данных. Потом в процессе развития приложения модель данных может измениться: какие-то поля добавляются, другие удаляются, появляются новые сущности. При изменении модели данных выпускается новая версия приложения, которая поддерживает все изменения. Но кэш, созданный в первой версии приложения, содержит данные в старом формате, соответсвующем начальной модели данных. Новая версия приложения должна уметь прочитать данные в старом формати и проапгрейдить кэш. В случае базы данных SQLite новая версия приложения может добавить/удалить колонки в таблицах или создать новые таблицы. 

В рамках этого задания нужно поддержать две версии модели данных. В отличие от реальных приложений, где одновременно используется только одна модель данных, в этом демо приложении можно выбрать версию модели данных -- за это и отвечают две кнопки "Расписание V1" и "Расписание V2" на первом экране. Конструктор `TimetableCache` принимает номер версии (1 или 2) в качестве параметра. В зависимости от этого параметра он должен работать с одной или с другой версией модели данных.

Демо приложение реализовано так, что при одном запуске, пока работает процесс приложения, можно выбрать только одну версию. Другую версию можно выбрать, только если остановить процесс приложения и запустить его снова (обычно достаточно выйти по кнопке Back в домашний экран и нажать на иконку приложения снова, но на некоторых устройствах может потребоваться зайти в Менеджер приложений в системных настройках и принудительно остановаить приложение). Это сделано для того, чтобы не возникло ситуации, когда во время работы приложения нужно закрыть БД и открыть ее снова -- так обычно никто не делает, это сложно и чревато багами. При реализации кэша вы можете считать, что во время работы приложения версия модели данных не меняется -- это значит, что `SQLiteOpenHelper` можно создать один раз, хранить его в синглтоне и никогда не закрывать.

### Версия 1

Модель данных представлена классом [TimetableEntry](https://github.com/IFMO-Android-2016/homework4/blob/master/app/src/main/java/ru/ifmo/droid2016/rzddemo/model/TimetableEntry.java), который соответствует одному поезду в расписании, с одним исключением -- поле `trainName` (название поезда, например "Сапсан") не поддерживается. Для этого класса должна быть создана одна таблица с 9 колонками, соответсвующими полям класса. В таблице *не должно* быть колонки для названия поезда (В реальной практике это означало бы, что в первой версии приложения разработчики "забыли" поддержать название поезда).

### Версия 2

Эта версия отличается от первой только тем, что в таблице расписаний добавлена одна колонка для названия поезда. (В реальной практике это означало бы, что разработчики "вспомнили" про название поезда и поддержвали его в новой версии приложения).

### Апгрейд с версии 1 на версию 2

Если при очередном запуске приложения выбрана версия 2 (нажатием на кнопку "Расписание V2"), а до этого уже был создан кэш с версией 1, то происходит апгрейд -- к уже существующей таблице следует добавить одну колонку с названием поезда с дефолтным значением `NULL` для всех строчек таблицы, которые были записаны раньше. Для этого следует использовать SQL выражение `ALTER TABLE`. При этом ранее записанные данные не теряются -- они остаются в кэше и могут быть прочитаны (очевидно, в них не будет названий поездов, даже если оригинальные данные, полученные от сервиса РЖД, содержали их).

### Даунгрейд с версии 2 на версию 1

Если при очередном запуске приложения выбрана версия 1 (нажатием на кнопку "Расписание V1"), а до этого уже был создан кэш с версией 2, то происходит даунгрейд -- из уже существующей таблицы следует удалить одну колонку с названием поезда. SQLite не позволяет удалить одну колонку из таблицы, для этого просто нет соответсвующего SQL выражения. Поэтому следует создать временную таблицу (`CREATE TABLE`) с новой моделью данных, скопировать в нее все данные из старой таблицы без названия поезда (`INSERT` с вложенным `SELECT`) , потом удалить старую таблицу (`DROP TABLE`), а временную переименовать (`ALTER TABLE`) -- чтобы в итоге осталась таблица с тем же именем, но без колонки с названием поезда, содержащая все данные, которые были до даунгрейда.

# Задание

- Реализовать кэш расписания поездов -- класс `TimetableCache`, который должен учитывать параметр конструктора с версией модели данных.
- Кэш должен быть реализован в виде базы данных SQLite с одной таблицей. 
- Для доступа к БД реализовать свой `SQLiteOpenHelper`, который в процессе работы приложения должен создаваться в единственном. экземпляре и использоваться для получения инстанса [SQLiteDatabase](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html).
- Реализовать сценарий создания базы данных с версий 1 и с версией 2.
- Реализовать сценарий апргейда базы данных с версии 1 на версию 2.
- Реализовать сценарий даунгрейда базы данных с версии 2 на версию 1.
- При записи данных в БД использовать явные транзакции и [SQLiteStatement](https://developer.android.com/reference/android/database/sqlite/SQLiteStatement.html) для оптимизации

# Оценка

- За создание БД, чтение и запись данных - 4 балла
- За использование явных транзакций - 1 балл
- За использование `SQLiteStatement` - 1 балл
- За сценарий апгрейда - 1 балл
- За сценарий даунгрейда - 3 балла

# Полезные ссылки

- Документация по работе с БД в Android: https://developer.android.com/training/basics/data-storage/databases.html
- Синтаксис SQL, поддерживаемый SQLite: http://sqlite.org/lang.html