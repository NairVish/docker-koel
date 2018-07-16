# docker-koel

This is a docker image for the [koel](https://koel.phanan.net/) music streaming server. It is derived from [0xcaff's minimal image](https://github.com/0xcaff/docker-koel), which did not work for me as-is. My setup involved the use of an nginx reverse proxy which proxied requests between the client and the koel container and also provided SSL. The existing setup led to mixed content errors/requests which prevented any of the pages from loading as the connection from the reverse proxy to the Apache web server in the container was unencrypted. I have made modifications to the image to encrypt this connection using a self-signed cert in the container, which resulted in this setup finally working.

## Usage

1. Modify koel's environment variables in `docker-compose.yml` as per your requirements.
2. Create your nginx conf as per your requirements to serve as a reverse proxy. Here is my root location block (which resulted in a working configuration):
```
location / {
    proxy_pass https://127.0.0.1:8080;
    proxy_set_header X-Forwarded-Host $server_name;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   Host $host;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto https;
    
    # Not sure if these next two lines are needed. I did not remove them as
    # I did not want risk breaking my working configuration. Just remember
    # to replace "koel.domain.tld" with your instance's domain.
    sub_filter "http://koel.domain.tld" "https://koel.domain.tld";
    sub_filter_once off;
}
```
3. Run `docker-compose up -d` in the working directory of this repo.

## Post-initialization

On the first start (after an upgrade or initial installation), the database
needs to be migrated. Run koel init with `docker exec` in the koel runtime
container:

    docker exec -it koel_docker_koel_1 php artisan koel:init
    
You can also have koel sync your media files via the command line using:

    docker exec -it koel_docker_koel_1 php artisan koel:sync

(If different, replace `koel_docker_koel_1` with the name of your koel container as seen in the last column of output from `docker container ls`.)

## Credits

All credit goes to [0xcaff](https://github.com/0xcaff) for creating the original image, and [phanan](https://github.com/phanan) for creating koel!
