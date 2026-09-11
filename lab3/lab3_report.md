University: [ITMO University](https://itmo.ru/ru/)<br>
Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Введение в веб технологии](https://itmo-ict-faculty.github.io/introduction-in-web-tech/)<br>
Year: 2026/2027<br>
Group: U4225<br>
Author: Milkina Maria Igorevna<br>
Lab: Lab3<br>
Date of create: 11.09.2026<br>
Date of finished:

**Лабораторная работа №3**

**Мониторинг с Prometheus и Grafana**

**Цель работы**

Разобраться с базовой настройкой системы мониторинга: запустить Prometheus и Node Exporter для сбора метрик, подключить Prometheus к Grafana и вывести основные показатели на дашборд.

**Ход работы**

**1. Настройка Prometheus**

Сначала в папке lab3 была создана отдельная папка prometheus и файл prometheus.yml.

В конфигурации был задан интервал сбора метрик 15 секунд. Prometheus должен собирать данные с самого себя и с Node Exporter.

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

Чтобы контейнеры могли обращаться друг к другу, была создана отдельная Docker-сеть:

```bash
docker network create monitoring
```

**2. Запуск Node Exporter**

Дальше был запущен Node Exporter. Он нужен для получения системных метрик.

Так как работа выполнялась на macOS через Docker Desktop, использовался вариант запуска без Linux-путей /proc, /sys и /rootfs.

```bash
docker run -d \
  --name node-exporter \
  --network monitoring \
  --restart unless-stopped \
  -p 9100:9100 \
  prom/node-exporter
```

После запуска работа Node Exporter была проверена командой:

```bash
curl http://localhost:9100/metrics
```

В терминале появился большой список метрик, значит сервис успешно работает и отдаёт данные.

![Метрики Node Exporter](images/01_node_exporter_metrics.png)

**3. Запуск Prometheus**

Для хранения данных Prometheus был создан отдельный volume:

```bash
docker volume create prometheus-data
```

После этого был запущен сам Prometheus.

```bash
docker run -d \
  --name prometheus \
  --network monitoring \
  --restart unless-stopped \
  -p 9090:9090 \
  -v prometheus-data:/prometheus \
  -v ~/lab3/prometheus:/etc/prometheus \
  prom/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/prometheus
```

После запуска через docker ps было проверено, что Prometheus и Node Exporter работают одновременно.

![Prometheus и Node Exporter](images/02_prometheus_node_exporter_running.png)

Далее был открыт интерфейс Prometheus в браузере.

```text
http://localhost:9090
```

На странице Target health были видны оба источника данных: Prometheus и Node Exporter. Оба находились в состоянии UP.

![Проверка Targets в Prometheus](images/03_prometheus_targets_up.png)

**4. Запуск Grafana**

Следующим шагом была установлена Grafana.

Для неё также был создан отдельный volume:

```bash
docker volume create grafana-data
```

После этого был запущен контейнер Grafana:

```bash
docker run -d \
  --name grafana \
  --network monitoring \
  --restart unless-stopped \
  -p 3000:3000 \
  -v grafana-data:/var/lib/grafana \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
  grafana/grafana
```

После запуска в системе одновременно работали три контейнера:

- Grafana;
- Prometheus;
- Node Exporter.

![Запущенные контейнеры](images/04_all_monitoring_containers.png)

Grafana была открыта в браузере по адресу:

```text
http://localhost:3000
```

**5. Подключение Prometheus к Grafana**

В Grafana был добавлен новый источник данных Prometheus.

В качестве адреса был указан:

```text
http://prometheus:9090
```

Такой адрес используется потому, что Grafana и Prometheus находятся в одной Docker-сети и Grafana может обращаться к Prometheus по имени контейнера.

После нажатия Save & test соединение успешно установилось.

![Подключение Prometheus к Grafana](images/05_grafana_prometheus_datasource.png)

**6. Создание дашборда**

После подключения источника данных был создан дашборд System Monitoring.

На него были добавлены три графика: CPU, Memory и Disk.

Для CPU использовалась метрика:

```text
node_cpu_seconds_total
```

Для памяти:

```text
node_memory_MemAvailable_bytes
```

Для диска:

```text
node_filesystem_avail_bytes
```

В результате получилось три графика с текущими данными системы.

![Дашборд System Monitoring](images/06_grafana_dashboard.png)

**Результат**

В ходе работы была настроена локальная система мониторинга с помощью Prometheus, Node Exporter и Grafana.

Prometheus успешно получает метрики от Node Exporter, а Grafana использует Prometheus как источник данных. На дашборде отображаются основные показатели системы: CPU, доступная память и свободное место на диске.
