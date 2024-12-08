## docker-compose.yml
```yml
version: '3.8'
networks:
  monitor:
    driver: bridge

services:
  redis:
    container_name: redis
    image: redis:6.2
    ports:
      - 6379:6379
    networks:
      - monitor
    restart: always

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    user: root
    restart: always
    ports:
      - 9090:9090
    volumes:
      - ./prometheus/config:/etc/prometheus/
      - ./prometheus/data:/prometheus
    networks:
      - monitor
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: always
    ports:
      - 3001:3000
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning/
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=5736
      - GF_USERS_ALLOW_SIGN_UP=false
    depends_on:
      - prometheus
    networks:
      - monitor

  redis-exporter:
    container_name: redis-exporter
    image: oliver006/redis_exporter:latest
    environment:
      - REDIS_ADDR=redis://redis:6379
    ports:
      - 9121:9121
    depends_on:
      - prometheus
    networks:
      - monitor
    restart: always
```

## prometheus.yml
```yml
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 1m
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'redis-exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['redis-exporter:9121']
```

## Redis 메트릭 수집
<img width="1728" alt="image" src="https://github.com/user-attachments/assets/df92ebf6-b7c5-4f72-a5e8-10204831c843">

## Redis Exporter 작동 방식
1. Redis 연결:
- Redis Exporter는 Redis 서버에 연결하여 INFO, CONFIG, CLIENT LIST 등의 명령어를 실행해 데이터를 가져옵니다.
2. Prometheus에 메트릭 노출:
- 수집된 데이터를 Prometheus가 이해할 수 있는 HTTP 엔드포인트(/metrics)에서 노출합니다.
3. Prometheus가 메트릭 수집:
- Prometheus는 주기적으로 Redis Exporter에서 메트릭을 가져와 저장하고, Grafana와 같은 도구로 시각화하거나 알림을 설정합니다.

