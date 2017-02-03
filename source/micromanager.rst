.. _micromanager:

********
Описание
********

**Micromanager** - гибкий менеджер процессов, ориентированный на процессы обработки видеопотоков. Является удобным
средством для управления и мониторинга процессами транскодера ffmpeg, сегментера HLS и другими.

Кроме того, micromanager обладает функциями для управления каталогом VOD с функцией инжестирования контента, в т.ч.
транскодирования файлов в необходимый формат.

.. _install-and-using:

*************************
Установка и использование
*************************

Для работы micromanager необходимо наличие установленных библиотек *libjsonrpc*, *libjson*.

Micromanager, а также все сопутствующие пакеты и необходимые библиотеки совместимых версий, поставляются в виде
установочных deb-пакетов и устанавливаются утилитой dpkg. См. :ref:`Где взять <download-software>`.

Для запуска micromanager используется init.d скрипт: ``/etc/init.d/micromanager``: ::

    $ /etc/init.d/micromanager
    Usage: /etc/init.d/micromanager {start|stop|restart|force-reload|reload}

Файлы логов по умолчанию сохраняются в ``/var/log/micromanager/micromanager.log``,
включена ротация логов через logrotate.d

.. _download-software:

Где взять
=========

Для всех
  Скачать необходимые инсталляционные пакеты можно в официальном техническом сообществе Microimpuls
  по ссылке http://forum.micro.im/ в разделе "Дистрибутивы и обновления ПО".

Для инженеров Microimpuls
  При установке ПО на сервер через систему оркестровки все необходимые установочные пакеты
  актуальных версий скачиваются из репозитория автоматически.

.. _configuration:

Конфигурация
============

Файл конфигурации находится в ``/etc/micromanager/micromanager.conf``,
задаётся в формате JSON. Пример: ::

    {
        "log-syslog": false,
        "log-path": "/var/log/micromanager/micromanager.log",
        "log-facility": 0,
        "log-verbose-level": 4,
        "log-foreground": true,

        "log-state-path": "/var/log/micromanager/state.log",
        "log-state-period": 0,

        "json-rpc-enabled": true,
        "json-rpc-listen-host": "0.0.0.0",
        "json-rpc-listen-port": 7089,

        "run-cmd": "%transcoder% -i %src% %ffmpeg-params% -f mpegts udp://@%dsthost%:%dstport%?pkt_size=%payloadsize%",
        "run-cmd-presets": [
            {
                "name": "custom-preset",
                "run-cmd": "/usr/local/bin/ffmpeg -i %src% %ffmpeg-params% -f mpegts udp://@%dsthost%:%dstport%?pkt_size=%payloadsize%"
            }
        ],

        "transcoder-path": "/usr/local/bin/ffmpeg",
        "ffprobe-path": "/usr/local/bin/ffprobe",

        "vod-directory": "/opt/storage/vod",
        "vod-ingest-directory": "/opt/storage/vod_ingest/",
        "vod-ingestion-path-checking-period": 15,
        "vod-ffmpeg-use-progress": true,
        "vod-default-naming-template": "/%preset_name%/%src_path_wo_ext%.mp4",

        "process-memory-limit": 512000,
        "process-cpu-limit": 0,
        "processes-to-watch": [
            { "name": "ffmpeg", "memory-limit": 512000, "cpu-limit": 150 },
        ],
        "low-bitrate-limit": 150000,
        "low-bitrate-checks": 30,
        "fail-max-probing": 5,
        "priority": 0,
        "streams": [
            {
                "name": "Stream 1",
                "enabled": true,
                //"timeout": 0, // seconds
                //"autostart": true,
                //"run-cmd-preset": "",
                //"run-cmd": "",
                //"process-memory-limit": 1024000,
                //"process-cpu-limit": 250,
                "priority": 1,
                "src": [
                    {
                        "name": "ABS-1 stream 1",
                        "enabled": true,
                        "params": [
                            { "name": "src", "value": "udp://@239.1.1.1:1234" },
                            { "name": "ffmpeg-params", "value": "" },
                            // other keys for replacement
                        ]
                    }
                ],
                "dst": "239.0.0.5 1234 multicast udp",
                //"m3u8-playlist": "/tmp/playlist.m3u8",
                "payload-size": 1316
            }
        ]
    }

.. _micromanager-options-description:

Описание параметров micromanager
--------------------------------

.. _micromanager-main-options:

Основные параметры
++++++++++++++++++

log-syslog ``bool``
  Использовать ли службу syslogd для записи логов в /var/log/syslog.
  Не рекомендуется включать при интенсивном логировании.

log-facility ``int``
  Тег в syslog.

log-path ``str``
  Путь до лог-файла для логирования напрямую без syslogd.

log-verbose-level ``int``
  Уровень логирования от 0 до 5, 5 - максимальный DEBUG уровень.

log-foreground ``bool``
  Вывод лога в stdout.

log-state-period ``int``
  Период записи лога состояния в минутах. При значении 0 запись отключается.

log-state-path ``str``
  Путь до файла в который будет записываться лог состояния.

json-rpc-enabled ``bool``
  Включает интерфейс JSON RPC API. Позволяет осуществлять мониторинг и управление процессами.

json-rpc-listen-host ``str``
  Адрес интерфейса для ожидания входящих подключений к JSON RPC API.
  Значение "0.0.0.0" означает слушать на всех интерфейсах.

json-rpc-listen-port ``int``
  Номер порта TCP для JSON RPC API, по умолчанию 7089.

run-cmd ``str``
  Строка запуска процесса транскодирования, например ffmpeg. В строке запуска можно использовать переменные вида
  ``%param%``, вместо этих переменных в момент запуска транскодера будут подставлены соответствующие значения,
  см. :ref:`params <micromanager-params>`.
  Вместо переменной ``%transcoder%`` будет подставлено значение **transcoder-path**.

run-cmd-presets ``list``
  Дополнительные варианты команд запуска транскодера.
  Формат пресета команды описан в :ref:`run-cmd-presets <micromanager-run-cmd-presets>`.
  
run-link-stderr ``bool``
  *С версии 1.4.5*
  
  Включает перехват потока stderr при запуске процесса.
  Если при завершении процесс вернёт код, отличный от 0, то из stderr будет прочитано не более 1500 байт и записано в лог.
  Перехват производится только для процессов инжестирования и дистрибьюции.
  По умолчанию false.

transcoder-path ``str``
  Путь до процесса транскодера, например /usr/local/bin/ffmpeg.

ffprobe-path ``str``
  Путь до ffprobe или avprobe. Используется для определения мета-информации о видео-файле для VOD.

vod-directory ``str``
  Путь до директории VOD, в которой размещаются файлы видеотеки.

vod-ingest-directory ``str``
  Путь до директории инжестирования. В этой директории с заданной периодичностью micromanager проверяет появление
  новых файлов и запускает для них процесс инжестирования - добавления в каталог VOD.

vod-delete-after-autoingestion ``bool``
  *С версии 1.4.3*
  
  Если true, то после успешного автоматического инжестирования исходный файл будет удаляться. По умалчанию false.
  
vod-ingestion-path-checking-period ``int``
  Период проверки директории **vod-ingest-directory**, задаётся в секундах. При значении 0 автоматическая проверка не
  осуществляется, однако возможно запустить инжестирование через API.

vod-ffmpeg-use-progress ``bool``
  Включает экспериментальную возможность использования параметра **-progress** для получения более подробной информации о процессе инжестирования.
  
vod-default-naming-template ``str``
  Шаблон имени файла в каталоге VOD после инжестирования. Доступны переменные:
  
  - ``%preset_name%`` - название пресета из **run-cmd-presets** либо **default**; 
  - ``%src_full_path_wo_ext%`` - путь до исходного файла внутри каталога **vod-ingest-directory** без расширения; 
  - ``%src_full_path%`` - полный путь до исходного файла внутри каталога **vod-ingest-directory**; 
  - ``%src_name%`` - имя исходного файла без расширения; 
  - ``%src_dir%`` - директория, в которой находится исходный файл относительно каталога **vod-ingest-directory**.

vod-distribution-enabled ``bool``
  *С версии 1.4.5*
  
  Включает дистрибьюцию ассета после успешного завершения инжестирования.
  По умолчанию false.
  
vod-default-distribution-template ``str``
  *С версии 1.4.5*
  
  Шаблона процесса дистрибьюции. По умолчанию ``"scp %parameters% %full_path% %address%:%destination%"``.
  Доступны следующие переменные:
  - ``%file_name%`` - имя ассета (без пути);
  - ``%full_path%`` - полный путь ассета;
  - ``%preset_name%`` - имя пресета;
  - ``%address%`` - поле адреса;
  - ``%parameters%`` - дополнительные параметры команды.
  
vod-distribution-addresses ``list``
  *С версии 1.4.5*
  
  Список адресов дистрибьюции.
  Формат описан в :ref:`vod-distribution-addresses <micromanager-vod-distribution-addresses>`.
  
process-memory-limit ``int``
  Лимит потребляемой оперативной памяти в байтах для основного процесса micromanager. По достижению этого лимита
  процессы транскодирования будут перезапущены.

process-cpu-limit ``int``
  Лимит потребляемых ресурсов процессора в процентах для основного процесса micromanager. По достижению этого лимита
  процессы транскодирования будут перезапущены.

processes-to-watch ``list``
  Список процессов, за потреблением CPU и Memory которых будет следить micromanager.
  Формат описан в :ref:`processes-to-watch <micromanager-processes-to-watch>`.

low-bitrate-limit ``int``
  Порог битрейта выходного потока в bps, ниже которого micromanager примет решение о том, что возникла ошибка и перезапустит
  процесс транскодера.

low-bitrate-checks ``int``
  Количество проверок битрейта выходного потока ниже порогового значения, перед тем как принять решение об ошибке.

fail-max-probing ``int``
  Количество проверок отсутствия выходного потока, после которого процесс транскодирования будет перезапущен.

priority ``int``
  Приоритет процесса в ОС, 0 - автоматический приоритет по выбору ОС.

streams ``list``
  Список транскодируемых потоков. Для каждого потока будет запущен инстанс транскодера по команде **run-cmd**, либо,
  если для потока определен **run-cmd-preset**, то команда из соответствующего пресета.
  Формат описан в :ref:`streams <micromanager-streams>`.

score-max-cpu-la1 ``float``
  *С версии 1.4.5*
  
  Максимальное значение средней загрузки вычислительных ресурсов за 1 минуту (CPU LA1).
  
.. _micromanager-run-cmd-presets:

Описание run-cmd-presets
++++++++++++++++++++++++

name ``str``
  Название пресета, имя **default** является зарезервированным и не рекомендуется к использованию.

run-cmd ``str``
  Команда запуска, идентично **run-cmd**.

naming-template ``str``
  Шаблон имени выходного файла, если пресет используется для транскодирования файла при инжестировании.
  Идентично **vod-default-naming-template**. Если данный параметр не задан для пресета, то будет использоваться шаблон по-умолчанию.

distribution-template ``str``
  Шаблона процесса дистрибьюции.
  Аналогично **vod-default-distribution-template**.  Если данный параметр не задан для пресета, то будет использоваться шаблон по-умолчанию.
  
.. _micromanager-processes-to-watch:

Описание processes-to-watch
+++++++++++++++++++++++++++

name ``str``
  Имя процесса, например ffmpeg. Отслеживание происхдит через команду ``ps``.

memory-limit ``int``
  Лимит потребляемой оперативной памяти для процесса, задаётся в байтах. По достижению лимита процесс будет убит.

cpu-limit ``int``
  Лимит потребляемых ресурсов процессора для процесса, задаётся в процентах. По достижению лимита процесс будет убит.

.. _micromanager-streams:

Описание streams
++++++++++++++++

name ``str``
  Название потока.

enabled ``bool``
  Флаг, означающий включен ли процесс транскодирования потока. При значении **false** процесс для данного потока не будет запущен.
  По умолчанию **true**.

autostart ``bool``
  Определяет, необходимо ли автоматически запускать процесс транскодирования, либо запускать только по запросу через API.
  Используется для экономии ресурсов транскодера.
  По умолчанию **true**.

timeout ``int``
  Определяет таймаут в секундах, после которого в случае неактивности (отсутствия запросов на запуск потока через API)
  процесс будет остановлен.
  По умолчанию 0.

run-cmd-preset ``str``
  Имя пресета команды запуска транскодера из списка **run-cmd-presets**. По умолчанию - пустое значение, используется
  команда из **run-cmd**.

run-cmd ``str``
  Переопределяет параметр **run-cmd** для конкретного процесса данного потока. По умолчанию - пустое значение.

process-memory-limit ``int``
  Переопределяет параметр **process-memory-limit** для конкретного процесса данного потока. По умолчанию не определен.

process-cpu-limit ``int``
  Переопределяет параметр **process-cpu-limit** для конкретного процесса данного потока. По умолчанию не определен.

priority ``int``
  Переопределяет параметр **priority** для конкретного процесса данного потока. По умолчанию не определен.

src ``list``
  Список источников для потока. При задании нескольких источников micromanager будет переключать транскодер на следующий
  источник при возникшей проблеме в текущем. Механизм можно использовать для резервирования.
  Формат описан в :ref:`src <micromanager-src>`.

dst ``str``
  Выходной адрес потока. За данным потоком micromanager будет следить, контролируя таким образом работу транскодера.
  Формат **dst** описан в документации `microporter/libmedia <http://mi-microporter-docs.readthedocs.io/en/latest/microporter.html#uri>`_

m3u8-playlist ``str``
  Путь до m3u8-плейлиста, который генерирует сегментер. Если не задан **dst**, но задан этот параметр, то micromanager
  будет осуществлять проверку существования плейлиста и его обновления новыми чанками и на основании этого делать
  вывод статусе потока.

payload-size ``int``
  Размер одного пакета Multicast-потока. По умолчанию 1316.

.. _micromanager-src:

Описание src
++++++++++++

name ``str``
  Название источника.

enabled ``bool``
  Флаг, определяющий, включен ли источник. По умолчанию **true**.

params ``list``
  Список параметров для процесса транскодера. Значения параметров подставляются в строку запуска транскодера в виде
  переменных ``%название_параметра%``.
  Формат описан в :ref:`params <micromanager-params>`.

.. _micromanager-params:

Описание params
+++++++++++++++

name ``str``
  Название параметра.

value ``str``
  Значение параметра.

.. __micromanager-vod-distribution-addresses:
  
Описание vod-distribution-addresses
+++++++++++++++++++++++++++++++++++

enabled ``bool``
  Включает использование данного адреса при дистрибьюции.

address ``str``
  Поле %address% шаблона процесса.
  
parameters ``str``
  Поле %parameters% шаблона процесса.
  
destination ``str``
  Поле %destination% шаблона процесса.
  
.. _middleware-integration:

*************
Лог состояния
*************

Если параметр log-state-period больше нуля, то micromanager с заданной периодичностью в минутах будет вести лог состояния.

Вид записей лога: ::

  Log OK: 28/09 06:24:49
  Mem used: 3790604 KiB, mem free: 267952 KiB
  Swap used: 25728 KiB, swap free: 8359804 KiB
  CPU load: 6.0%
  In VoD directory: 255539 MiB free
  __________________________________________________________________________________________________________________________________
  | NAME                  | ENABLE| SRC                    | DST                            | STATE         | BITRATE  | STOP TIME |
  ----------------------------------------------------------------------------------------------------------------------------------
  | Еврокино              | true  | udp://@239.1.2.3:5000  | 239.120.0.6 1234 multicast udp | NOT_RUNNING   | -        | -         |

  ________________________________________
  | PATH                                  |
  ----------------------------------------
  | Sintel                                |
  | ED.avi                                |
  | failfile                              |

Каждая запись лога состояния содержит две таблицы: таблицу потоков и список файлов в директории ожидания (по умоллчанию /var/vod_ingest).
  
Поля таблицы потоков:
  
- NAME - Имя потока.
- ENABLE - true, если поток включен.
- SRC - Источник потока.
- DST - Исходящий адрес потока.
- STATE - Статус потока.
- BITRATE - Текущий битрейт потока.
- STOP TIME - Время, когда будет остановлен поток. 

***********************
Интеграция с Middleware
***********************

Решение Micromanager интегрировано с системой Microimpuls IPTV/OTT Middleware `Smarty <http://mi-smarty-docs.readthedocs.io/>`_.

.. _troubleshooting:

******************************
Решение проблем и рекомендации
******************************

.. _sysctl.conf:

Ошибки CC error при транскодировании HD и Full HD каналов при высокой нагрузке на CPU
=====================================================================================

Для HD-потоков можно определить более высокий приоритет процесса через параметр **priority** в списке **streams**.

Рекомендуемые параметры ядра
============================

Изменения нужно вносить в файл /etc/sysctl.conf: ::

    kernel.shmmax = 2473822720
    kernel.shmall = 4097152000
    net.core.rmem_default = 8388608
    net.core.rmem_max = 16777216
    net.core.wmem_default = 8388608
    net.core.wmem_max = 16777216
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 0
    net.ipv4.tcp_keepalive_time = 10
    net.ipv4.tcp_fin_timeout = 5

Затем выполнить команду для применения изменений: ::

    sysctl -p

