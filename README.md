# Sprawozdanie_Lab7

# Krok 1: Instalacja buildx

```
docker buildx create --use --name mybuilder
docker buildx inspect mybuilder --bootstrap
```

# Krok 2: Utworzenie pliku Dockerfile

```
FROM node:14-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:alpine
COPY --from=build /app /usr/share/nginx/html
```

# Krok 3: Zdefiniowanie zmiennych środowiskowych

```
$env:IMAGE_NAME="konsulrima/labtest:v7"
$env:CACHE_NAME="konsulrima/labtest-cache:latest"
```

# Krok 4: Budowanie obrazu z użyciem cache

```
1. Budujemy obraz bez cache i wyeksportujemy cache do rejestru:

docker buildx build `
  --platform linux/amd64,linux/arm64 `
  -t $env:IMAGE_NAME `
  --cache-to=type=registry,ref=$env:CACHE_NAME,mode=max `
  --push .

2. Używamy cache przy kolejnym budowaniu obrazu:

docker buildx build `
  --platform linux/amd64,linux/arm64 `
  -t $env:IMAGE_NAME `
  --cache-from=type=registry,ref=$env:CACHE_NAME `
  --cache-to=type=registry,ref=$env:CACHE_NAME,mode=max `
  --push .

3. Weryfikacja użycia cache:

 => importing cache manifest from konsulrima/labtest-cache:latest                                         2.7s
 => => inferred cache manifest type: application/vnd.oci.image.index.v1+json                              0.0s
 => [linux/amd64 build 1/6] FROM docker.io/library/node:14-alpine@sha256:434215b487a329c9e867202ff89e704  0.3s
 => => resolve docker.io/library/node:14-alpine@sha256:434215b487a329c9e867202ff89e704d3a75e554822e07f3e  0.3s
 => [linux/amd64 stage-1 1/2] FROM docker.io/library/nginx:alpine@sha256:516475cc129da42866742567714ddc6  0.3s
 => => resolve docker.io/library/nginx:alpine@sha256:516475cc129da42866742567714ddc681e5eed7b9ee0b9e9c01  0.3s
 => [linux/arm64 stage-1 1/2] FROM docker.io/library/nginx:alpine@sha256:516475cc129da42866742567714ddc6  0.3s
 => => resolve docker.io/library/nginx:alpine@sha256:516475cc129da42866742567714ddc681e5eed7b9ee0b9e9c01  0.3s
 => [linux/arm64 build 1/6] FROM docker.io/library/node:14-alpine@sha256:434215b487a329c9e867202ff89e704  0.3s
 => => resolve docker.io/library/node:14-alpine@sha256:434215b487a329c9e867202ff89e704d3a75e554822e07f3e  0.2s
 => [internal] load build context                                                                         0.1s
 => => transferring context: 315B                                                                         0.0s
 => [auth] konsulrima/labtest-cache:pull token for registry-1.docker.io                                   0.0s
 => CACHED [linux/arm64 build 2/6] WORKDIR /app                                                           0.0s
 => CACHED [linux/arm64 build 3/6] COPY package*.json ./                                                  0.0s
 => CACHED [linux/arm64 build 4/6] RUN npm install                                                        0.0s
 => CACHED [linux/arm64 build 5/6] COPY . .                                                               0.0s
 => CACHED [linux/arm64 build 6/6] RUN npm run build                                                      0.0s
 => CACHED [linux/arm64 stage-1 2/2] COPY --from=build /app /usr/share/nginx/html                         0.0s
 => CACHED [linux/amd64 build 2/6] WORKDIR /app                                                           0.0s
 => CACHED [linux/amd64 build 3/6] COPY package*.json ./                                                  0.0s
 => CACHED [linux/amd64 build 5/6] COPY . .                                                               0.0s
 => CACHED [linux/amd64 build 6/6] RUN npm run build                                                      0.0s
 => CACHED [linux/amd64 stage-1 2/2] COPY --from=build /app /usr/share/nginx/html                         0.0s
 => exporting to image                                                                                    5.4s
 => => exporting layers                                                                                   0.0s
 => => exporting manifest sha256:ea63f65dac985bf36e82177fe5a91e42a27092c737bdb92e8e21c14834c5c134         0.1s
 => => exporting config sha256:a5204253f8b6388040291364ee9f63e27e8fc4ae8d8091fcd5eeed72c1cbabb6           0.0s
 => => exporting attestation manifest sha256:7e74fc5121dbc31a28c63e2a7e37fca86c83a1c84f1cd11c0f0a0a5d73a  0.1s
 => => exporting manifest sha256:07dfade5add2569b406cb6a7c7257a0e05d9db7b0f392a49574d8e58ae498cba         0.0s
 => => exporting config sha256:e186a2d639c41deda7d5850a18e659f5c51fff56bfbbb6cc54eea636c2a6aff5           0.0s
 => => exporting attestation manifest sha256:cdf001d214bc93c5234360721141a870ce48020e01563fc36d2df2a5ca1  0.1s
 => => exporting manifest list sha256:c49327a85ddc65f74355114831792f254052280401986aaa41f0fbeec1658d62    0.1s
 => => pushing layers                                                                                     2.8s
 => => pushing manifest for docker.io/konsulrima/labtest:v7@sha256:c49327a85ddc65f74355114831792f2540522  1.9s
 => [auth] konsulrima/labtest:pull,push token for registry-1.docker.io                                    0.0s
 => exporting cache to registry                                                                          12.9s
 => => preparing build cache for export                                                                  12.9s
 => => writing layer sha256:0d0c16747d2c6b6c26c064652afcb964c15f1b1e596ec052b2aa19b83948ae27              1.1s
 => => writing layer sha256:13fcfbc94648785b918ecc1af675ac5187cdfc30f4fdaf9afa8bd2e9dedf548b              0.2s
 => => writing layer sha256:1580aa8be68b95f9f911eaefe6fce439411185f29c2bc9305d4abdc1313d3469              0.4s
 => => writing layer sha256:166b80e00f74c3a053c98236bd987af4ee91e23276257bf52858ebc900d55bb3              0.2s
 => => writing layer sha256:182e691fb2cc710f857dcd9016d4854cea790a28508e33bf2ef928f61ae6c4d6              0.2s
 => => writing layer sha256:4abcf20661432fb2d719aaf90656f55c287f8ca915dc1c92ec14ff61e67fbaf8              0.2s
 => => writing layer sha256:4f6b4e3940df0e5a4b975a6943afdcd034a2a0e2404258436cc76fd145d04d0c              0.2s
 => => writing layer sha256:5406ed7b06d9a94b5bd15843d2a1c7e38796a3ec5dc7f40f16f70cc1d045f453              0.2s
 => => writing layer sha256:561cb69653d56a9725be56e02128e4e96fb434a8b4b4decf2bdeb479a225feaf              0.2s
 => => writing layer sha256:58fbfa0c6cf298e9fc0d945f739b24dafcfa88a2b583ad68c939b6a7876fc656              0.6s
 => => writing layer sha256:6f276336db8f65d4ea517aa94e213a15c35dbd5345dac8c4aea6a6b93782a75b              0.3s
 => => writing layer sha256:861626db486980e79bb0f6df66f2d35de79880554390c346094c183cf780316c              0.5s
 => => writing layer sha256:8a095dcd659d469204606bfe9f3b0badf15ff7c25a932d99df5afe4e7299ccd6              0.4s
 => => writing layer sha256:8a3742a9529dc5c00974dfcf5e465be9f1606ff8a1911527b3928cf86ad57465              0.3s
 => => writing layer sha256:8f665685b215c7daf9164545f1bbdd74d800af77d0d267db31fe0345c0c8fb8b              0.4s
 => => writing layer sha256:98ff282c446606ae80b6c7812c9248a7b9d56f86b18c0b9b068ba821517cdb66              0.2s
 => => writing layer sha256:9a77618b11f6a65ecf1d0f3ba22b48dd2dd6033187f3fe80f5bc2049129d8894              0.6s
 => => writing layer sha256:a83296a673ce8848db1a03260b4104984c299b3619a73842eeecb887e6ebd1c0              0.2s
 => => writing layer sha256:b4f3127eb6227853129ee22d81700ccf48bcdcb17929680fc9f01ac3c7bc6dbd              0.2s
 => => writing layer sha256:d4bca490e609acaaf54ca73363442d31a31fd136a47a20a12370cf2025f0a10b              0.2s
 => => writing layer sha256:ddf9db5a05cbf61e62d46b1225986f3238390e770aedf7c3633b0f4984df6a6b              0.2s
 => => writing layer sha256:e5fca6c395a62ec277102af9e5283f6edb43b3e4f20f798e3ce7e425be226ba6              0.2s
 => => writing layer sha256:e6ef242c157026935bf8a69e6cf19f8f6635e44507c813daf0cc644f2e22396b              0.2s
 => => writing layer sha256:f56be85fc22e46face30e2c3de3f7fe7c15f8fd7c4e5add29d7f64b87abdaa09              0.2s
 => => writing layer sha256:fb6f98c365db3e5f6bf32873f213dafd80ee348fa72cf435a8fb4683d27be007              0.3s
 => => writing layer sha256:fc21a1d387f514f53589abea6d67cd6b329dfd3c9059bc96a552af3b3c97b413              0.2s
 => => writing config sha256:524d09137ca452579d079113b86cdd111024ba094880862cc1ee98d0b6be2624             1.2s
 => => writing cache manifest sha256:dcbce457338f854e6e7a2729e0dd65fe71a9500a986ffa502126726aeebaf39d     1.3s
 => [auth] konsulrima/labtest-cache:pull,push token for registry-1.docker.io                              0.0s
```

# Krok 5: Potwierdzenie wieloplatformowego obrazu

```
docker buildx imagetools inspect $env:IMAGE_NAME

>>
Name:      docker.io/konsulrima/labtest:v7
MediaType: application/vnd.oci.image.index.v1+json
Digest:    sha256:c49327a85ddc65f74355114831792f254052280401986aaa41f0fbeec1658d62

Manifests:
  Name:        docker.io/konsulrima/labtest:v7@sha256:ea63f65dac985bf36e82177fe5a91e42a27092c737bdb92e8e21c14834c5c134
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/amd64

  Name:        docker.io/konsulrima/labtest:v7@sha256:07dfade5add2569b406cb6a7c7257a0e05d9db7b0f392a49574d8e58ae498cba
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/arm64

  Name:        docker.io/konsulrima/labtest:v7@sha256:7e74fc5121dbc31a28c63e2a7e37fca86c83a1c84f1cd11c0f0a0a5d73aa4775
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    unknown/unknown
  Annotations:
    vnd.docker.reference.type:   attestation-manifest
    vnd.docker.reference.digest: sha256:ea63f65dac985bf36e82177fe5a91e42a27092c737bdb92e8e21c14834c5c134

  Name:        docker.io/konsulrima/labtest:v7@sha256:cdf001d214bc93c5234360721141a870ce48020e01563fc36d2df2a5ca1e44ed
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    unknown/unknown
  Annotations:
    vnd.docker.reference.digest: sha256:07dfade5add2569b406cb6a7c7257a0e05d9db7b0f392a49574d8e58ae498cba
    vnd.docker.reference.type:   attestation-manifest
```

