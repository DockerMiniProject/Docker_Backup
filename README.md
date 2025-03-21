# 🪄Docker_Backup
Docker를 활용하여 Spring Boot 애플리케이션과 MySQL 데이터베이스를 컨테이너화하고, MySQL 자동 백업 및 복원 기능을 구현함

### ✅ 실습 목표

1. `docker-compose`를 이용한 Spring Boot & MySQL 컨테이너 실행
2. MySQL 데이터 자동 백업 및 복원 스크립트 구현
3. 컨테이너 기반으로 운영 및 배포 자동화 학습

---

## 프로젝트 디렉토리 아키텍처
```
999.test/
├── Dockerfile
├── docker-compose.yml
├── restore.sh
├── mysql-backup/
├──
│   ├── backup.sh
│   ├── Dockerfile.mysql-backup
│   └──  crontab
└── backups/  (→ 자동으로 생성될 볼륨 마운트 위치)
```

### 🗂️999.test
---
1. Dockerfile

![image](https://github.com/user-attachments/assets/4f362b52-eed4-452e-9e64-d0d908875394)

2. docker-compose.yml

```yaml
services:
  db:
    container_name: mysqldb
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: fisa
      MYSQL_USER: user01
      MYSQL_PASSWORD: user01
    networks:
      - spring-mysql-net
    healthcheck:
      test: ['CMD-SHELL', 'mysqladmin ping -h 127.0.0.1 -u root --password=$${MYSQL_ROOT_PASSWORD} || exit 1']
      interval: 10s
      timeout: 2s
      retries: 100
  app:
    container_name: springbootapp
    image: my-springboot-app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      MYSQL_HOST: db
      MYSQL_PORT: 3306
      MYSQL_DATABASE: fisa
      MYSQL_USER: user01
      MYSQL_PASSWORD: user01
    depends_on:
      db:
        condition: service_healthy
    networks:
      - spring-mysql-net
  app2:
    container_name: springbootapp2
    image: my-springboot-app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8088:8080"
    environment:
      MYSQL_HOST: db
      MYSQL_PORT: 3306
      MYSQL_DATABASE: fisa
      MYSQL_USER: user01
      MYSQL_PASSWORD: user01
    depends_on:
      db:
        condition: service_healthy
    networks:
      - spring-mysql-net
  mysql-backup:
    build:
      context: ./mysql-backup
      dockerfile: Dockerfile.mysql-backup
    container_name: mysql-backup
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./backups:/backups  # 호스트와 컨테이너 간 백업 공유
    environment:
      MYSQL_ROOT_PASSWORD: root
    networks:
      - spring-mysql-net
networks:
  spring-mysql-net:
    driver: bridge
volumes:
  db_backups:
```



4. run-all.sh
   
![image](https://github.com/user-attachments/assets/405b4ca0-a6e1-424e-b897-9ab7ae166603)

### 📁mysql-backup/
1. backup.sh

![image](https://github.com/user-attachments/assets/8dbfb78a-8db4-494b-bc91-d418534a6e19)

2. Dockerfile.mysql-backup

![image](https://github.com/user-attachments/assets/937224c4-c121-4a2f-94e0-2a450bf16a95)

3. crontab

![image](https://github.com/user-attachments/assets/77312e4c-0bd8-4b1f-81b5-5d5b6e8bc720)






