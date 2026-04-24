# Home Automation Guide

## Core Platforms

### Home Assistant
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/homeassistant.sh)"
```

### Zigbee2MQTT
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/zigbee2mqtt.sh)"
```

## Zigbee Devices

| Device | Protocol | Integration |
|--------|----------|------------|
| Aqara sensors | Zigbee | Zigbee2MQTT |
| Philips Hue | Zigbee | Zigbee2MQTT |
| IKEA TRÅDFRI | Zigbee | Zigbee2MQTT |

## Links

- [[HomeLab]]
---

[[index|Back to Proxmox VE]]
