## Installation

### 1. Copy the 'zabbix-agent2-plugin-pacemaker' to Zabbix Agent 2 directory

Copy the downloaded 'zabbix-agent2-plugin-pacemaker' file to the target server.

Install the plugin:

```bash
install -D -o root -g root -m 755 \
    /tmp/zabbix-agent2-plugin-pacemaker \
    /usr/lib/zabbix/agent2/zabbix-agent2-plugin-pacemaker
```

Verify the installation:

```bash
ls -l /usr/lib/zabbix/agent2/zabbix-agent2-plugin-pacemaker
file /usr/lib/zabbix/agent2/zabbix-agent2-plugin-pacemaker
```

### 2. Configure Zabbix Agent 2

Create the plugin configuration directory:

```bash
mkdir -p /etc/zabbix/zabbix_agent2.d/plugins.d
```

Create the Pacemaker configuration file:

```bash
vim /etc/zabbix/zabbix_agent2.d/plugins.d/pacemaker.conf
```

Add the following configuration:

```ini
Plugins.Pacemaker.System.Path=/usr/lib/zabbix/agent2/zabbix-agent2-plugin-pacemaker
```

Make sure the main configuration file `/etc/zabbix/zabbix_agent2.conf` includes the plugin configuration directory:

```ini
Include=/etc/zabbix/zabbix_agent2.d/plugins.d/*.conf
```

### 3. Configure permissions

```bash
visudo -f /etc/sudoers.d/zabbix-pacemaker
```

Add:

```sudoers
zabbix ALL=(root) NOPASSWD: /usr/sbin/crm_mon --one-shot --output-as=xml --inactive --failcounts --operations --timing-details
```

Validate the configuration:

```bash
visudo -cf /etc/sudoers.d/zabbix-pacemaker
```

Ensure the sudoers command and arguments exactly match the invocation used by the plugin.

Test access:

```bash
sudo -u zabbix sudo -n /usr/sbin/crm_mon \
    --one-shot \
    --output-as=xml \
    --inactive \
    --failcounts \
    --operations \
    --timing-details
```

The command should return valid Pacemaker XML.

### 4. Restart Zabbix Agent 2

Restart the service:

```bash
systemctl restart zabbix-agent2
```

Check the service status:

```bash
systemctl status zabbix-agent2
```

Check plugin logs:

```bash
journalctl -u zabbix-agent2 -f
```

### 5. Test the plugin

Test cluster monitoring:

```bash
zabbix_agent2 -t 'pacemaker.ping'
zabbix_agent2 -t 'pacemaker.quorum'
zabbix_agent2 -t 'pacemaker.dc'
zabbix_agent2 -t 'pacemaker.failures'
```

Test node discovery and monitoring:

```bash
zabbix_agent2 -t 'pacemaker.node.discovery'
zabbix_agent2 -t 'pacemaker.node.state[<node_name>]'
zabbix_agent2 -t 'pacemaker.node.is_dc[<node_name>]'
```

Test resource discovery:

```bash
zabbix_agent2 -t 'pacemaker.resource.discovery'
```

Test resource monitoring:

```bash
zabbix_agent2 -t 'pacemaker.resource.state[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.online[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.managed[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.failed[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.node[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.uptime[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.last_change[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.last_change_age[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.last_operation[<resource_name>]'
zabbix_agent2 -t 'pacemaker.resource.last_operation_time[<resource_name>]'
```

If all commands return valid results, the Pacemaker plugin is installed and ready to be monitored through Zabbix Agent 2.

<img width="3228" height="1035" alt="obraz" src="https://github.com/user-attachments/assets/904f8395-7536-45ac-ab9d-ba9cc9f43799" />

