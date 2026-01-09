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
<p><strong>Заметки по Ansible-playbook:</strong></p>
<p><span style="font-weight: 300;">В начале нашего плейбука в разделе install packages есть модуль yum, можно сразу указывать пакеты после with_items.</span></p>
<p>Поскольку мы используем "левый" репозиторий Vagrant со старыми дистрибутивами ОС (см.выше), на этих ВМ необходимо запустить следующие команды, иначе пакеты не будут устанавливаться:</p>
<pre>sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*<br />sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*<br /></pre>
<p>Поэтому надо добавить эти команды в плейбук в разделе repair yum перед разделом install packages. Не будем устанавливать ntp (уберем установку пакета из раздела  install packages) и просто запустим службу chronyd (внесем в плейбук соответствующие изменения в разделе start chronyd).</p>
<p>В разделе плейбука copy resolv.conf to the client добавим в хосты client2, чтобы на него тоже попали настройки.</p>
<p><span style="font-weight: 300;">В каталоге provisoning находятся также файлы для дополнительной настройки (их использует Ansible-плейбук)</span><span style="font-weight: 300;">:</span></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 300;">client-motd &mdash; файл, содержимое которого будет появляться перед пользователем, который подключился по SSH</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">named.ddns.lab и named.dns.lab &mdash; файлы описания зон ddns.lab и dns.lab соответсвенно</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">master-named.conf и slave-named.conf &mdash; конфигурационные файлы, в которых хранятся настройки DNS-сервера</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">client-resolv.conf и servers-resolv.conf &mdash; файлы, в которых содержатся IP-адреса DNS-серверов</span></li>
</ul>
<p><span style="font-weight: 300;">Нужно подкорректировать файл /etc/resolv.conf для DNS-серверов: на хосте ns01 указать nameserver 192.168.50.10, а на хосте ns02 &mdash; 192.168.50.11</span></p>
<p><span style="font-weight: 300;">В Ansible для этого можно воспользоваться шаблоном с Jinja. Изменим имя файла servers-resolv.conf на servers-resolv.conf.j2 и укажем там следующие условия:</span></p>
<p><em><span style="font-weight: 300;">domain dns</span></em><em><span style="font-weight: 300;">.</span></em><em><span style="font-weight: 300;">lab</span></em></p>
<p><em><span style="font-weight: 300;">search dns</span></em><em><span style="font-weight: 300;">.</span></em><em><span style="font-weight: 300;">lab</span></em></p>
<p><em><span style="font-weight: 300;">#Если имя сервера ns02, то указываем nameserver 192.168.50.11</span></em></p>
<p><em><span style="font-weight: 300;">{%</span></em> <em><span style="font-weight: 300;">if</span></em><em><span style="font-weight: 300;"> ansible_hostname </span></em><em><span style="font-weight: 300;">==</span></em> <em><span style="font-weight: 300;">'ns02'</span></em> <em><span style="font-weight: 300;">%}</span></em></p>
<p><em><span style="font-weight: 300;">nameserver </span></em><em><span style="font-weight: 300;">192.168.50.11</span></em></p>
<p><em><span style="font-weight: 300;">{%</span></em><em><span style="font-weight: 300;"> endif </span></em><em><span style="font-weight: 300;">%}</span></em></p>
<p><em><span style="font-weight: 300;">#Если имя сервера ns01, то указываем nameserver 192.168.50.10</span></em></p>
<p><em><span style="font-weight: 300;">{%</span></em> <em><span style="font-weight: 300;">if</span></em><em><span style="font-weight: 300;"> ansible_hostname </span></em><em><span style="font-weight: 300;">==</span></em> <em><span style="font-weight: 300;">'ns01'</span></em> <em><span style="font-weight: 300;">%}</span></em></p>
<p><em><span style="font-weight: 300;">nameserver </span></em><em><span style="font-weight: 300;">192.168.50.10</span></em></p>
<p><em><span style="font-weight: 300;">{%</span></em><em><span style="font-weight: 300;"> endif </span></em><em><span style="font-weight: 300;">%}</span></em><em><span style="font-weight: 300;">&nbsp;</span></em></p>
<p><span style="font-weight: 300;">&nbsp;</span></p>
<p><span style="font-weight: 300;">После внесения изменений в файл, внесём измения в плейбук: вместо src=servers-resolv.conf добавим src=servers-resolv.conf.j2</span></p>
<p><span style="font-weight: 300;">Измененные файлы и плейбук также прикладываю сюда.</span><span style="font-weight: 300;"></span>
<p><span style="font-weight: 300;">После внесения изменений, можно попробовать развернуть наши ВМ, для этого нужно воспользоваться командой: </span><code>vagrant up</code>; увидим, что создано 4 ВМ.</p>
<img width="1087" height="435" alt="image" src="https://github.com/user-attachments/assets/85227fb2-3153-47fc-9edb-e5ae7dcc6cbb" />
<p>&nbsp;</p>
<p><strong>Добавление имён в зону dns.lab</strong></p>
<p><span style="font-weight: 300;">Проверим, что зона dns.lab уже существует на DNS-серверах:</span></p>
<p><span style="font-weight: 300;">Фрагмент файла </span><em><span style="font-weight: 300;">/etc/named.conf</span></em><span style="font-weight: 300;"> на сервере ns01:</span></p>
<img width="769" height="294" alt="image" src="https://github.com/user-attachments/assets/156925bd-74d2-4223-9927-9a7a1608adf8" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Похожий фрагмент файла </span><em><span style="font-weight: 300;">/etc/named.conf</span></em><span style="font-weight: 300;">&nbsp; находится на slave-сервере ns02:</span></p>
<img width="769" height="302" alt="image" src="https://github.com/user-attachments/assets/2a42dce3-7906-44c3-8b4f-82a6c78bb2c5" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Также на хосте ns01 мы видим файл /etc/named/named.dns.lab с настройкой зоны:</span></p>
<img width="769" height="405" alt="image" src="https://github.com/user-attachments/assets/34797d64-dea9-40f8-b93e-034efaac5a0a" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Именно в этот файл нам потребуется добавить имена. Допишем в конец файла следующие строки:&nbsp;</span></p>
<p><span style="font-weight: 300;">; Web</span></p>
<p><span style="font-weight: 300;">web1&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.15</span></p>
<p><span style="font-weight: 300;">web2&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.16</span></p>
<img width="913" height="517" alt="image" src="https://github.com/user-attachments/assets/cc213b18-6dfb-4011-bcea-5a93d212d2c8" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Для применения настроек нужно:</span></p>
<ul>
<li style="font-weight: 300;"><span style="font-weight: 300;">Перезапустить службу named: </span><em><span style="font-weight: 300;">systemctl restart named</span></em><span style="font-weight: 300;">&nbsp;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 300;">Изменить значение Serial (</span><span style="font-weight: 300;">добавить +1 к числу 2711201407</span><span style="font-weight: 300;">), изменение значения serial укажет slave-серверам на то, что были внесены изменения и что им надо обновить свои файлы с зонами.</span></li>
</ul>
<img width="913" height="540" alt="image" src="https://github.com/user-attachments/assets/c971acd5-111e-4cf4-8f27-8e029d47fbbf" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После внесения изменений, выполним проверку с клиента:</span></p>
<p><span style="font-weight: 300;">[vagrant@client ~]$ dig @192.168.50.10 web1.dns.lab</span></p>
<img width="913" height="693" alt="image" src="https://github.com/user-attachments/assets/c68413dc-c6dd-4cd5-9840-701cbc70e2ba" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">[vagrant@client ~]$ dig @192.168.50.11 web2.dns.lab</span></p>
<img width="1047" height="517" alt="image" src="https://github.com/user-attachments/assets/63588591-7cdb-4709-a6b9-9e75627e0975" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Мы обратились к разным DNS-серверам с разными запросами.</span></p>
<p><strong>Создание новой зоны и добавление в неё записей</strong></p>
<p><span style="font-weight: 300;">Для того, чтобы прописать на DNS-серверах новую зону нам потребуется:&nbsp;</span></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 300;">На хосте </span><em><span style="font-weight: 300;">ns01</span></em><span style="font-weight: 300;"> добавить зону в файл /</span><em><span style="font-weight: 300;">etc/named.conf:</span></em></li>
</ul>
<p><em><span style="font-weight: 300;">// lab's newdns zone</span></em></p>
<p><em><span style="font-weight: 300;">zone "newdns.lab" {</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;type master;</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;allow-transfer { key "zonetransfer.key"; };</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;allow-update { key "zonetransfer.key"; };</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;file "/etc/named/named.newdns.lab";</span></em></p>
<p><em><span style="font-weight: 300;">};</span></em></p>
<ul>
<li style="font-weight: 400;"><em><em><span style="font-weight: 300;">На хосте ns02 также добавить зону и указать с какого сервера запрашивать информацию об этой зоне (фрагмент файла /etc/named.conf):</span></em></em></li>
</ul>
<p><em><span style="font-weight: 300;">// lab's newdns zone</span></em></p>
<p><em><span style="font-weight: 300;">zone "newdns.lab" {</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;type slave;</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;masters { 192.168.50.10; };</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;file "/etc/named/named.newdns.lab";</span></em></p>
<p><em><span style="font-weight: 300;">};</span></em></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 300;">На хосте ns01 создадим файл </span><em><span style="font-weight: 300;">/etc/named/named.newdns.lab</span></em></li>
</ul>
<p><em><span style="font-weight: 300;">nano /etc/named/named.newdns.lab</span></em></p>
<p><em><span style="font-weight: 300;">$TTL 3600</span></em></p>
<p><em><span style="font-weight: 300;">$ORIGIN newdns.lab.</span></em></p>
<p><em><span style="font-weight: 300;">@ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; SOA &nbsp; &nbsp; ns01.dns.lab. root.dns.lab. (</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2711201007 ; serial</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3600 &nbsp; &nbsp; &nbsp; ; refresh (1 hour)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;600&nbsp; &nbsp; &nbsp; &nbsp; ; retry (10 minutes)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;86400&nbsp; &nbsp; &nbsp; ; expire (1 day)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;600&nbsp; &nbsp; &nbsp; &nbsp; ; minimum (10 minutes)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)</span></em></p>
<p>&nbsp;</p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IN&nbsp; &nbsp; &nbsp; NS&nbsp; &nbsp; &nbsp; ns01.dns.lab.</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IN&nbsp; &nbsp; &nbsp; NS&nbsp; &nbsp; &nbsp; ns02.dns.lab.</span></em></p>
<p>&nbsp;</p>
<p><em><span style="font-weight: 300;">; DNS Servers</span></em></p>
<p><em><span style="font-weight: 300;">ns01&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.10</span></em></p>
<p><em><span style="font-weight: 300;">ns02&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.11</span></em></p>
<p>&nbsp;</p>
<p><em><span style="font-weight: 300;">;WWW</span></em></p>
<p><em><span style="font-weight: 300;">www &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.15</span></em></p>
<p><em><span style="font-weight: 300;">www &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.16</span></em></p>
<p><br /><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;В конце этого файла добавим записи www. У файла должны быть права 660, владелец &mdash; root, группа &mdash; named.&nbsp;</span></p>
<img width="650" height="568" alt="image" src="https://github.com/user-attachments/assets/cc3bf0a9-9ee3-4164-8696-aa3078632991" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После внесения данных изменений, изменяем значение serial </span><em><span style="font-weight: 300;">(добавляем +1 к значению 2711201007)</span></em><span style="font-weight: 300;"> и перезапускаем named: </span><em><span style="font-weight: 300;">systemctl restart named</span></em></p>
<img width="650" height="568" alt="image" src="https://github.com/user-attachments/assets/bb80fcae-15f7-4bd2-9c29-3c2be7700d31" />
<p>&nbsp;</p>
<p><strong>Настройка Split-DNS&nbsp;</strong></p>
<p><span style="font-weight: 300;">У нас уже есть прописанные зоны dns.lab и newdns.lab. Однако по заданию client1&nbsp; должен видеть запись web1.dns.lab и не видеть запись web2.dns.lab. Client2 может видеть обе записи из домена dns.lab, но не должен видеть записи домена newdns.lab Осуществить данные настройки нам поможет технология Split-DNS.&nbsp;&nbsp;</span></p>
<p><span style="font-weight: 300;">Для настройки Split-DNS нужно:&nbsp;</span></p>
<p><span style="font-weight: 300;">1) Создать дополнительный файл зоны dns.lab, в котором будет прописана только одна запись: </span><em><span style="font-weight: 300;">nano /etc/named/named.dns.lab.client</span></em></p>
<img width="650" height="568" alt="image" src="https://github.com/user-attachments/assets/402aa39d-9a7d-4463-a5e7-b17ae6ffcc8d" />
<p>&nbsp;</p>
<p><em><span style="font-weight: 300;">У файла должны быть права 660, владелец &mdash; root, группа &mdash; named.&nbsp;&nbsp;</span></em></p>
<p><span style="font-weight: 300;">2) Внести изменения в файл /etc/named.conf на хостах ns01 и ns02</span></p>
<p><span style="font-weight: 300;">Прежде всего нужно сделать access листы для хостов client и client2. Сначала сгенерируем ключи для хостов client и client2, для этого на хосте ns01 запустим утилиту tsig-keygen (ключ может генериться 5 минут и более):&nbsp;</span></p>
<img width="650" height="143" alt="image" src="https://github.com/user-attachments/assets/1f74bf34-bc35-41aa-b0b5-47203a7e1be0" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После генерации, мы увидим ключ (secret) и алгоритм с помощью которого он был сгенерирован. Оба этих параметра нам потребуются в access листе. Всего нам потребуется 2 таких ключа. </span></p>
<img width="650" height="250" alt="image" src="https://github.com/user-attachments/assets/7dc13414-23fa-48ae-8123-ec8c808f369c" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">После их генерации добавим блок с access листами в конец файла /etc/named.conf</span></p>
<img width="881" height="547" alt="image" src="https://github.com/user-attachments/assets/a5986151-b57a-4d96-b81d-81453f65d11e" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">В данном блоке access листов мы выделяем 2 блока:</span></p>
<ul>
<li style="font-weight: 300;"><span style="font-weight: 300;">client имеет адрес 192.168.50.15, использует client-key и не использует client2-key</span></li>
<li style="font-weight: 300;"><span style="font-weight: 300;">client2 имеет адрес 192.168.50.16, использует client2-key и не использует client-key</span></li>
</ul>
<p><span style="font-weight: 300;">Описание ключей и access листов будет одинаковое для master и slave сервера.</span></p>
<p><span style="font-weight: 300;">Далее нужно создать файл с настройками зоны dns.lab для client, для этого на мастер сервере создаём файл </span><em><span style="font-weight: 300;">/etc/named/named.dns.lab.client</span></em><span style="font-weight: 300;"> и добавляем в него следующее содержимое:</span></p>
<p><em><span style="font-weight: 300;">$TTL 3600</span></em></p>
<p><em><span style="font-weight: 300;">$ORIGIN dns.lab.</span></em></p>
<p><em><span style="font-weight: 300;">@ &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; SOA &nbsp; &nbsp; ns01.dns.lab. root.dns.lab. (</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2711201407 ; serial</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3600 &nbsp; &nbsp; &nbsp; ; refresh (1 hour)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;600&nbsp; &nbsp; &nbsp; &nbsp; ; retry (10 minutes)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;86400&nbsp; &nbsp; &nbsp; ; expire (1 day)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;600&nbsp; &nbsp; &nbsp; &nbsp; ; minimum (10 minutes)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IN&nbsp; &nbsp; &nbsp; NS&nbsp; &nbsp; &nbsp; ns01.dns.lab.</span></em></p>
<p><em><span style="font-weight: 300;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IN&nbsp; &nbsp; &nbsp; NS&nbsp; &nbsp; &nbsp; ns02.dns.lab.</span></em></p>
<p><em><span style="font-weight: 300;">; DNS Servers</span></em></p>
<p><em><span style="font-weight: 300;">ns01&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.10</span></em></p>
<p><em><span style="font-weight: 300;">ns02&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.11</span></em></p>
<p><em><span style="font-weight: 300;">;Web</span></em></p>
<p><em><span style="font-weight: 300;">web1&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IN&nbsp; &nbsp; &nbsp; A &nbsp; &nbsp; &nbsp; 192.168.50.15</span></em></p>
<p><span style="font-weight: 300;">Это почти скопированный файл зоны dns.lab, в конце которого </span><span style="font-weight: 300;">удалена строка с записью web2</span><span style="font-weight: 300;">. Имя зоны надо оставить такое же &mdash; dns.lab</span></p>
<p><span style="font-weight: 300;">Теперь можно внести правки в /</span><em><span style="font-weight: 300;">etc</span></em><span style="font-weight: 300;">/named.conf</span></p>
<p><span style="font-weight: 300;">Технология Split-DNS реализуется с помощью описания представлений (view), для каждого отдельного acl. В каждое представление (view) добавляются только те зоны, которые разрешено видеть хостам, адреса которых указаны в access листе.</span></p>
<p><span style="font-weight: 300;">Все ранее описанные зоны должны быть перенесены в модули view. Вне view зон быть недолжно, зона any должна всегда находиться в самом низу.&nbsp;</span></p>
<p><span style="font-weight: 300;">После применения всех вышеуказанных правил на хосте ns01 мы получим следующее содержимое файла /</span><em><span style="font-weight: 300;">etc</span></em><span style="font-weight: 300;">/named.conf</span></p>
<p>options {</p>
<p>// network <br /> listen-on port 53 { 192.168.50.10; };<br /> listen-on-v6 port 53 { ::1; };</p>
<p>// data<br /> directory "/var/named";<br /> dump-file "/var/named/data/cache_dump.db";<br /> statistics-file "/var/named/data/named_stats.txt";<br /> memstatistics-file "/var/named/data/named_mem_stats.txt";</p>
<p>// server<br /> recursion yes;<br /> allow-query { any; };<br /> allow-transfer { any; };<br /> <br /> // dnssec<br /> dnssec-enable yes;<br /> dnssec-validation yes;</p>
<p>// others<br /> bindkeys-file "/etc/named.iscdlv.key";<br /> managed-keys-directory "/var/named/dynamic";<br /> pid-file "/run/named/named.pid";<br /> session-keyfile "/run/named/session.key";<br />};</p>
<p>logging {<br /> channel default_debug {<br /> file "data/named.run";<br /> severity dynamic;<br /> };<br />};</p>
<p>// RNDC Control for client<br />key "rndc-key" {<br /> algorithm hmac-md5;<br /> secret "GrtiE9kz16GK+OKKU/qJvQ==";<br />};<br />controls {<br /> inet 192.168.50.10 allow { 192.168.50.15; 192.168.50.16; } keys { "rndc-key"; }; <br />};</p>
<p># client<br />key "client-key" {<br /> algorithm hmac-sha256;<br /> secret "9frRpmeMw4cA/840hAruk2liZyo6MC03qaaH2xUhAhE=";<br />};<br /># client2<br />key "client2-key" {<br /> algorithm hmac-sha256;<br /> secret "3rq7weyAv96cxYxZcZYb9+Z2dmortQWI+nHFGwvjIrY=";<br />};</p>
<p>// ZONE TRANSFER WITH TSIG<br />include "/etc/named.zonetransfer.key"; <br />server 192.168.50.11 {<br /> keys { "zonetransfer.key"; };<br />};</p>
<p># access-lists<br />acl client { !key client2-key; key client-key; 192.168.50.15; };<br />acl client2 { !key client-key; key client2-key; 192.168.50.16; };</p>
<p>view "client" {<br /> match-clients { client; };</p>
<p>zone "dns.lab" {<br /> type master;<br /> file "/etc/named/named.dns.lab.client";<br /> also-notify { 192.168.50.11 key client-key; };<br /> };</p>
<p>// newdns.lab zone<br /> zone "newdns.lab" {<br /> type master;<br /> file "/etc/named/named.newdns.lab";<br /> also-notify { 192.168.50.11 key client-key; };<br /> };<br />};</p>
<p>view "client2" {<br /> match-clients { client2; };</p>
<p>// dns.lab zone<br /> zone "dns.lab" {<br /> type master;<br /> file "/etc/named/named.dns.lab";<br /> also-notify { 192.168.50.11 key client2-key; };<br /> };</p>
<p>// dns.lab zone reverse<br /> zone "50.168.192.in-addr.arpa" {<br /> type master;<br /> file "/etc/named/named.dns.lab.rev";<br /> also-notify { 192.168.50.11 key client2-key; };<br /> };<br />};</p>
<p>view "default" {<br /> match-clients { any; };</p>
<p>// root zone<br /> zone "." IN {<br /> type hint;<br /> file "named.ca";<br /> };</p>
<p>// zones like localhost<br /> include "/etc/named.rfc1912.zones";<br /> // root DNSKEY<br /> include "/etc/named.root.key";</p>
<p>// dns.lab zone<br /> zone "dns.lab" {<br /> type master;<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.dns.lab";<br /> };</p>
<p>// dns.lab zone reverse<br /> zone "50.168.192.in-addr.arpa" {<br /> type master;<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.dns.lab.rev";<br /> };</p>
<p>// ddns.lab zone<br /> zone "ddns.lab" {<br /> type master;<br /> allow-transfer { key "zonetransfer.key"; };<br /> allow-update { key "zonetransfer.key"; };<br /> file "/etc/named/named.ddns.lab";<br /> };</p>
<p>// newdns.lab zone<br /> zone "newdns.lab" {<br /> type master;<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.newdns.lab";<br /> };<br />};</p>
<p><span style="font-weight: 300;">Далее внесем изменения в файл /</span><em><span style="font-weight: 300;">etc</span></em><span style="font-weight: 300;">/named.conf на сервере ns02. Файл будет похож на файл, лежащий на ns01, только в настройках будет указание забирать информацию с сервера ns01:</span></p>
<p>options {</p>
<p>// network <br /> listen-on port 53 { 192.168.50.11; };<br /> listen-on-v6 port 53 { ::1; };</p>
<p>// data<br /> directory "/var/named";<br /> dump-file "/var/named/data/cache_dump.db";<br /> statistics-file "/var/named/data/named_stats.txt";<br /> memstatistics-file "/var/named/data/named_mem_stats.txt";</p>
<p>// server<br /> recursion yes;<br /> allow-query { any; };<br /> allow-transfer { any; };<br /> <br /> // dnssec<br /> dnssec-enable yes;<br /> dnssec-validation yes;</p>
<p>// others<br /> bindkeys-file "/etc/named.iscdlv.key";<br /> managed-keys-directory "/var/named/dynamic";<br /> pid-file "/run/named/named.pid";<br /> session-keyfile "/run/named/session.key";<br />};</p>
<p>logging {<br /> channel default_debug {<br /> file "data/named.run";<br /> severity dynamic;<br /> };<br />};</p>
<p>// RNDC Control for client<br />key "rndc-key" {<br /> algorithm hmac-md5;<br /> secret "GrtiE9kz16GK+OKKU/qJvQ==";<br />};<br />controls {<br /> inet 192.168.50.11 allow { 192.168.50.15; 192.168.50.16;} keys { "rndc-key"; };<br />};</p>
<p># client<br />key "client-key" {<br /> algorithm hmac-sha256;<br /> secret "9frRpmeMw4cA/840hAruk2liZyo6MC03qaaH2xUhAhE=";<br />};<br /># client2<br />key "client2-key" {<br /> algorithm hmac-sha256;<br /> secret "3rq7weyAv96cxYxZcZYb9+Z2dmortQWI+nHFGwvjIrY=";<br />};</p>
<p>// ZONE TRANSFER WITH TSIG<br />include "/etc/named.zonetransfer.key"; <br />server 192.168.50.10 {<br /> keys { "zonetransfer.key"; };<br />};</p>
<p># access-lists<br />acl client { !key client2-key; key client-key; 192.168.50.15; };<br />acl client2 { !key client-key; key client2-key; 192.168.50.16; };</p>
<p>view "client" {<br /> match-clients { client; };</p>
<p>zone "dns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> file "/etc/named/named.dns.lab";<br />};</p>
<p>// newdns.lab zone<br /> zone "newdns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> file "/etc/named/named.newdns.lab";<br /> };<br />};</p>
<p>view "client2" {<br /> match-clients { client2; };</p>
<p>// dns.lab zone<br /> zone "dns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> file "/etc/named/named.dns.lab";<br /> };</p>
<p>// dns.lab zone reverse<br /> zone "50.168.192.in-addr.arpa" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> file "/etc/named/named.dns.lab.rev";<br /> };<br />};</p>
<p>view "default" {<br /> match-clients { any; };</p>
<p>// root zone<br /> zone "." IN {<br /> type hint;<br /> file "named.ca";<br /> };</p>
<p>// zones like localhost<br /> include "/etc/named.rfc1912.zones";<br /> // root DNSKEY<br /> include "/etc/named.root.key";</p>
<p>// dns.lab zone<br /> zone "dns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.dns.lab";<br /> };</p>
<p>// dns.lab zone reverse<br /> zone "50.168.192.in-addr.arpa" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.dns.lab.rev";<br /> };</p>
<p>// ddns.lab zone<br /> zone "ddns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> allow-transfer { key "zonetransfer.key"; };<br /> allow-update { key "zonetransfer.key"; };<br /> file "/etc/named/named.ddns.lab";<br /> };</p>
<p>// newdns.lab zone<br /> zone "newdns.lab" {<br /> type slave;<br /> masters { 192.168.50.10; };<br /> allow-transfer { key "zonetransfer.key"; };<br /> file "/etc/named/named.newdns.lab";<br /> };<br />};</p>
<p><span style="font-weight: 300;">После внесения данных изменений можно перезапустить (по очереди) службу named на серверах ns01 и ns02.</span></p>
<p><span style="font-weight: 300;">Далее, нужно будет проверить работу Split-DNS с хостов client и client2. Для проверки можно использовать утилиту ping</span></p>
<p><span style="font-weight: 300;">Проверка на client:</span></p>
<img width="881" height="473" alt="image" src="https://github.com/user-attachments/assets/015c0889-75e2-4009-b54e-7f42a7053a39" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">На хосте мы видим, что client видит обе зоны (dns.lab и newdns.lab), однако информацию о хосте web2.dns.lab он получить не может.</span><span style="font-weight: 300;">&nbsp;</span></p>
<p><span style="font-weight: 300;">Проверка на client2:&nbsp;</span></p>
<img width="881" height="473" alt="image" src="https://github.com/user-attachments/assets/d95c5d32-1d4e-42db-b9f3-8cb8f7beab1b" />
<p>&nbsp;</p>
<p><span style="font-weight: 300;">Тут мы понимаем, что client2 видит всю зону dns.lab и не видит зону newdns.lab</span></p>
<p><span style="font-weight: 300;">Split-DNS работает. Задание завершено.</span></p>


