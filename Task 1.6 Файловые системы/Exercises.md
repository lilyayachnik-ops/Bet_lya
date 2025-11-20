# Checkpoint

Необходимо создать каталог `~/test` и в нем файл `test_123` с любым содержимым:

```bash
mkdir ~/test && echo "Never give up" > test_123
```
Проверим содержимое файла `cat ~/test/test_123`:

<img width="644" height="49" alt="image" src="https://github.com/user-attachments/assets/61df8025-6a0e-4319-bb08-82ebfa5d9cb1" />

Дальше, создадим символическую ссылку на каталог `~/test` по такому пути `/tmp/soft_link`, используя команду `ln -s ~/test /tmp/soft_link`:

<img width="748" height="271" alt="image" src="https://github.com/user-attachments/assets/71d76b52-9c53-42ec-83cc-440fe0d1be64" />

Используя ссылку `/tmp/soft_link`, скопируем файл `test_123` в каталог `/tmp` с тем же именем, используя команду `cp /tmp/soft_link/test_123 /tmp`:

<img width="744" height="135" alt="image" src="https://github.com/user-attachments/assets/6f51a5b9-eb26-4f31-80b8-bc1bb8e134aa" />

Создадим жесткую ссылку на файл `/tmp/test_123` с именем `/tmp/hard_link`, используя команду `ln /tmp/test_123 /tmp/hard_link`:

<img width="743" height="131" alt="image" src="https://github.com/user-attachments/assets/642cefc6-eb15-47d0-a097-cec6c6ae4b6f" />

Выведем общее количество `inodes`, используя команду `df -i` для всей системы и `df -i /tmp` для каталога `/tmp`:

<img width="748" height="284" alt="image" src="https://github.com/user-attachments/assets/8e053452-e132-4e49-93fe-e936739f8627" />

Как узнать, что файл является жесткой ссылкой, это можно сделать с помощью команды `ls -i`:

<img width="750" height="61" alt="image" src="https://github.com/user-attachments/assets/b01746f1-2260-4b0f-a4e0-049e3852946b" />

Чтобы подключить к VM новый диск, можно залезть в настройки VM. Выбрать раздел **Носители**. Дальше в **Контроллер: SATA**. Затем в **Добавить в контроллер**, где настраиваете диск, `remote_3.vdi`:

<img width="1111" height="621" alt="image" src="https://github.com/user-attachments/assets/2aca5c67-e274-4223-bcdd-c667b4114873" />

После того, как выполнили действия выше, запустим нашу VM. Проверим в терминале с помощью команды `lsblk` подключенные блочные устройства (устройства хранения данных, которые считывают или записывают данные в блоках определенного рамзера):

<img width="712" height="296" alt="image" src="https://github.com/user-attachments/assets/49c92fe7-12d3-4c99-9d72-04300e3636b3" />

Создадим на нем раздел размером 2 ГБ:

```bash
# Простая утилита для разметки диска в терминале
fdisk /dev/sdd
```

**Важно помнить**:

```bash
fdisk опции устройство

Пару опций:

-B — не стирать первые 512 байт диска, чтобы не повредить загрузочную запись
-l — вывести все разделы на выбранных устройствах или если устройство не выбрано, то на всех
-o — указывает, какие поля данных надо показывать в выводе программы
-u — настраивает формат вывода размера разделов (cylinders, sectors, которое по умолчанию)
-w — режим стирания файловой системы или RAID с диска. Возможные значения: auto, never или always по умолчанию
-v — версия утилиты
-h — показать справку по утилите

Пару команд:

a — включение/выключение флага boot для раздела
d — удалить раздел
F — показать свободное место
l — вывести список известных типов разделов
n — создать новый раздел
p — вывести таблицу разделов
t — изменение типа раздела
w — записать новую таблицу разделов на диск
g — создать пустую таблицу разделов GPT
o — создать пустую таблицу разделоов MBR
q — выйти без сохранения
```

Покажем в терминале:

<img width="747" height="450" alt="image" src="https://github.com/user-attachments/assets/63464bb1-f87f-42c6-b91f-6d0249d52d6b" />
<img width="750" height="59" alt="image" src="https://github.com/user-attachments/assets/c9c802cb-54f6-4528-bbc8-83344e64b3ae" />

Поясним, что за манипуляции мы сделали:

```bash
— n создать новый раздел 
— p тип раздела primary (основной) 
— 1 номер раздела 
— Enter начальный сектор (по умолчанию) 
— +2G размер раздела 
— w записать изменения и выйти 
```

Разметим раздел, как `xfs`, и смонтируем раздел по пути `/mnt`:

```bash
sudo mkfs.xfs /dev/sdd1
sudo mount /dev/sdd1 /mnt
```

Покажем в терминале:

<img width="753" height="174" alt="image" src="https://github.com/user-attachments/assets/3dd12752-e15a-4340-b386-247099ad355e" />
<img width="748" height="23" alt="image" src="https://github.com/user-attachments/assets/7c6f1b4c-e708-4989-8e18-3f329b486ebb" />

Создадим текстовый файл в `/mnt`, используя команду `echo "Never give up" >> /mnt/test_file.txt`:

<img width="749" height="170" alt="image" src="https://github.com/user-attachments/assets/11c4c998-c84e-40e7-a1a2-28549dca3d79" />

Для того, чтобы проверить себя, выполним команду `df -hT, где T — показать тип файловой системы, а h — человеко-читаемый формат`:

<img width="749" height="222" alt="image" src="https://github.com/user-attachments/assets/7a246d4c-146c-4567-9ca3-dcaf8ec5e69c" />

Расширили раздел до 3 ГБ. Мы отмонтировали раздел с помощью команды `sudo unmount /dev/sdd1; sudo fdisk /dev/sdd`. Удалили существующий раздел. И создали новый на 3 ГБ. Проверим:

<img width="753" height="325" alt="image" src="https://github.com/user-attachments/assets/20c74e6b-7baa-4273-9b4f-1c96a2ddd03d" />

Расширим файловую систему до 3 ГБ, используя команду `xfs_grows /mnt`, не забыв после примонтировать его обратно:

<img width="752" height="200" alt="image" src="https://github.com/user-attachments/assets/43e0c072-e3cf-42bc-a10c-e4e268d2223c" />
<img width="748" height="24" alt="image" src="https://github.com/user-attachments/assets/483d69aa-1cd7-4824-939c-f027071d71b7" />

Проверим себя:

<img width="746" height="284" alt="image" src="https://github.com/user-attachments/assets/ba72c12a-3dae-467e-b1ab-6eba67e9e108" />

**Важно**:
```bash
# View the total number of Physical Volumes View the total no of hard disks, partitions, storage size and available size
lsblk
#  View the total number of Physical Volumes
pvdisplay (lsblk)
#  View Volume groups
vgs or lvdisplay
# View Logical volumes
lvs or lvdisplay
# The command for creating a PV from disk “/dev/vda”
pvcreate /dev/vda
# We have 2 physical volumes “/dev/vda” and “/dev/vdb” , now to create a volume group from these physical volumes we can use this command 
vgcreate VolumeGroupName /dev/vda /dev/vdb
# “-n” is used to declare name of Logical Volume, “-L” used to declare the size of LV
lcreate -n NewLogicalVolume -L 20G NewVolumeGroup
```

Добавим два новых диска в настройках VM `remote_1.vdi` и `remore_2.vdi`:

<img width="1113" height="631" alt="image" src="https://github.com/user-attachments/assets/4ddbc453-0683-458f-bb43-8bf6d698db60" />

Посмотрим доступное пространство на всех примонтированных разделах и информацию о них в читаемом виде:

<img width="625" height="200" alt="image" src="https://github.com/user-attachments/assets/26664014-5f8a-4728-abd9-a429801deb0d" />

Выполним команду `pvs`(выводит краткую информацию о физических томах (physical volumes) в системе):

<img width="440" height="54" alt="image" src="https://github.com/user-attachments/assets/3e5d5519-bc52-4a2b-be5b-86ae052a2bed" />

Выполним `lvs` (выводит краткую информацию о логических томах в системе):

<img width="475" height="69" alt="image" src="https://github.com/user-attachments/assets/4aa02cdd-420a-4e70-a298-f7fd9dc8fa1b" />

Выполним `vgs` (выводит краткую информацию о групповых томах в системе):

<img width="466" height="70" alt="image" src="https://github.com/user-attachments/assets/2d97c0ba-bad7-44cb-bc43-76b85bfeda7f" />

После того, как подключили два новых физических устройства, выведим информацию об устройствах `lsblk`:

<img width="634" height="226" alt="image" src="https://github.com/user-attachments/assets/4e6b668f-e1ee-4748-88b2-d167564e7d33" />

Создадим физический том, используя команду:

```bash
sudo pvcreate /dev/sdb /dev/sdc
```

<img width="729" height="88" alt="image" src="https://github.com/user-attachments/assets/d2517ac5-94e9-4726-beb5-8b385d1224dd" />

Чтобы убедиться, что данные диски можно использовать для `LVM`, введём команду `sudo pvdisplay`:

<img width="748" height="487" alt="image" src="https://github.com/user-attachments/assets/39797003-c857-478b-95aa-dbaf24f9b964" />

Далее займемся созданием томов. Создадим группу томов с названием `test` и добавим туда `/dev/sdb`:

```bash
sudo vgcreate test /dev/sdb
sudo vgdisplay
```

<img width="747" height="459" alt="image" src="https://github.com/user-attachments/assets/486acc9d-1922-4675-ac20-74b61742fae9" />

Дальше создадим два логических тома:

```bash
# Вместо -l можно использовать -L. Тогда пишем 5G. -n задает имя логического тома, а test - это имя группы томов
sudo lvcreate -l 50%VG -n lv1 test
sudo lvcreate -l 50%VG -n lv2 test
```

<img width="750" height="42" alt="image" src="https://github.com/user-attachments/assets/f19b4162-2832-4651-83cd-2a04893afd07" />

Проверим:

```bash
sudo lvdisplay
```
<img width="749" height="455" alt="image" src="https://github.com/user-attachments/assets/291b4e13-b471-4717-a0ce-ffbfb8ae1563" />
<img width="748" height="256" alt="image" src="https://github.com/user-attachments/assets/899459d0-3bf4-476b-9880-e1d433d5e488" />

Создадим на обоих томах файловую системы `xfs`:

```bash
sudo mkfs.xfs /dev/test/lv1
sudo mkfs.xfs /dev/test/lv2
```

<img width="747" height="426" alt="image" src="https://github.com/user-attachments/assets/93539f15-cc42-44f5-a16f-68e52a9bc815" />

Создадим две точки монтирования в `/mnt` и смонтируем каждый из томов:

```bash
sudo mkdir /mnt/data_volume1
sudo mount /dev/test/lv1 /mnt/data_volume1

sudo mkdir /mnt/data_volume2
sudo mount /dev/test/lv2 /mnt/data_volume2
```

<img width="750" height="185" alt="image" src="https://github.com/user-attachments/assets/aa92bff6-b83e-4c7c-98ba-63c54f108ba9" />
<img width="744" height="69" alt="image" src="https://github.com/user-attachments/assets/84699f25-3b8b-443f-94e7-a26cfbcca7e1" />

Посмотрим в терминале:

<img width="753" height="243" alt="image" src="https://github.com/user-attachments/assets/420ae0fc-343c-46e0-8c20-2d7e00d2ebfd" />

Добавим второй диск `/dev/sdc` в VG `test`, используя команду

```bash
sudo vgextended test /dev/sdc
```

<img width="756" height="71" alt="image" src="https://github.com/user-attachments/assets/b6a60382-1f9f-4845-971e-4079b561deb6" />

Выведем подробную информацию о группах томов:

```bash
sudo vgdisplay
```

<img width="751" height="465" alt="image" src="https://github.com/user-attachments/assets/e9ccf316-3852-4309-88bd-f0ba1a2c1d7b" />

Выделим все нераспределенное пространство в группе `test`:

```bash
sudo lvextended -l +100%FREE /dev/test/lv1
sudo lvdisplay
```

<img width="749" height="461" alt="image" src="https://github.com/user-attachments/assets/04a62f42-901d-40c1-a83f-342d121f31a5" />

Расширим файловую систему на размер нового доступного пространства:

```bash
sudo xfs_growfs /dev/test/lv1
```

<img width="750" height="195" alt="image" src="https://github.com/user-attachments/assets/d86c3995-e7e6-4a72-a68d-df6d87e34c01" />

Вывод команды `df -h`:

<img width="743" height="254" alt="image" src="https://github.com/user-attachments/assets/6f501446-335a-4de9-b049-57f9c40fe853" />

Вывод команды `pvs`:

<img width="559" height="111" alt="image" src="https://github.com/user-attachments/assets/f55ff837-58e5-492f-8d2e-ce895fe388dd" />

Вывод команды `vgs`:

<img width="580" height="94" alt="image" src="https://github.com/user-attachments/assets/fe872db3-30dc-4a61-966d-c82800a8af9f" />

Вывод команды `lvs`:

<img width="749" height="102" alt="image" src="https://github.com/user-attachments/assets/f4bbc68e-5bd5-423c-8594-62f0e861e634" />

Вывод команды `lsblk`:

<img width="749" height="273" alt="image" src="https://github.com/user-attachments/assets/59a4c565-f31d-4c65-a191-1c4c7b10ffe2" />

**Чуть-чуть теории**:

`LVM` — подсистема операционных систем Linux, позволяющая использовать разные области физического жесткого диска или разных жестких дисков как один логический том. LVM встроена в ядро Linux и реализуется на базе device mapper.

<img width="424" height="575" alt="image" src="https://github.com/user-attachments/assets/1f293d29-4dc1-4310-bca1-f31744468cef" />
