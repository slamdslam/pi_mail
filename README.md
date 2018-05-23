_NOTE: The current instructions align best with the most recent version of the containers. I recommend pulling a new version of the repository and building a fresh image. You can remove the current version of the containers using `docker rm -f dovecot postfix rainloop`_

# Install Docker (Raspbian Stretch)
The easiest method of installing Docker on a Raspberry Pi running Raspbian, is to execute a scripted install from the Docker website. 

1. `curl -sSL get.docker.com | sh` to install Docker
2. `sudo usermod -aG docker pi` to allow the pi user to manage Docker
3. `echo gpu_mem=16 >> /boot/config.txt` to decrease memory reserved for graphics operations (Optional step to improve performance)

# Launch and configure mail services
In its current state, this repository provides a minimally functional ARM-based mail environment based on separate postfix (SMTP), dovecot (IMAP), and rainloop (webmail) containers.

1. Clone or copy the repo to a raspberry pi or other ARM based device.
2. Navigate to the sub-directories for each service and build a tagged image. 
- `docker build -t postfix .` in `mail/postfix`
- `docker build -t dovecot .` in `mail/dovecot`
- `docker build -t rainloop .` in `mail/rainloop`
3. Create a user-defined container network: 
- `docker network create --driver bridge --opt com.docker.network.bridge.name=docker-mail docker-mail`
4. Launch the containers as shown below:
- `docker run -itd --name dovecot --network docker-mail -p 143:143 dovecot`
- `docker run -itd --name postfix --network docker-mail -p 25:25 postfix`
- `docker run -itd --name rainloop --network docker-mail -p 80:80 rainloop`
5. In a browser, navigate to `http://SERVER/?admin` to configure a new domain:
- Configure a new domain called `test.pi`
  - Default admin: `admin` / `12345`
  - Test domain: `test.pi`
  - SMTP: `postfix.docker-mail:587`
  - IMAP: `dovecot.docker-mail:143`
- Set the default domain under the Login tab to `test.pi`
6. Navigate to `http://SERVER/` to access webmail.
- Test user: `pi@test.pi` / `password`

# Add / remove users
You can add or remove users by editing the included `users_template` and copying it into the running container at `/etc/dovecot/users`.

- `docker cp LOCAL_FILE CONTAINER_NAME:/etc/dovecot/users`

*CONTAINER__NAME = dovecot if you have followed examples above*

The general format of the user file is one `username:password` pair per line. The username should not include the _@domain_ suffix. The template demonstrates the format of adding a user with a plaintext password. You can leverage the dovecot container image to create a strong password hash:

- `docker run -it --rm IMAGE_NAME doveadm pw -s SHA256-CRYPT`

*IMAGE_NAME = dovecot if you have followed examples above*

# Restart suspended containers
At the moment, restarting the device will result in the containers being suspended. You can restart a suspended container by calling `docker start` with the container name. To create a container that restarts itself, call `docker run` with the `--restart always` parameter.
