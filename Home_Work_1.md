
1.0. **Развернул ВМ в YC и PG18**
```bash
## Create VM
yc compute instance create \
--name homework-01 \
--zone ru-central1-a \
--create-boot-disk image-folder-id=standard-images,image-family=ubuntu-2204-lts,size=100,auto-delete=true \
--network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
--memory 16G \
--cores 4 \
--hostname homework-01 \
--ssh-key ~/.ssh/id_rsa.pub
```

```bash
## Install PG-18
sudo apt update \
&& sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y \
&& sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
> /etc/apt/sources.list.d/pgdg.list' \
&& wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc \
| sudo apt-key add - && sudo apt-get update \
&& sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-18 unzip atop htop pgtop
```

```bash
## Tuning OS
# wapiness 60-> 1
vim /etc/sysctl.conf
vm.swappiness=1

# transparent_hugepage - отключить
cat /sys/kernel/mm/transparent_hugepage/enabled
#always [madvise] never
echo never > /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/enabled
#always madvise [never]
```
2.0. **Первичный бенчмарк**

2.1. Простой
```bash
sudo -iu postgres pgbench -i postgres
sudo -iu postgres pgbench -P 1 -T 20 postgres
# 572
sudo -iu postgres pgbench -P 1 -c 20 -T 20 postgres
# 614
```

2.2. Расширенный
```bash
sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 572
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 634
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 605
```

3.0. **Настройка оптимальной производительности**
```bash
# Tuning PG https://pgconfigurator.cybertec.at/
vim postgresql.conf

systemctl restart postgresql

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 621
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 613
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 555
```

4.0. **Настройка максимальной производительности не обращая внимания на ACI~~D~~**

```sql
sudo -iu postgres psql -c "ALTER SYSTEM SET synchronous_commit = off;"
sudo -iu postgres psql -c "SELECT pg_reload_conf();"

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 2813
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 2451
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 2000
```

5.0.* **Замедляем в 100500 раз по мотивам статьи [Making Postgres 42,000x slower because I am unemployed](https://byteofdev.com/posts/making-postgres-slow/)**
```bash
su - postgres
vim postgresql.conf

shared_buffers = 8MB
autovacuum_vacuum_insert_threshold = 1
autovacuum_vacuum_threshold = 0
autovacuum_vacuum_scale_factor = 0
autovacuum_vacuum_max_threshold = 1
autovacuum_naptime = 1
vacuum_cost_limit = 10000
vacuum_cost_page_dirty = 0
vacuum_cost_page_hit = 0
vacuum_cost_page_miss = 0
autovacuum_analyze_threshold = 0
autovacuum_analyze_scale_factor = 0
maintenance_work_mem = 128kB
log_autovacuum_min_duration = 0
logging_collector = on
log_destination = stderr
wal_writer_flush_after = 0
wal_writer_delay = 1
min_wal_size = 32MB
max_wal_size = 32MB
checkpoint_timeout = 30
checkpoint_flush_after = 1
wal_sync_method = open_datasync
wal_level = logical
wal_log_hints = on
summarize_wal = on
track_wal_io_timing = on
checkpoint_completion_target = 0
random_page_cost = 10000
cpu_index_tuple_cost = 10000
io_method = worker
io_workers = 1

systemctl restart postgresql

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 52
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 53
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 45
```
