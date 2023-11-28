### Minio 

### 使用说明

>minio 有三个大版本。启动方式也不一样。

- 3.x 镜像版本 `minio/minio:RELEASE.2020-12-03T05-49-24Z`
```sh
      - command:
        - /bin/sh
        - -ce
        - /usr/bin/docker-entrypoint.sh minio -S /etc/minio/certs/ server /export
```
- 4.x 镜像版本 `minio/minio:RELEASE.2023-03-24T21-41-23Z`
```sh
        - command:
          - /bin/bash
          - -c
          args:
          - minio server /data --console-address :9001
```
- 5.x 镜像版本 `minio/minio:RELEASE.2023-11-20T22-40-07Z`
```sh
        command:
          - /bin/bash
          - -c
        args:
          - minio server /data --console-address :9001
```