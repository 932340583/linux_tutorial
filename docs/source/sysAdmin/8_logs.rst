通过日志排查问题
==========================================
日志文件记录了系统中发生的各种事件，包括 `kernel`_ 、服务和应用程序的日志。这些信息有助于排查系统\
中发生的问题或监控系统状态。

日志记录是通过内置于系统中的 `syslog`_ 协议来实现的,系统上的应用也可以\
使用它来记录和管理自己的日志，同时也方便了系统审计与排查应用问题。

.. _kernel: https://www.kernel.org/
.. _syslog: https://baike.baidu.com/item/syslog/2802901

syslog日志的后台服务程序
----------------------------------------
syslog日志主要由以下两个服务程序来处理：

- ``systemd-journald`` 服务

  ``systemd-journald`` 是一台后台程序，它从多个源（sources）中收集日志，然后将其转发给 ``Rsyslog`` 以便\
  进行后续处理， ``systemd-journald`` 从以下源中收集日志：

  - ``kernel``
  - 系统启动过程中的早期阶段
  - 各种服务启动和运行时的标准输出与错误输出日志
  - ``syslog``

- ``Rsyslog`` 服务

  ``Rsyslog`` 服务会将 ``syslog`` 的日志按类别和优先级进行分类，然后将它们写入到 ``/var/log`` 目录中，\
  ``/var/log`` 目录就是用来存储日志的地方。
  
syslog日志分类后的日志文件
----------------------------------------
syslog日志经过分类处理后，会被写入到以下文件中：

- ``/var/log/messages`` ：所有的 ``syslog`` 日志被会被写入到这个文件中

  .. image:: ../images/sysAdmin/8_logs/1-3.png
    :align: center

- ``/var/log/secure`` ：安全和身份验证相关的日志，此类日志不会写入到 ``messages`` 文件中

  .. image:: ../images/sysAdmin/8_logs/1-2.png
    :align: center

- ``/var/log/maillog``：邮件服务器相关日志，如 ``postfix`` 的日志就在这里，此类日志不会写入到 ``messages`` 文件中
  
  .. image:: ../images/sysAdmin/8_logs/1-1.png
    :align: center

- ``/var/log/cron``：定时执行任务的相关日志，此类日志不会写入到 ``messages`` 文件中

  .. image:: ../images/sysAdmin/8_logs/1-4.png
    :align: center

.. note:: 

  以上仅列举了部分日志文件，实际上 ``/var/log`` 目录下还有很多其他的日志文件，\
  这取决于系统中安装了哪些服务和应用。

通过命令行查看日志
----------------------------------------
日志功能是 ``systemd`` 的一分部，它可以用来查看和管理日志文件，解决了传统日志记录的相关问题，\
并与系统紧密结合在一起，支持各种日志技术和对于日志分类的访问管理。

使用 ``journalctl`` 命令可以查看各类日志：

- 查看系统日志

  - ``journalctl`` ：查看所有日志
  - ``journalctl FILEPATH`` ：查看与指定文件相关的日志， ``FILEPATH`` 可以是可执行的二进制文件、\
    可执行脚本或设备文件（硬件设备被视为文件，存放于 ``/dev`` 下），输入 ``journalctl /`` 连按 ``tab`` 键\
    可以查看与日志记录相关的文件，如执行 ``journalctl /usr/sbin/NetworkManager`` 可以查看网络管理服务的日志
  - ``journalctl -b`` ：查看系统本次的启动（boot）日志
  - ``journalctl -k -b``：查看系统本次启动的内核（kernel）日志

- 查看服务日志

  - ``journalctl -u SERVICE`` ：查看指定服务的日志， 如 ``journalctl -u postfix.service`` 可以查看 ``postfix`` 服务的日志
  - ``journalctl _SYSTEMD_UNIT=SERVICE _PID=pid`` ：查看指定进程的日志， ``pid`` 可以通过 ``systemctl status SERVICE`` 命令中\
    的 ``Main PID`` 查看，如 ``journalctl _SYSTEMD_UNIT=postfix.service _PID=887`` ：

    .. image:: ../images/sysAdmin/8_logs/1-5.png
      :align: center

  - ``journalctl -u SERVICE -u SERVICE`` ：查看多个服务的日志，如 ``journalctl -u postfix.service -u chronyd.service`` 可以查看 ``postfix`` 和 ``chronyd`` 服务的日志
  - ``journalctl -S TIME -U TIME`` ：查看指定时间段的日志，S表示start，U表示until， ``TIME`` 可以是一个时间，如 ``journalctl -u NetworkManager.service -S "2025-05-07 10:08:00" -U "2025-05-07 11:49:00"`` ，\
    也可以是时间单词："yesterday", "today", "tomorrow"，如 ``journalctl -u NetworkManager.service -S yesterday -U now`` 

- 查看日志的特殊方法

  - ``journalctl -p priority`` ：查看指定级别的日志， ``priority`` 可以是以下级别：

    - ``emerg`` ：系统不可用，发生了严重错误，编号为0
    - ``alert`` ：需要立即采取行动，发生了中断性错误，编号为1
    - ``crit`` ：严重错误，产生了一些问题，编号为2
    - ``err`` ：错误，通常不会影响系统的正常运行，编号为3
    - ``warning`` ：警告，不影响系统的正常运行，编号为4
    - ``notice`` ：通知，提示性消息，编号为5
    - ``info`` ：信息，通常用于记录系统状态，编号为6
    - ``debug`` ：调试，用于调试程序，编号为7

    如查看系统错误类的消息： ``journalctp -p 3``

  - ``journalctl -g KEYWORD`` ：查看包含指定关键字的日志， ``KEYWORD`` 可以是任意字符串，\
    如 ``journalctl -g error`` 可以查看包含 ``error`` 关键字的日志

  - ``journalctl -f`` ：实时查看最新的日志，如需要观察某个服务的日志，可以使用此方法，如 ``journalctl -f -u postfix.service`` 可以实时查看 ``postfix`` 服务的日志
  - ``journalctl -x`` ：为日志添加解释说明文本，解释日志发生时服务的状态，如 ``journalctl -x -u postfix.service``

    .. image:: ../images/sysAdmin/8_logs/1-6.png
      :align: center

  - ``journalctl -e`` ：直接跳转到日志的末尾
  
以上方法是 ``journalctl`` 命令的常用用法，各选项可以组合使用，以满足不同的需求，\
更多细节的用法可以通过 ``man journalctl`` 命令查看。
