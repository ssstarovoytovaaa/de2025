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
    <p>HQ-RTR</p>
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
</details>
