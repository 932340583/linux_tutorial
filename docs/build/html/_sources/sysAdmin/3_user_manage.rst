用户管理与用户组管理
========================
Linux系统也是一个多用户操作系统 [#f1]_。在多人使用的场景下，就有可能出现权限问题：\
如一个文件只允许某个用户查看，另一个用户不允许查看；系统设置只允许管理员进行操作，\
其它用户无允许操作等等。

用户与用户组
-----------------------------
用户与用户组的管理是Linux系统管理的核心任务之一，Linux系统上有大量不同的用户与\
用户组，而系统内的程序也是以某个用户的身份运行的，所以用户管理相关工作不止止是管理\
“人”的账户。

用户与用户组的概念
^^^^^^^^^^^^^^^^^^^^^^^
我们先在系统上创建一个文件，以此为支点了解Linux系统上的用户与用户组是什么样的：

.. code-block:: shell

    touch test_file
    ls -l

.. image:: ../images/sysAdmin/2-24.png
    :align: center

``touch`` 命令是用来更新文件的创建时间、修改时间等信息的，此处被用来创建一个空的文件。\
``ls -l`` 列出了文件的详细内容（可通过 ``ls --help`` 查询ls命令的所有参数），通过这些\
内容来了解Linux系统的用户和用户组概念。

``ls -l`` 依次列出了以下内容（以test_file为例）：

  * ``-rw-r--r--.``：一共10位字符，第一位表示文件类型， **-** 表示普通文件；后九位每三位为一组，第一组为用户权限，\
    第二组为用户组权限，第三组为其它用户权限，rwx分别表示读写和执行权限。 **.** 表示文件受SELinux\
    控制。
  * ``1 root root   0``：`硬链接`_ 数量、用户名、用户组名及文件大小。
  * ``Dec  4 14:49 test_file``：文件修改时间和文件名。

.. _硬链接: https://baike.baidu.com/item/%E7%A1%AC%E9%93%BE%E6%8E%A5

``ls -l`` 的更多信息可参考 `官方文档`_。

.. _官方文档: https://www.gnu.org/software/coreutils/manual/html_node/What-information-is-listed.html#index-_002dl-7

通过 ``test_file`` 文件创建，可以了解到，Linux的用户权限有三个类别，分别是属主（owner）、属组（group owner）和其它用户（other users）；\
使用权限也有三个，分别是读（r/read）、写（w/write）和执行（x/execute）；还有一个额外的安全控制权限（security context） `SELinux`_。它们共\
同构成了Linux系统权限控制。

.. _SELinux: https://baike.baidu.com/item/SELinux

而用户和用户组都拥有系统中唯一标识的ID，分别是UID（user ID）和GID（group ID）。执行 ``id`` 命令\
可以查看当前用户和用户所在的用户组：

.. image:: ../images/sysAdmin/2-25.png
    :align: center

context是SELinux的标识。

一个组内可以有多个用户，如果一个文件的属组权限是该组，该组内的用户则共享读、写和执行权限。

**以上概念或许苍白无力，一时难以理解，不要紧的，随着对Linux系统的深入，\
这些概念之后会自然理解。**

保留用户与用户组的配置
^^^^^^^^^^^^^^^^^^^^^^^^^^
系统为系统用户和组保留了1000以下的ID，这些保留的ID可以通过下面的命令查看：

.. code-block:: shell

  cat /usr/share/doc/setup/uidgid

因此新用户与用户组的创建在分配ID时，会从1000开始。红帽推荐ID分配从5000开始，\
可以通过修改 ``/etc/login.defs`` 来达成此目的：

1. 使用nano打开这个配置文件：``nano /etc/login.defs``
2. 按键盘中的方向键的↓键，找到UID_MIN的配置：

.. code-block:: linux-config

  # Min/max values for automatic uid selection in useradd(8)
  #
  UID_MIN                  1000

3. 将UID_MIN的值更改为5000

.. code-block:: linux-config

  # Min/max values for automatic uid selection in useradd(8)
  #
  UID_MIN                  5000

3. 继续按↓键打到GID_MIN的配置

.. code-block:: linux-config

  # Min/max values for automatic gid selection in groupadd(8)
  #
  GID_MIN                  1000

4. 将GID_MIN的值更改为5000

.. code-block:: linux-config

  # Min/max values for automatic gid selection in groupadd(8)
  #
  GID_MIN                  5000

5. 按下Ctrl+X键退出编辑，输入Y保存修改，然后按下回车键保存到文件中

之后在新建用户与用户组时，系统会为它们分配5000之后的ID

.. warning:: 

  ``/etc/login.defs`` 中的其它配置项在不理解的情况下不要随意修改，该文件是系统配置\
  文件，随意修改可能会导致系统产生意外问题。

.. rubric:: 脚注

.. [#f1] 即一个操作系统可同时供多人使用