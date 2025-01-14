用户与权限管理
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
    第二组为用户组权限，第三组为其它用户权限，rwx分别表示读、写和执行权限。 **.** 表示文件受SELinux\
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

保留用户与用户组的配置修改
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

用户私有组
^^^^^^^^^^^^^^^^^^^^
在组管理上，系统使用了一种叫用户私有组（user private group，UPG）的配置：即在某个用户\
创建时，会同时创建一个与该用户名同名的组名，这个组内只有该用户。如使用 ``id`` 查看当前登录\
的用户与组信息时，可以看到用户名和组名都是 **root**。

UPG的配置会让权限管理更方便更安全：某个用户创建了一个文件，UPG会设定好一个默认权限（如\
上面的test_file的例子），该用户与组内用户都可以访问修改这个文件，其它用户无法访问修改；另一个用户如果想访问\
修改这个文件，直接把他加入到此用户组中即可。

系统上所有的用户组可通过 ``cat /etc/group`` 查看。

.. note:: 

  系统运行时，有很多系统服务也在后台同时运行，因此通过 ``cat /etc/group`` 查看系统上的用户组\
  时，会有很多用户组，而我们从未创建过它们，这些用户组是在操作系统安装过程中创建的且由系统服务使用。

用户账号及用户组的权限管理
--------------------------------
用户账号分为以下类型:

* 常规用户（Normal user accounts）
  
  可以被创建，修改和删除的常规账号

* 系统用户（System user accounts）

  系统用户在系统上对应着一个系统服务，这类用户只有在软件安装时创建，之后不会进行修改。

  系统用户的UID为1000以下，常规用户的UID从1000开始，不过我们通过 ``/etc/login.defs`` \
  修改为从5000开始，这也是红帽推荐的配置修改。

.. warning:: 

  系统用户一般只配置本机可用，不允许远程登录，否则可能会导致系统服务出问题。

* 组（group）

  组是多个用户集合在一起的单元，用来达到某些特殊目的。比如将一个文件的读写权限赋予\
  给某个组，那这个组内的所有用户都可以对这个文件进行读写。

通过命令行创建用户
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
创建一个test用户并为其设置密码：

* 创建test用户

  .. code:: shell

    useradd test

* 查看创建好的test用户

  .. code:: shell

    id test

  该用户是系统上第一个创建的常规用户，它的uid及gid都应该是5000。

* 为test的用户设置密码

  .. code:: shell

    passwd test

  passwd会提示输入两次密码，这两次密码要保持一致，当提示 ``passwd: all authentication tokens updated successfully.`` ，\
  说明为test用户设置密码的操作已成功。

* 登录新创建的test用户

  可以在vmware中直接登录刚刚创建好的test用户，如果此前已登录了系统，可先执行 ``exit`` 命令退出。

  .. image:: ../images/sysAdmin/2-26.png
    :align: center
  
以上便是创建新用户test的过程，是不是非常简单？

创建用户组
^^^^^^^^^^^^^^^^^^^^^^
创建一个用户组：

* 创建group_test用户组：

  .. code:: shell

    groupadd group_test

* 确认group_test用户组创建结果：

  .. code:: shell

    tail /etc/group

.. image:: ../images/sysAdmin/2-27.png
  :align: center

.. hint:: 

  通过 ``man groupadd`` 了解groupadd的更多信息

  通过 ``man tail`` 了解tail的更多信息

为用户添加附加组
^^^^^^^^^^^^^^^^^^^^^^^^
将用户添加到附加组中可以管理用户的访问权限

* 将test用户添加到group_test用户组中：

  .. code:: shell

    usermod --append -G group_test test

* 确认test用户已加入到group_test组中：

  .. code:: shell

    groups test

.. hint:: 

  通过 ``man usermod`` 和 ``man groups`` 了解它们的作用

创建group_test的文件夹
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
前面我们提到，文件有 ``-rw-r--r--`` 十位权限字符，从第二个开始，每三位为一组，分别分别表示\
用户、用户组、其它用户这三种权限。在UPG系统的配置下，通过设置用户组权限（setgid: set-group identification permission ）\
来实现多用户对某个文件夹的权限控制，将这个文件夹授权给别个用户组，同属于这个用户组的多个用户\
都可以这个访问这个特定的文件夹，文件夹内的文件也将继承这些权限。

接下来我们将实现这个功能

* 创建文件夹

  .. code:: shell

    mkdir /opt/share_dir
    # 查看创建好的文件夹
    ls -dl /opt/share_dir

* 将创建好的group_test组与这个文件夹关联

  .. code:: shell

    chgrp group_test /opt/share_dir/
    # 再次查看创建好的文件夹，比对之前的权限区别
    ls -dl /opt/share_dir

* 增加写（write）权限

  .. code:: shell

    chmod g+rwxs,o-rx /opt/share_dir/
    # 查看权限变化
    ls -dl /opt/share_dir

对比 ``/opt/share_dir`` 最初的权限和最新的权限变化：

.. image:: ../images/sysAdmin/2-28.png
  :align: center

.. hint:: 

  ``#`` 开头的内容为注释说明文字，命令不会执行相关内容。

  mkdir是用来创建文件夹的命令，意为“make directories”。

  ls命令中的d参数是用来列出文件夹的信息，ls默认只列出文件。

  chgrp是改变文件属组的命令，意为“change group ownership”。

  chmod是改变文件权限的命令，参数说明如下:

  * +号用来增加权限，-号用来取消权限
  * u表示user，g表示group，o表示other users，a表示all users
  * r表示read，w表示write，x表示execute（主要用于文件夹的权限，无此权限无法进入文件夹），\
    s也表示execute权限（用于用户或用户组）
  * 权限也可用数字来表示，read（4）write（2）execute（1）来分别表示，如用户权限是读和执行，则权限为5（读和执行相加）。\
    可以一次设置用户、用户组和其它用户三组权限，如上面例子中的 ``chmod g+rwxs,o-rx /opt/share_dir`` 可写作\
    ``chmod 770 /opt/share_dir``，每一个数字代表一组权限。
  
  以上命令可通过 ``man`` 命令来了解学习更多内容

验证share_dir的权限是否生效
""""""""""""""""""""""""""""
作为对比，我们再创建一个用户，这个用户不加入 ``group_test`` 用户组来访问 ``share_dir`` 文件夹。

.. code:: shell

  # 添加用户test1
  useradd test1
  # 为用户test1设置密码
  passwd test1

xshell可以复制当前会话，这样我们就可以有两个窗口来进行测试，双击会话选项页标题（下图红色部分）即可：

.. image:: ../images/sysAdmin/2-29.png
  :align: center


在其中一个窗口执行 ``su - test`` ，另一个窗口执行 ``su - test1`` ，这样就相当于登录了\
这两个测试用户：

.. image:: ../images/sysAdmin/2-30.png
  :align: center

此时，两个用户都执行 ``cd /opt/share_dir`` ，看结果如何：

.. image:: ../images/sysAdmin/2-31.png
  :align: center

.. hint:: 

  ``@`` 前为用户名，后为主机名，因此在执行su命令后， ``@`` 前的用户名变了。
  
  su命令用来以另一个用户的身份执行命令，此处可以用来切换用户，通过 ``man su`` 了解多更信息。

  cd命令用来跳转文件夹，意为“Change directory”。

可以看到 ``test1`` 用户因为不在 ``group_test`` 的用户组中，在访问属于 ``group_test`` 的\
文件夹时，提示 ``Permission denied`` ，也就是没有权限（此处测试x权限）。

两个用户都执行创建文件的命令 ``touch /opt/share_dir/test_file``，看结果如何（此处测试w权限）：

.. image:: ../images/sysAdmin/2-32.png
  :align: center

两个用户都执行 ``cat /opt/share_dir/test_file`` ，看结果如何（此处测试r权限）：

.. image:: ../images/sysAdmin/2-33.png
  :align: center

可以看到，在其它用户权限为 ``---`` 时，test1用户没有任何访问 ``/opt/share_dir`` 的权限，这就是UPG系统的作用，用户可以通过加入特别的用户组中，获得特别的权限。

.. rubric:: 脚注

.. [#f1] 即一个操作系统可同时供多人使用