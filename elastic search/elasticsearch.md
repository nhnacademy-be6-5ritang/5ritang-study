
# Docker Compose 설정

## docker-compose.yml

```yaml
version: '3.7'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m  # 메모리 설정
      - xpack.security.enabled=true  # X-Pack 보안 활성화
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - esdata:/usr/share/elasticsearch/data  # 데이터 지속성을 위한 볼륨

  kibana:
    image: docker.elastic.co/kibana/kibana:7.14.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:7.14.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
      - ./mysql-connector-java-8.0.25.jar:/usr/share/logstash/mysql-connector-java.jar
      - ./template.json:/usr/share/logstash/template.json
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

volumes:
  esdata:
```

## Docker 명령어

- Docker 실행 명령어:
  ```bash
  docker-compose down
  docker-compose up -d
  ```

- 시스템 메모리 확인:
  ```bash
  free -h
  ```

## Elasticsearch 설정 파일

- **경로**: `/usr/share/elasticsearch/config/elasticsearch.yml`
```yaml
xpack.security.enabled: true
discovery.type: single-node
```

- Docker에서 Elasticsearch 재시작:
  ```bash
  docker restart elasticsearch
  ```

- 비밀번호 자동으로 설정하기:
  ```bash
  docker exec -it elasticsearch /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
  ```

## Kibana 설정 파일

- **경로**: `/usr/share/kibana/config/kibana.yml`
```yaml
elasticsearch.username: "kibana_system"
elasticsearch.password: "생성된_kibana_system_비밀번호"
```

- Docker에서 Kibana 재시작:
  ```bash
  docker restart kibana
  ```

## Kibana 접속

브라우저에서 [http://133.186.134.30:5601/](http://133.186.134.30:5601/) 에 접속하면 로그인 화면이 나타납니다. 'elastic' 사용자와 해당 비밀번호로 로그인할 수 있습니다.

## 비밀번호 목록

- **apm_system**: `xuj2OvH2i2gJODoCWLkI`
- **kibana_system**: `VxEbkQmrtqjFF8narbN4`
- **kibana**: `VxEbkQmrtqjFF8narbN4`
- **logstash_system**: `EV7RgBCzNPnymFX0Jz6p`
- **beats_system**: `Ypos5SXtkAa0bBAdHKNI`
- **remote_monitoring_user**: `Z27nB7a3YlV5wi3Ibar7`
- **elastic**: `mlEkmDZuSVApIYg3a9Qy`

## Elasticsearch 인덱스 확인

- 인덱스 목록 확인:
  ```bash
  curl -u elastic:mlEkmDZuSVApIYg3a9Qy -X GET "localhost:9200/_cat/indices?v"
  ```