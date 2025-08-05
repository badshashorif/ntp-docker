## ✅ `README.md` — Dockerized NTP Server Using `cturra/ntp`

```md
# 🕒 Dockerized NTP Server with `cturra/ntp`

This repository contains a production-ready deployment of a secure and lightweight NTP (Network Time Protocol) server using the [cturra/ntp](https://hub.docker.com/r/cturra/ntp) Docker image.

---

## 📦 Features

- Lightweight NTP server using Chrony inside container
- Read-only root filesystem for security
- Custom NTP servers (Asia pool)
- Static IP in a custom Docker bridge network (e.g., `lancache_net`)
- Suitable for internal time sync within a datacenter or network

---

## 📁 Directory Structure

```

.
├── docker-compose.yml
└── README.md

````

---

## 🚀 Deployment

### 1. Create external bridge network (if not already created)

```bash
docker network create \
  --driver=bridge \
  --subnet=172.21.0.0/24 \
  --gateway=172.21.0.1 \
  lancache_net
````

> ✅ Make sure this network matches the IP plan you're using for services like AdGuard or other containers.

---

### 2. docker-compose.yml Configuration

```yaml
version: "3.9"

services:
  ntp:
    image: cturra/ntp
    container_name: ntp
    restart: always
    read_only: true
    tmpfs:
      - /etc/chrony:rw,mode=1750
      - /run/chrony:rw,mode=1750
      - /var/lib/chrony:rw,mode=1750
    environment:
      - NTP_SERVERS=0.asia.pool.ntp.org,1.asia.pool.ntp.org
    ports:
      - "172.16.172.14:123:123/udp"
    networks:
      lancache_net:
        ipv4_address: 172.16.172.14

networks:
  lancache_net:
    external: true
```

---

### 3. Disable conflicting NTP services on host (Important)

```bash
sudo systemctl stop systemd-timesyncd
sudo systemctl disable systemd-timesyncd
```

Or if using `ntpd`:

```bash
sudo systemctl stop ntp
sudo systemctl disable ntp
```

---

### 4. Start the NTP Server

```bash
docker-compose up -d
```

---

### 5. Validate the NTP Server

```bash
ntpdate -q 172.16.172.14
```

Expected Output:

```
server 172.16.172.14, stratum 3, offset 0.000123, delay 0.02567
```

Or use chrony:

```bash
docker exec -it ntp chronyc tracking
docker exec -it ntp chronyc sources
```

---

## 🔧 Troubleshooting

| Problem                                        | Solution                                                                                                    |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `address already in use`                       | Check if host is already using port 123/udp. Stop any NTP services.                                         |
| Container restarts repeatedly                  | Likely due to bad environment variable syntax (e.g. space-separated NTP servers). Use comma-separated list. |
| `no server suitable for synchronization found` | NTP container not reachable or not responding. Verify port and IP.                                          |
| Cannot exec into container                     | May be restarting/crashed. Run `docker logs ntp` to check reason.                                           |

### 🔍 Example of common error and fix

#### Error:

```log
Fatal error : Could not parse server directive at line 7 in file /etc/chrony/chrony.conf
```

#### Fix:

Ensure NTP\_SERVERS is comma-separated:

✅ Correct:

```yaml
NTP_SERVERS=0.asia.pool.ntp.org,1.asia.pool.ntp.org
```

❌ Wrong:

```yaml
NTP_SERVERS=0.asia.pool.ntp.org 1.asia.pool.ntp.org
```

---

## 📜 License

MIT

---

## ✍️ Author

**MD SHORIFUL ISLAM**
Network & System Engineer – [Dot Internet](https://www.dotinternet.com.bd)
GitHub: [shoriful-dot](https://github.com/badshashorif)

---
