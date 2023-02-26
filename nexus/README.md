# Nexus Repository

```sh
# docker run -d -p 8081:8081 --name nexus sonatype/nexus3:3.42.0
docker compose up
```

```sh
docker exec local-nexus cat /nexus-data/admin.password
# e36c7fa9-c6be-45c6-a95c-3058ba62f5dd
```

`admin` 계정으로 로그인한 후 원하는 repository를 추가한다.
