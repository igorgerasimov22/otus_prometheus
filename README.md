# Ansible Playbook для настройки Prometheus и Grafana
Этот Ansible Playbook предназначен для автоматизации процесса установки и настройки мониторинга с использованием **Prometheus** и **Grafana** на целевых хостах, определённых в инвентаре ```otus-prometheus```.

## Содержание
* [Общее описание](#review)
* [Структура Playbook](#structure)
* [Задачи](#tasks)
* [Хендлеры](#handlers)
* Как использовать

<a name="summary"></a> 

## Общее описание
Playbook выполняет следующие ключевые действия:
1. Изменяет имя хоста на ```otus-prom```.
2. Обновляет кэш пакетов.
3. Устанавливает Prometheus и настраивает его конфигурацию.
4. Загружает и устанавливает **Grafana**.
5. Настраивает источники данных и дашборды для Grafana.

<a name="structure"></a> 

## Структура Playbook
Playbook состоит из одной основной части, которая включает задачи и хендлеры:
```yaml
- name: Configuration prometheus and Grafana
  hosts: otus-prometheus
  become: true
```

<a name="tasks"></a> 

## Задачи
Каждая задача имеет своё назначение и выполняется последовательно:\
**1. Изменение имени хоста**
```yaml
- name: Change hostname
  ansible.builtin.hostname:
    name: otus-prom
  notify: Reboot host
```
**2. Обновление кэша пакетов**
```yaml
- name: Update package cache
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600
```
**3. Установка Prometheus**
```yaml
- name: Install prometheus
  ansible.builtin.apt:
    name:
      - prometheus
    state: latest
  notify: prometheus restart
```
**4. Копирование конфигурации Prometheus**
```yaml
- name: Copy prometheus.yml
  ansible.builtin.copy:
    src: ~/ansible/outs_promethes/prometheus.yml
    dest: /etc/prometheus/prometheus.yml
    owner: root
    group: root
    mode: '0644'
    remote_src: no
  notify: prometheus reload Configuration
```
**5. Загрузка и установка Grafana**
```yaml
- name: Download Grafana
  ansible.builtin.get_url:
    url: https://oddler.ru/images/com/soblog/items/2096/grafana-enterprise_10.2.0_amd64.deb
    dest: /tmp/grafana-enterprise_10.2.0_amd64.deb

- name: Install Grafana
  ansible.builtin.apt:
    deb: /tmp/grafana-enterprise_10.2.0_amd64.deb
    force: true
  notify: Restart Grafana
```
**6. Настройка источника данных для Grafana**
```yaml
- name: Provision Grafana datasource
  ansible.builtin.copy:
    src: ~/ansible/outs_promethes/datasource.yaml
    dest: /etc/grafana/provisioning/datasources/datasource.yaml
    owner: root
    group: grafana
    mode: '0640'
    remote_src: no
  notify: Restart Grafana
```
**7. Копирование конфигураций дашбордов для Grafana**
```yaml
- name: Copy grafana dashboards Provision configs
  ansible.builtin.copy:
    src: ~/ansible/outs_promethes/dashboards/sample.yaml
    dest: /etc/grafana/provisioning/dashboards/sample.yaml
    owner: root
    group: grafana
    mode: '0644'
    remote_src: no
  notify: Restart Grafana
```
**8. Создание папки для дашбордов**
```yaml
- name: create folder for dashboards
  ansible.builtin.file:
    path: /var/lib/grafana/dashboards
    state: directory
    owner: grafana
    group: grafana
    mode: '0755'
```
**9. Копирование дашбордов для Grafana**
```yaml
- name: Provision Grafana dashboards
  ansible.builtin.copy:
    src: ~/ansible/outs_promethes/linux-node-exporter.json
    dest: /var/lib/grafana/dashboards/linux-node-exporter.json
    owner: grafana
    group: grafana
    mode: '0644'
    remote_src: no
  notify: Restart Grafana
```

<a name="handlers"></a> 

## Хендлеры
Хендлеры выполняются по уведомлению от задач, когда задача изменяет состояние системы. В этом Playbook определены следующие хендлеры:
**1. Перезагрузка хоста**
```yaml
- name: Reboot host
  ansible.builtin.reboot:
    reboot_timeout: 600
    connect_timeout: 600
```
**2. Перезагрузка Grafana**
```yaml
- name: Restart Grafana
  ansible.builtin.systemd:
    name: grafana-server.service
    state: restarted
    enabled: true
```
**3. Перезагрузка Prometheus**
```yaml
- name: prometheus restart
  ansible.builtin.systemd:
    name: prometheus.service
    state: restarted
    enabled: true
```
**4. Перезагрузка конфигурации Prometheus**
```yaml
- name: prometheus reload Configuration
  ansible.builtin.systemd:
    name: prometheus.service
    state: reloaded
```

## Как использовать
1. Убедитесь, что у вас установлен **Ansible**.
2. Настройте инвентарь, указав хосты, на которых будет выполняться Playbook.
3. Запустите Playbook с помощью команды:
```bash
ansible-playbook путь/к/playbook.yml
```
Где ```путь/к/playbook.yml``` — это путь к вашему Playbook.

## Заключение
Этот Ansible Playbook позволяет быстро и эффективно настроить мониторинг с помощью Prometheus и Grafana на целевых серверах с минимальными усилиями. Вы можете адаптировать его под свои нужды, изменяя конфигурации и добавляя новые задачи.