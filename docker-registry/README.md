# Docker Registry

- [Docker Registry](#docker-registry)
  - [Distribution](#distribution)
  - [Docker Registry vs. Docker Repositor](#docker-registry-vs-docker-repositor)
  - [실행](#실행)
  - [기본 API](#기본-api)
  - [이미지 업로드](#이미지-업로드)
  - [repository 목록](#repository-목록)
  - [특정 repository의 tag 목록](#특정-repository의-tag-목록)
  - [이미지 manifest 조회](#이미지-manifest-조회)
  - [이미지 삭제](#이미지-삭제)
  - [참고](#참고)

## Distribution

- [xpdojo/kubernetes](https://github.com/xpdojo/kubernetes/blob/main/registry/docker-distribution.md)

## Docker Registry vs. Docker Repositor

> [Azure Tips and Tricks](https://microsoft.github.io/AzureTipsAndTricks/blog/tip57.html)

**Docker Registry** is a service that stores your docker images,
but it could be hosted by a third party and even private if you need so.
A couple of examples are:

- Docker Hub(opens new window): `default`
- Azure Container Registry(opens new window)
- Quay Enterprise(opens new window)
- There are also other choices such as Google or AWS Container Registry.

A **Docker Repository** is a collection of related images with same name, that have different tags.
Tags are an alphanumeric identifier attached to images within a repository (e.g., 1.1 or latest).

## 실행

```sh
docker compose up
```

## 기본 API

## 이미지 업로드

private registry에 업로드하기 위해서는 tag에 registry 주소를 붙여야 한다.
사실 모든 이미지가 registry 주소를 붙여야 하지만,
생략할 경우 Docker Hub(`docker.io`)가 기본으로 설정된다.

> `docker pull ubuntu` instructs docker to pull an image
> named `ubuntu` from the official Docker Hub.
> This is simply a shortcut for the longer `docker pull docker.io/library/ubuntu` command -
> [Understanding image naming](https://docs.docker.com/registry/introduction/#understanding-image-naming), Docker Docs

```sh
sudo docker tag redis:7.0.5-bullseye localhost:5000/redis:7.0.5-bullseye
```

```sh
sudo docker push localhost:5000/redis:7.0.5-bullseye
```

## repository 목록

```sh
# curl -X GET http://localhost:5000/v2/_catalog
http get http://localhost:5000/v2/_catalog
```

```json
{
    "repositories": [
        "redis"
    ]
}
```

## 특정 repository의 tag 목록

```sh
# curl -X GET http://localhost:5000/v2/redis/tags/list
http get http://localhost:5000/v2/redis/tags/list
```

```json
{
    "name": "redis",
    "tags": [
        "7.0.5-bullseye"
    ]
}
```

## 이미지 manifest 조회

```sh
# curl -X GET http://localhost:5000/v2/redis/manifests/7.0.5-bullseye
http get http://localhost:5000/v2/redis/manifests/7.0.5-bullseye
```

```json
HTTP/1.1 200 OK
Content-Length: 13646
Content-Type: application/vnd.docker.distribution.manifest.v1+prettyjws
Date: Tue, 28 Feb 2023 05:10:38 GMT
Docker-Content-Digest: sha256:c82dc2aef993164a1536c50be8f7e3ab49aa4835fead49368845b6e8e662d660
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:c82dc2aef993164a1536c50be8f7e3ab49aa4835fead49368845b6e8e662d660"
X-Content-Type-Options: nosniff

{
   "schemaVersion": 1,
   "name": "redis",
   "tag": "7.0.5-bullseye",
   "architecture": "amd64",
   "fsLayers": [
      {
         "blobSum": "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
      },
      ...
   ],
   "history": [
      {},
      ...
   ],
   "signatures": [
      {
         "header": {
            "jwk": {
               "crv": "P-256",
               "kid": "LKH7:T745:OGZI:LI5Y:XNJW:2DKA:D4MS:CFXS:QGGU:MHFS:ANKD:CYDU",
               "kty": "EC",
               "x": "cTNGLiwiLkgjBCGdPzr8jiB4ELbBtwHJoRDKjO3I2Yg",
               "y": "qvWgFiJyFrLm7syf606B7CTJWzs42Q2H0bhgyXFgdxo"
            },
            "alg": "ES256"
         },
         "signature": "s4KNGj5RR_dapOY3kHIRH--SI-Q1T95pUWU2PEts72zzp3Yo7m6Urf0JBYYz_PcvMUJ0vZlyJAJ8z6I_UpwnKg",
         "protected": "eyJmb3JtYXRMZW5ndGgiOjEyOTk4LCJmb3JtYXRUYWlsIjoiQ24wIiwidGltZSI6IjIwMjMtMDItMjhUMDU6MTA6MzhaIn0"
      }
   ]
}
```

## 이미지 삭제

```sh
# curl -X DELETE http://localhost:5000/v2/redis/manifests/sha256:c82dc2aef993164a1536c50be8f7e3ab49aa4835fead49368845b6e8e662d660
http delete http://localhost:5000/v2/redis/manifests/sha256:c82dc2aef993164a1536c50be8f7e3ab49aa4835fead49368845b6e8e662d660
```

```json
HTTP/1.1 404 Not Found

{
    "errors": [
        {
            "code": "MANIFEST_UNKNOWN",
            "message": "manifest unknown"
        }
    ]
}
```

*왜 다른 digest가 표시되지?*

```sh
ls ${DOCKER_REGISTRY_HOME}/data/images/docker/registry/v2/repositories/redis/_manifests/revisions/sha256
# 2bd864580926b790a22c8b96fd74496fe87b3c59c0774fe144bab2788e78e676
```

```sh
http delete http://localhost:5000/v2/redis/manifests/sha256:2bd864580926b790a22c8b96fd74496fe87b3c59c0774fe144bab2788e78e676
```

```json
HTTP/1.1 202 Accepted
Content-Length: 0
Date: Tue, 28 Feb 2023 05:40:21 GMT
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
```

## 참고

- [Docker Registry](https://docs.docker.com/registry/)
- [Understanding image naming](https://docs.docker.com/registry/introduction/#understanding-image-naming), Docker Docs
- [Docker Registry v2 HTTP API](https://docs.docker.com/registry/spec/api/)
