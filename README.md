<!DOCTYPE html>
<body>
    <h1>МОДУЛЬ 1</h1>
</body>
    <html lang="ru">
    <p>1. Произведите базовую настройку устройств:</p>
    <ul>
      <li>Настройте имена устройств согласно топологии. Используйте полное доменное имя</li>
      <li>На всех устройствах необходимо сконфигурировать IPv4</li>
      <li>IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918</li>
      <li>Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 64 адресов</li>
      <li>Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не более 16 адресов</li>
      <li>Локальная сеть в сторону BR-SRV должна вмещать не более 32 адресов</li>
      <li>Локальная сеть для управления(VLAN999) должна вмещать не более 8 адресов</li>
      <li>Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 3</li>
    </ul>
<details>
    <summary>РЕШЕНИЕ</summary>
        <html lang="ru">
	<h1>Настройка адресации</h1>
		<h2>HQ-RTR, BR-RTR</h2>
    		<p>Настройка имен устройств на ALT Linux:</p>
    		<pre><code>hostnamectl set-hostname ^name^</code></pre>
    		<p>Добавить в файл vim /etc/modules строки:</p>
    		<p><code>ip_gre</code></p>
        	<p><code>ipip</code></p>
    		<p>Включить форвардинг пакетов в файле vim /etc/net/sysctl.conf:</p>
    		<pre><code>net.ipv4.ip_forward = 1</code></pre>
    		<p>IP адресация, туннель, подъинтерфейсы для VLAN: vim /etc/netplan/config.yaml</p>
<p>HQ-RTR</p>
<pre><code>
    network:
        ethernets:
            ens192:
                dhcp4: false
            ens224:
                dhcp4: false
                dhcp6: false
                addresses: [172.16.4.2/28]
                routes:
                    - to: default
                    via: 172.16.4.1
                nameservers:
                    addresses: [8.8.8.8]
                    search: [axample.com]
            ens256:
                dhcp4: false
                dhcp6: false
                addresses: [192.168.55.x/24]
        tunnels:
            ipip30:
                mode: ipip
                local: 172.16.4.2
                remote: 172.16.5.2
                addresses:
                    - 10.10.10.1/30
                ttl: 30
        vlans:
            vlan.100:
                id: 100
                link: ens192
                addresses: [192.168.100.1/26]
            vlan.200:
                id: 200
                link: ens192
                addresses: [192.168.200.1/28]
            vlan.999:
                id: 999
                link: ens192
                addresses: [192.168.99.1/29]
        version: 2
</code></pre>
<p>BR-RTR</p>
<pre><code>
    network:
        ethernets:
            ens192:
                dhcp4: false
                dhcp6: false
                addresses: [192.168.0.1/27]
            ens224:
                dhcp4: no
                dhcp6: no
                addresses: [172.16.5.2/28]
                routes:
                    - to: default
                    via: 172.16.5.1
                nameservers:
                    addresses: [8.8.8.8]
                    search: [axample.com]
            ens256:
                dhcp4: false
                dhcp6: false
                addresses: [192.168.55.x/24]
        tunnels:
            ipip30:
                mode: ipip
                local: 172.16.5.2
                remote: 172.16.4.2
                addresses:
                    - 10.10.10.2/30
                ttl: 30
        version: 2
</code></pre>
    <p>Проверка</p>
		<pre><code>netplan apply</code></pre>
	<p>Перезапуск</p>
		<pre><code>reboot</code></pre>
	<p>Включить firewalld:</p>
		<pre><code>systemctl enable firewalld --now</code></pre>
	<p>Включить динамическую трансляцию адресов:</p>
		<pre><code>firewall-cmd --permanent --zone=external --add-interface=ens224</code></pre>
	<p>Проверить доступность интернета, работоспособность туннеля:</p>
		<pre><code>ping ya.ru</code></pre>
		<pre><code>ping 8.8.8.8</code></pre>
		<pre><code>ping 10.10.10.1</code></pre>
		<pre><code>ping 10.10.10.2</code></pre>
<h1>Настройка адресации</h1>		
<h2>HQ-SRV, BR-SRV</h2>
		<p>Настройка имен устройств на ALT Linux:</p>
		<pre><code>hostnamectl set-hostname ^name^</code></pre>
	<p>IP адресация: vim /etc/netplan/config.yaml</p>
	<p>HQ-SRV</p>
<pre><code>
    network:
        ethernets:
            ens224:
                dhcp4: false
                dhcp6: false
                addresses: [192.168.55.х/24]
            ens192:
                dhcp4: false
                dhcp6: false
			vlans:
            vlan.100:
                id: 100
                link: ens192
                addresses: [192.168.100.2/26]
                routes:
                    - to: default
                    via: 192.168.100.1
                nameservers:
                    addresses: [8.8.8.8]
                    search: [axample.com]
        version: 2
</code></pre>
	<p>BR-SRV</p>
<pre><code>
    network:
        ethernets:
            ens224:
                dhcp4: false
                dhcp6: false
                addresses: [192.168.55.х/24]
            ens192:
                dhcp4: false
                addresses: [192.168.0.2/27]
                routes:
                    - to: default
                    via: 192.168.0.1
                nameservers:
                    addresses: [8.8.8.8]
                    search: [axample.com]
            
        version: 2
</code></pre>
<p>Проверить доступность интернета:</p>
	<pre><code>ping ya.ru</code></pre>
	<pre><code>ping 8.8.8.8</code></pre>
<h1>Настройка OSPF</h1>
<h1>HQ-RTR, BR-RTR</h1>
	<p>Добавить протокол OSPF в firewalld:</p>
		<pre><code>firewall-cmd --permanent --add-protocol=ospf</code></pre>
		<pre><code>firewall-cmd --reload</code></pre>
	<p>Установить пакет FRR</p>
		<pre><code>apt-get install frr -y</code></pre>
	<p>Включить службы ospfd, zebra:</p>
		<pre>ospfd=yes</pre>
		<pre>zebra=yes</pre>
	<p>Заполнить файл:</p>
		<pre><code>vim /etc/frr/frr.conf</code></pre>
<p>HQ-RTR</p>
<pre><code>	
interface ipip30
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 KEY
 ip ospf mtu-ignore
 no ip ospf passive
exit
!
router ospf
 passive-interface default
 network 10.10.10.0/30 area 0
 network 192.168.100.0/26 area 0
 network 192.168.200.0/28 area 0
exit
!
</code></pre>
<p>BR-RTR</p>
<pre><code>	
interface ipip30
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 KEY
 ip ospf mtu-ignore
 no ip ospf passive
exit
!
router ospf
 passive-interface default
 network 10.10.10.0/30 area 0
 network 192.168.0.0/27 area 0
exit
!
</code></pre>
		<p>Поставить службу frr в автозагрузку и включить:</p>
			<pre><code>systemctl enable --now frr</code></pre>
		<p>Проверить таблицу маршрутизации:</p>
			<pre><code>ip route</code></pre>
<h1>Создание локальных учетных записей (вроде можно не делать)</h1>
<h2>HQ-SRV</h2>
	<p>Создать пользователя:</p>
		<pre><code>useradd sshuser -G wheel -u 1010</code></pre>
	<p>Назначить пароль: </p>
		<pre><code>passwd sshuser ^P@ssw0rd^</code></pre>
	<p>Проверить создание пользователя: </p>
		<pre><code>cat /etc/passwd </code></pre>
	<p>Настроить запуск sudo без дополнительной аутентификации:</p>
		<pre><code>vim /etc/sudoers</code></pre>
	<p>Снять комментарий со строки WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL </p>
	<p>Сохранить изменения</p>
		<pre><code>esc + : + wq!</code></pre>
	<p>!!!НА BR/HQ-RTR ДЕЛАЕМ ВСЕ ТОЖЕ САМОЕ!!!</p>
		<pre><code>	useradd net_admin -G wheel</code></pre>
		<pre><code>	passwd net_admin</code></pre>
		<pre><code>	P@$$word</code></pre>
		<pre><code>	vim /etc/sudoers</code></pre>
	<p>Снять комментарий со строки WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL </p>
<h1>Настройка безопасного удаленного доступа</h1>
<h2>HQ-SRV, BR-SRV</h2>
		<p>Открыть конфигурационный файл службы sshd и внести изменения:</p>
			<pre><code>vim /etc/openssh/sshd_config  </code></pre>
		<img src="https://github.com/ssstarovoytovaaa/de2025/blob/main/ssh.png" alt="Описание изображения">
</details>
