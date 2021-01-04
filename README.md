# SysEventLog

[project]:https://github.com/mazzy-ax/SysEventLog
[license]:https://github.com/mazzy-ax/SysEventLog/blob/master/LICENSE
[ax2009]:ax2009
[ax2012]:ax2012
[ax4]:ax4

[SysEventLog][project] &ndash; это класс-обёртка над [System.Diagnostics.EventLog.WriteEntry](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/sidebar/system-diagnostics-eventlog-writeentry), написанный на X++ в [Microsoft Dynamics AX 2009][ax2009], [Microsoft Dynamics AX 2012][ax2012] и [Axapta 4.0][ax4].

Класс `SysEventLog` позволяет записать из X++ информацию в [Windows Event Log](https://docs.microsoft.com/en-us/windows/win32/wes/windows-event-log).
Класс скрывает преобразования между .net и x++, а также предлагает приемлемые для Аксапты значения по умолчанию.

Класс предлагает программисту 5 публичных статических методов:

* `info`
* `warning`
* `error`
* `write`
* `writeOnServer`

где `info`, `warning`, `error` - это методы-обёртки для повседневной работы с минимумом параметров,
метод `writeOnServer` позволяет гарантировано записать сообщение в Event Log на сервере `AOS`,
а метод `write` &ndash; это рабочая лошадка, которая выполняет основную работу.

## Event Source (Источник)

Для каждой записи в `Windows Event Log` требуется указать строку, которая содержит название источника.
Предполагается, что источник будет связан с названим программы, которая создала данную запись.
Полный список источников можно найти в реестре по адресу `HKLM:SYSTEM\CurrentControlSet\Services\EventLog\Application`.
Создавать новые источники можно только с привилегией локального администратора.

В этом есть определенная проблема для Аксапты - Аксаптовский клиент редко запускается "run as Administrator",
да и Аксаптовский сервер далеко не всегда запускается из-под пользователя с админскими правами.

Поэтому в классе реализовано 3 стратегии для выбора источника:

* прежде всего, программист может указать источник вручную как параметр в методе `write`

* если программист никак не правил класс `SysEventLog` и не указал источник, то класс будет использовать:
  * источник `Dynamics Server ##` для вывода в EventLog на сервере
  * источник `Dynamics Client` для вывода в EventLog на клиенте

  Эти источники создаются во время установки Аксапты и являются достаточно "безопасными".
  Но фильтровать логи и назначать задачи на эти источники достаточно неудобно.

* если программист поправит метод `source` и укажет в коде другую стратегию, то класс будет использовать источники вида
  * `AOS$01-Server`
  * `AOS05$02-WorkingThread`
  * `AOS05$02-Client`
  * `AOS06$01-COMObject`

  Источники специально сделаны похожими на название Windows сервисов, которые реализуют AOS-сервера.
  А правая часть &ndash; это `ClientType`.

* также программист может реализовать свою стратегию для источников.

## Исключение, если источник не зарегистрирован

Если источник не зарегистрирован на компьютере, то класс бросает исключение с длинным сообщением для человека:

> На компьютере %1 не зарегистрирован источник для метода System.Diagnostics.EventLog::WriteEntry (source=%2).
>
> Внимание! источник должен быть создан заранее пользователем с привилегиями администратора.
> Перед первой записью в Windows EventLog должна быть некоторая задержка после создания источника.
>
> Для создания источника на компьютере %1 выполните с привилегиями администратора либо команду PowerShell:
>
> PS> [System.Diagnostics.EventLog]::CreateEventSource("%2", "%3")
>
> либо команду в CMD:
>
> c:\> eventcreate /ID 1 /T INFORMATION /SO "%2" /L "%3" /D log-created

Класс специально жёстко прерывает работу, чтобы незарегистрированные источники были обнаружены как можно раньше.

## Дополнительная информация в тексте сообщения

К началу сообщения класс всегда добавляет информационную строку вида:

> AOS05$02-Client, User:xxx, Company:yyy

## ChangeLog

* [CHANGELOG.md](CHANGELOG.md)
* <https://github.com/mazzy-ax/SysEventLog/releases>

## Помощь проекту

Буду признателен за ваши замечания, предложения и советы по проекту как в разделе [Issues](https://github.com/mazzy-ax/SysEventLog/issues), так и в виде письма на адрес <mazzy@mazzy.ru>

Мазуркин Сергей (mazzy)
