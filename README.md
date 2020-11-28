# Инструкци
Скачиваем набор утилит для работы с RPM пакетами 


    dnf install -y rpmdevtools


При помощи команды **rpmdev-setuptree** создаем готовую структуру файлов для сборки пакетов. Важно что директороия **rpmbuild** по явится в домашней директории пользователя от которого запущена команда 


    [root@centos8 vagrant]# rpmdev-setuptree
    [root@centos8 ~]# tree rpmbuild/
    rpmbuild/
    |-- BUILD
    |-- RPMS
    |-- SOURCES
    |-- SPECS
    `-- SRPMS


Длее заходим в дирекоторию **SPECS** и создаем там **spec** файл  

    cd /root/rpmbuild/SPECS/
    vim checkip.spec

Содржание **spec** файла:

    Name:       check-ip
    Version:    1
    Release:    1
    Summary:    Most simple RPM package
    License:    FIXME

    %description
    This is my first RPM package, which does nothing.

    %prep
    # we have no source, so nothing here

    %build

    cat > check-ip.py <<EOF
    #!/usr/bin/env python3

    ip, mask = (input('input ip addres : 10.1.1.0/24: ')).split('/')

    oct1, oct2, oct3, oct4 = ip.split('.')
    bin_ip = f'{int(oct1):08b}{int(oct2):08b}{int(oct3):08b}{int(oct4):08b}'
    bin_mask = f'{"1"*int(mask):<032}'
    bin_net = bin_ip[:int(mask) - 32] + '0' * (32 - int(mask))

    print(f'''
    Network:
    {int(bin_net[0:8], 2):<8}  {int(bin_net[8:16], 2):<8}  {int(bin_net[16:24], 2):<8}  {int(bin_net[24:], 2):<8}
    {bin_net[0:8]}  {bin_net[8:16]}  {bin_net[16:24]}  {bin_net[24:]}''')

    print(f'''
    Mask:
    /{mask}
    {int(bin_mask[0:8], 2):<8}  {int(bin_mask[8:16], 2):<8}  {int(bin_mask[16:24], 2):<8}  {int(bin_mask[24:], 2):<8}
    {bin_mask[0:8]}  {bin_mask[8:16]}  {bin_mask[16:24]}  {bin_mask[24:]}''')
    EOF

    %install
    mkdir -p %{buildroot}/usr/bin/
    install -m 755 check-ip.py %{buildroot}/usr/bin/check-ip.py

    %files
    /usr/bin/check-ip.py

    %changelog
    # let's skip this for now


Дальше собраем нашь пакет утилой **rpmbuild**

    rpmbuild -ba checkip.spec


Все готово переходим к созданию нашего репозитория
Сначала устанавливаем утилиту **createrepo** потом создаем директорию для репозитория и переносим туда готавый RPM пакет 


    dnf install -y createrepo
    mkdir /srv/check
    cp /home/vagrant/rpmbuild/RPMS/x86_64/check-ip-1-1.x86_64.rpm /srv/check/


И теперь содаем все мето данный для нашего репозитория командой **createrepo**

    createrepo /srv/check


Все готово тпереь нужно внести информацию о нашем репозитории на машине где мы хотим установить наше приложение.


    cat >> /etc/yum.repos.d/otus.repo << EOF
    [otus]
    name=otus
    baseurl=http://94.180.57.243
    gpgcheck=0
    enabled=1
    EOF


Все готово можно устанавливать наш пакет. 


    dnf install -y chech-ip
