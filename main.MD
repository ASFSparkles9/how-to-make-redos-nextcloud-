# Шаг 1: Настройка SRV2 (Prometheus)

## Установка Prometheus

1. **Установите Prometheus**:
    ```bash
    sudo dnf install golang-github-prometheus
    ```

2. **Запустите и добавьте в автозагрузку**:
    ```bash
    sudo systemctl enable prometheus --now
    ```

3. **Проверьте статус**:
    ```bash
    sudo systemctl status prometheus
    ```

---

## Настройка Prometheus

1. **Откройте конфигурационный файл**:
    ```bash
    sudo nano /etc/prometheus/prometheus.yml
    ```

2. **Измените или дополните секцию `scrape_configs`**:
    ```yaml
    scrape_configs:
      - job_name: 'RTR'
        static_configs:
          - targets: ['172.16.5.1:9100', '172.16.4.1:9100']
      - job_name: 'SRV1'
        static_configs:
          - targets: ['172.16.4.2:9100']
      - job_name: 'SRV2'
        static_configs:
          - targets: ['172.16.5.2:9100']
    ```

3. **Перезапустите службу**:
    ```bash
    sudo systemctl restart prometheus
    ```

---

## Настройка временной зоны

1. **Откройте файл `/etc/prometheus/prometheus.yml`**:
    ```bash
    sudo nano /etc/prometheus/prometheus.yml
    ```

2. **Добавьте или измените параметр**:
    ```yaml
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
      external_labels:
        monitor: 'SRV2'
    ```

---

# Шаг 2: Настройка узлов для мониторинга

## Установка Node Exporter

На каждом узле (RTR, SRV1, SRV2) выполните следующие шаги:

1. **Создайте пользователя**:
    ```bash
    sudo useradd -rs /bin/false nodeusr
    ```

2. **Установите Node Exporter**:
    ```bash
    sudo dnf install prometheus-node_exporter
    ```

3. **Запустите и добавьте в автозагрузку**:
    ```bash
    sudo systemctl enable node_exporter --now
    ```

---

## Проверка доступности

Для проверки откройте в браузере:

- **RTR**: `http://172.16.5.1:9100/metrics` и `http://172.16.4.1:9100/metrics`
- **SRV1**: `http://172.16.4.2:9100/metrics`
- **SRV2**: `http://172.16.5.2:9100/metrics`

---

# Шаг 3: Настройка Grafana на SRV2

## Установка Grafana

1. **Установите Grafana**:
    ```bash
    sudo dnf install grafana
    ```

2. **Запустите и добавьте в автозагрузку**:
    ```bash
    sudo systemctl enable grafana-server --now
    ```

3. **Проверьте статус**:
    ```bash
    sudo systemctl status grafana-server
    ```

---

## Настройка Grafana

Веб-интерфейс доступен по адресу:
```text
http://<IP-адрес_SRV2>:3010
