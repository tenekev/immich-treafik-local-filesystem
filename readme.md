
## CLI Import of local files

[Documentation](https://documentation.immich.app/docs/features/bulk-upload)

The official documentation doesn't address some details but it gives a general direction. Here is the rest.

### Mounting folders in compose
`immich-server` and `immich-microservices`need the necessary photos folder mounted like so.
This is necessary because the folder structure must be identical between the immich containers and the immich CLI tool.

```yml
services:
  immich-server:
    ...
    volumes:
      - /path/to/Folder1:/folder1:ro
      - /path/to/Folder2:/folder2:ro

  immich-microservices:
    ...
    volumes:
      - /path/to/Folder1:/folder1:ro
      - /path/to/Folder2:/folder2:ro
```

### Settings in the UI

Before continuing, you need to set up two things.

* An API key to use with the CLI tool.
* Define a root directory for the user.

⚠️ API keys are per user. You can use `username` and `password` but an API key is better.
![API keys](/images/api.png)



⚠️ **Defining a root directory for the user is crucial**. Otherwise the upload process through the CLI will fail due to bad permissions.
![API keys](/images/folders.png)

In this case, the Admin user will be able to upload only in `/folder1`. 

Another user can be given access to `/folder2`. 

If you want a user to see and upload to all folders, use `/`


### CLI commands

This creates an ephemeral container with the CLI tool that mounts the same photos folders and uploads from them to Immich.

```bash
docker run -it --rm \
  -v /path/to/Folder1:/folder1 \
  ghcr.io/immich-app/immich-cli:latest \
    upload \
    --key immich-api-key \
    --server http://<immich-proxy-addr>:8080/api \
    --recursive \
    --album \
    --import \
    --yes \
    /folder1

docker run -it --rm \
  -v /path/to/Folder2:/folder2 \
  ghcr.io/immich-app/immich-cli:latest \
    upload \
    --key immich-api-key \
    --server http://<immich-proxy-addr>:8080/api \
    --recursive \
    --album \
    --import \
    --yes \
    /folder2
```

### Adding a cronjob 

[Cron Job Generator](https://crontab.guru/every-10-minutes)

[Adding Cron Job](https://www.cyberciti.biz/faq/how-do-i-add-jobs-to-cron-under-linux-or-unix-oses/)

These are not full examples but they give an idea.

##### Job for single user that run every 10min
```bash
*/10 * * * * docker run ... -v /path/to/Folder1:/folder1 immich-app/immich-cli:latest upload ... --key api-key-user1 ... /folder1
*/10 * * * * docker run ... -v /path/to/Folder2:/folder2 immich-app/immich-cli:latest upload ... --key api-key-user1 ... /folder2
```

##### Job for multiple users that run every 10min
Take not of folders and keys.
```bash
# User1
*/10 * * * * docker run ... -v /path/to/Folder1:/folder1 immich-app/immich-cli:latest upload ... --key api-key-user1 ... /folder1
*/10 * * * * docker run ... -v /path/to/Folder2:/folder2 immich-app/immich-cli:latest upload ... --key api-key-user1 ... /folder2

# User2
*/10 * * * * docker run ... -v /path/to/Folder3:/folder3 immich-app/immich-cli:latest upload ... --key api-key-user2 ... /folder3
*/10 * * * * docker run ... -v /path/to/Folder4:/folder4 immich-app/immich-cli:latest upload ... --key api-key-user2 ... /folder4
```