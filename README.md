# Zabbix_agent_setup_WEB3_monitoring
Tutorial for Zabbix agent setup to connect to Elevante WEB3 node monitoring server


Requairments:
  * Ubuntu Server
  * systemd service setup
  
:heavy_exclamation_mark: To connect to Elevante WEB3 nodes monitoring server you need to <put a name here - discord ? / telegram ? > and request support from one of our admins. When granted you can proceed with preparation your node to be connected to monitoring server :heavy_exclamation_mark:

## Instaling Zabbix agent

To install and enable zabbix agent to start after reebot do as follow:
```
apt install zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl restart zabbix-agent
sudo systemctl status zabbix-agent
```

Successfull output should looks like this:
```
<user>@<host>:~$ systemctl status zabbix-agent.service
● zabbix-agent.service - Zabbix Agent
     Loaded: loaded (/lib/systemd/system/zabbix-agent.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-05-31 18:18:20 UTC; 1h 26min ago
    Process: 644 ExecStart=/usr/sbin/zabbix_agentd -c $CONFFILE (code=exited, status=0/SUCCESS)
   Main PID: 678 (zabbix_agentd)
      Tasks: 6 (limit: 4566)
     Memory: 12.7M
     CGroup: /system.slice/zabbix-agent.service
             ├─678 /usr/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
             ├─679 /usr/sbin/zabbix_agentd: collector [idle 1 sec]
             ├─680 /usr/sbin/zabbix_agentd: listener #1 [waiting for connection]
             ├─681 /usr/sbin/zabbix_agentd: listener #2 [waiting for connection]
             ├─682 /usr/sbin/zabbix_agentd: listener #3 [waiting for connection]
             └─683 /usr/sbin/zabbix_agentd: active checks #1 [idle 1 sec]

May 31 18:18:20 vps-d88734b7 systemd[1]: Starting Zabbix Agent...
May 31 18:18:20 vps-d88734b7 systemd[1]: Started Zabbix Agent.
```

## Configuring Zabbix agent

At this point we have clean install of zabbix agent. To make is connecting to monitoring server we need to configure some basic options:

### Encryption

For enrypted connection to server we will setup **PSK enryption** by generating enryption key:
```
sh -c "openssl rand -hex 32 > /etc/zabbix/zabbix_agentd.psk"
```

Output should looks like this:
```
<user>@<host>:~# cat /etc/zabbix/zabbix_agentd.psk
87febe006ce1f97d46031b8398073eb22b8d048a2026342cebdf91add6a8d54e 
```

### Zabbix agent config file

Open **zabbix_agentd.conf** 
```nano /etc/zabbix/zabbix_agentd.conf```

then find and replace following entries to looks as below:
  * `Server=127.0.0.1` to `Server=<elevante_web3_monitoring_server_domain>` /contact Elevante group for this information
  * `TLSConnect=unencrypted` to `TLSConnect=psk`
  * `TLSAccept=unencrypted` to `TLSAccept=psk`
  * `TLSPSKIdentity=<NODE_IP_without_dots>` #for example 18843416870
  * `TLSPSKFile=/etc/zabbix/zabbix_agentd.psk`
after changing all entriex use `Ctrl+X` then `Y` to save and exit **nano**

### Zabbix systemd plugin

Next we need to install zabbix systemd plugin from here [zabbix-systemd-service-monitoring](https://github.com/MogiePete/zabbix-systemd-service-monitoring).

```git clone https://github.com/MogiePete/zabbix-systemd-service-monitoring.git```

alternative go to [zabbix-systemd-service-monitoring](https://github.com/MogiePete/zabbix-systemd-service-monitoring) and download whole repository
**Code (green button) -> Download ZIP** and send to server using "scp" for example.

then make __*.bin__ filex executable and copy them to **/usr/local/bin** directory:
```
cd zabbix-systemd-service-monitoring
chmod +x usr/local/bin/zbx_service*.bin
cp usr/local/bin/zbx_service_* /usr/local/bin/
```

Next step is to add our systemd service to whitelist. This action will make sure that plugin will control exactly what we are running on our server:

```nano service_discovery_whitelist```

add what is crucial for you, in our example we will add hydradx.service to whitelisting:

```sshd|zabbix-agent|hydradx```

after changing all entriex use `Ctrl+X` then `Y` to save and exit **nano**. To apply whitelist services to monitor, set:

```
cp service_discovery_* /etc/zabbix/
cp userparameter_systemd_services.conf /etc/zabbix/zabbix_agentd.d/
systemctl restart zabbix-agent.service
```

That's it, now you should be able to connect to monitoring server. After completing all tasks above, please collect data required to make your server visible in monitoring:
  * Hostname - unique name for your server, example HydraDX <telemetry_ID>
  * Type of server - virtual machine or bare metal
  * IP address of your server
  * PSK Key - from `cat /etc/zabbix/zabbix_agentd.psk`
  * PSK Identity - `TLSPSKIdentity=<NODE_IP_without_dots>` #for example 18843416870


