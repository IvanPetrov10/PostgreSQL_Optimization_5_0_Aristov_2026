
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
echo never > /sys/kernel/mm/transparent_hugepage/enabled
cat /sys/kernel/mm/transparent_hugepage/enabled
```
2.0. **Первичный бенчмарк**

2.1. Простой
```bash
sudo -iu postgres pgbench -i postgres
sudo -iu pgbench -P 1 -T 20 postgres
# 
sudo -iu pgbench -P 1 -c 10 -j 4 -T 20 postgres
# 
```

2.2. Расширенный
```bash
sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 
```

3.0. **Настройка оптимальной производительности**
```bash
# Tuning PG
su - postgres
vim $PGDATA/postgresql.conf

# DB Version: 18
# OS Type: linux
# DB Type: web
# Total Memory (RAM): 16 GB
# CPUs num: 4
# Connections num: 100
# Data Storage: ssd

max_connections = 100
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 40329kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 4
max_parallel_workers_per_gather = 2
max_parallel_workers = 4
max_parallel_maintenance_workers = 2

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 10 postgres
# 
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 10 postgres
# 
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 10 postgres
# 
```

4.0. **Настройка максимальной производительности не обращая внимания на ACI~~D~~**

```sql
sudo -iu postgres psql -c "ALTER SYSTEM SET synchronous_commit = off;"
sudo -iu postgres psql -c "SELECT pg_reload_conf();"

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 
```

5.0. **Замедляем в 100500 разпо мотивам статьи <.....> **
```bash
su - postgres
vim $PGDATA/postgresql.conf

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
log_destination = stderr,jsonlog
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
random_page_cost = 1e300
cpu_index_tuple_cost = 1e300
io_method = worker
io_workers = 1

sudo -iu postgres pgbench -P 1 -c 20 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 40 -j 4 -T 20 postgres
# 
sudo -iu postgres pgbench -P 1 -c 80 -j 4 -T 20 postgres
# 
```
