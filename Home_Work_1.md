
1.0. **Развернул ВМ в YC и PG18**
```bash
# VM
yc compute instance create \
--name homework \
--zone ru-central1-a \
--create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2204-lts,size=100,auto-delete=true \
--network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
--memory 4G \
--cores 4 \
--hostname homework \
--ssh-key ~/.ssh/id_rsa.pub
```

```bash
# PG
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y && \
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
> /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc \
| sudo apt-key add - && sudo apt-get update && \
sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-18 unzip atop htop pgtop
```

2.0. **Первичный бенчмарк**
```bash
```

2.1. Простой
```bash
```

2.2. Расширенный
```bash
```

3.0. **Настройка оптимальной производительности**
```bash
```

4.0. **Настройка максимальной производительности не обращая внимания на ACI~~D~~**

```sql

```
