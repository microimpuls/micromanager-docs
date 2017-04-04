.. _troubleshooting:

*********************************
C. Решение проблем и рекомендации
*********************************

С.1 Ошибки CC error при транскодировании HD и Full HD каналов при высокой нагрузке на CPU
=========================================================================================

Для HD-потоков можно определить более высокий приоритет процесса через параметр **priority** в списке **streams**.

.. _sysctl.conf:

С.2 Рекомендуемые параметры ядра
================================

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

