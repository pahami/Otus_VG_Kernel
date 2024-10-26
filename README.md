 Тема: Vagrant-стенд для обновления ядра и создания образа системы
Задача:
1) Запустить ВМ с помощью Vagrant.
2) Обновить ядро ОС из репозитория ELRepo.
3) Оформить отчет в README-файле в GitHub-репозитории.

Выполнение:

Создаём Vagrantfile

vagrant init

Редактируем Vagrantfile

sudo nano Vagrantfile

Вставляем содержимое

#Описываем Виртуальные машины
MACHINES = {
  # Указываем имя ВМ "kernel update"
  :"kernel-update" => {
              #Какой vm box будем использовать
              :box_name => "generic/centos8s",
              #Указываем box_version
              :box_version => "4.3.4",
              #Указываем количество ядер ВМ
              :cpus => 2,
              #Указываем количество ОЗУ в мегабайтах
              :memory => 1024,
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Отключаем проброс общей папки в ВМ
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Применяем конфигурацию ВМ
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
    end
  end
end

Запускаем виртуальную мащину и подключемся по ssh

vagrant up
vagrant ssh

Обновляем репозиторий (без обновления не подключиться репозиторий elrepo)

sudo yum update

Узнаем версию ядра

uname -r
4.18.0-277.el8.x86_64

Далее подключим репозиторий, откуда возьмём необходимую версию ядра:

sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm

Установим последнее ядро из репозитория elrepo-kernel:

sudo yum --enablerepo elrepo-kernel install kernel-ml -y

Параметр --enablerepo elrepo-kernel указывает что пакет ядра будет запрошен из репозитория elrepo-kernel.

Если требуется, можно назначить новое ядро по-умолчанию вручную:
1) Обновить конфигурацию загрузчика:
	sudo grub2-mkconfig -o /boot/grub2/grub.cfg
2) Выбрать загрузку нового ядра по-умолчанию:
   	sudo grub2-set-default 0
Перезагружаем
	sudo reboot
Проверяем версию ядра
	uname -r
6.11.4-1.el8.elrepo.x86_64


