## Docker安装

### docker启动

```bash
mkdir -p ~/minio/data
docker run \
 -p 9000:9000 \
 -p 9090:9090 \
 --name minio \
 -v ~/minio/data:/data \
 -e "MINIO_ROOT_USER=ROOTNAME" \
 -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
 quay.io/minio/minio server /data --console-address ":9090"
```

### docker compose配置

```bash
minio:
    container_name: minio
    image: "quay.io/minio/minio"
    ports:
      - "9000:9000"
      - "9090:9090"
    volumes:
      - "/home/keepthinker/minio/data:/data"
    environment:
      MINIO_ROOT_USER: "username"
      MINIO_ROOT_PASSWORD: "password"
    command:
      server /data  --console-address ":9090"
```

### Install the MinIO Client

The MinIO Client allows you to work with your MinIO server from the commandline.

Download the mc client and install it to a location on your system PATH such as /usr/local/bin. You can alternatively run the binary from the download location.

```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/mc
```

Use mc alias set to create a new alias associated to your local deployment. You can run mc commands against this alias:

```bash
mc alias set local http://127.0.0.1:9000 minioadmin minioadmin
mc admin info local
```

## Access Management

MinIO uses Policy-Based Access Control (PBAC) to define the authorized actions and resources to which an authenticated user has access.



## 参考

[MinIO High Performance Object Storage &#8212; MinIO Object Storage for Container](https://min.io/docs/minio/container/index.html)

[Java Client API Reference &#8212; MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/java/API.html#getPresignedObjectUrl)

[Java Quickstart Guide &#8212; MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/java/minio-java.html)

[Access Management &#8212; MinIO Object Storage for Linux](https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html#id5)
