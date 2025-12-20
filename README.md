# adm_training_task24
<h1 align="center">Занятие 24. Настраиваем split-dns</h1>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Описание задания</span></span></h3>
<ol>
<li><span style="font-weight: 300;"> взять стенд </span><a href="https://github.com/erlong15/vagrant-bind"><span style="font-weight: 300;">https://github.com/erlong15/vagrant-bind</span></a></li>
</ol>
<ul>
<li style="font-weight: 300;"><span style="font-weight: 300;">добавить еще один сервер client2</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">завести в зоне dns.lab имена:</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">web1 - смотрит на клиент1</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">web2 - смотрит на клиент2</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">завести еще одну зону newdns.lab</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">завести в ней запись</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">www - смотрит на обоих клиентов</span></li>
</ul>
<ol start="2">
<li><span style="font-weight: 300;"> настроить split-dns</span></li>
</ol>
<ul>
<li style="font-weight: 300;"><span style="font-weight: 300;">клиент1 - видит обе зоны, но в зоне dns.lab только web1</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">клиент2 - видит только dns.lab</span></li>
</ul>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Хостовая ОС Ubuntu 24.04 Desktop с установленным Vagrant 2.3.5. VirtualBox версия 7.0.26 r168464</span></span></p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></span></h3>
<p>******************************</p>
<p>Ввиду невозможности в нынешних реалиях воспользоваться стандартными репозиториями Vagrant (в РФ они сейчас не доступны), предложено обходное решение. Использован репозиторий, развернутый на&nbsp;<a href="https://vagrant.elab.pro/" rel="nofollow">https://vagrant.elab.pro/</a></p>
<p>Так как официальный пакет последней версии Vagrant также не доступен для скачивания, пакет взят оттуда же. Версия 2.3.5.</p>
<img width="411" height="88" alt="image" src="https://github.com/user-attachments/assets/7591c684-f543-4bf6-bfa8-3706a7648c97" />
<p>&nbsp;</p>
<p>Для подключения репозитория надо добавить в Vagrant-файл строку:</p>
<p><code>ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'</code></p>
<p>И изменить box_name и box_version (как в репозитории, если туда зайти).</p>
<img width="246" height="145" alt="image" src="https://github.com/user-attachments/assets/82b82c99-e94c-43af-b112-5acaa47cce17" />
<p>******************************</p>
<p>Создадим каталог ~/task24 и в нем выполним команду:</p>
<p><code>git clone https://github.com/erlong15/vagrant-bind</code></p>
<p>Получим каталог ~/task24/vagrant-bind и перейдем туда:</p>
<img width="813" height="495" alt="image" src="https://github.com/user-attachments/assets/82ed1e9f-db63-4c3a-aedc-8094b78b6f2c" />
<p>&nbsp;</p>
<p>Изменим Vagrantfile в соответствии с заданием (добавить еще один сервер client2) и нашим обходным решением. Измененный файл прикладываю сюда.</p>
<img width="910" height="864" alt="image" src="https://github.com/user-attachments/assets/e0cd8a95-7a35-4401-8a6d-b57ac6c51bf1" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Vagranfile описывает создание 4 виртуальных машин на CentOS 7, каждой машине будет выделено по 256 МБ ОЗУ. В начале файла есть модуль, который отвечает за настройку ВМ с помощью Ansible (запускается плейбук из каталога provisioning). </span></p>
<p>Поскольку мы используем "левый" репозиторий Vagrant со старыми дистрибутивами ОС (см.выше), на этих ВМ необходимо запустить следующие команды, иначе пакеты не будут устанавливаться:</p>
<pre>sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*<br />sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*<br /></pre>
<p>Установка пакетов производится в этом стенде с помощью Ansible, поэтому надо добавить эти команды в плейбук. Также в плейбуке для настройки второго клиента нужно в раздел hosts добавить client2. Не будем устанавливать ntp и просто запустим службу chronyd (внесем в плейбук соответствующие изменения). Измененный плейбук также прикладываю сюда.</p>
<p><span style="font-weight: 300;">В каталоге provisoning находятся также файлы для дополнительной настройки (их использует Ansible-плейбук)</span><span style="font-weight: 300;">:</span></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 300;">client-motd &mdash; файл, содержимое которого будет появляться перед пользователем, который подключился по SSH</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">named.ddns.lab и named.dns.lab &mdash; файлы описания зон ddns.lab и dns.lab соответсвенно</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">master-named.conf и slave-named.conf &mdash; конфигурационные файлы, в которых хранятся настройки DNS-сервера</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">client-resolv.conf и servers-resolv.conf &mdash; файлы, в которых содержатся IP-адреса DNS-серверов</span></li>
</ul>
<p><span style="font-weight: 300;">После внесения изменений, можно попробовать развернуть наши ВМ, для этого нужно воспользоваться командой: </span><code>vagrant up</code>; увидим, что создано 4 ВМ.</p>









