---
title: "Deployment Processes - Finding Another Way to Not Build a Light Bulb"
date: 2025-04-05
description: "Using Ansible and Docker Compose together to make your deployment process easier"
tags: ["fediverse", "projects", "habitat"]
showComments: true
---

> I have not failed 700 times. I’ve succeeded in proving 700 ways how not to build a light bulb.

This is apparently the response that Thomas Edison gave to a reporter when asked about his repeated failures in
attempting to invent the light bulb. I say _apparently_ because the exact quote is disputed, as is the validity of
Edison's accomplishments. I don't know, I'm no historian, but I sure do like the positivity of the quote!

Development often feels this way, banging your head against an error for what feels like a lifetime, only to celebrate
the success of having the opportunity to bang your head against a different error. The last few months have been no
exception.

## The Problem

To bring you up to speed, [I'm Building Habitat](/posts/building-habitat): A free and open source self-hosted
social platform for local communities. I have plans for it to be [decentralised](https://en.wikipedia.org/wiki/Fediverse).

For a while now, I've been struggling with the difficulty of using Ansible provisioning for Linux server hosting, along
with using PHP Deployer and Docker for those who might want to host an instance of Habitat on their own machine, or if
they have very deep pockets, Cloud Docker hosting.

My initial preference for hosting was with Docker, and so initially it got a lot of attention in development. After
trying it out, and finding that the costs for Cloud hosting were egregiously expensive, I decided to focus on
installation on an AWS EC2 instance. My efforts with Docker had fallen by the wayside in favour of Ansible.

I still wanted to make sure that Docker was supported, but it had taken a backseat, and my efforts had been effectively
doubled. Everything I built had to be tested on both types of environments.

## A Guiding Lightning Strike

Earlier this year, it seems that my project got the attention of [Nutomic](https://github.com/Nutomic), a maintainer of
Lemmy. This is exciting news for me. Lemmy is a federated, free and open source, self-hosted social platform. Does this
sound familiar at all?

Habitat could certainly benefit from the lessons that Lemmy has had over the years. Nutomic [posted on my project
board](https://github.com/carlnewton/habitat/issues/39#issuecomment-2569121925), and I snatched up the opportunity to
ask him about provisioning and deployment techniques. Here's what he said:

> What we are doing is Docker, with Ansible to set it up. So you can use both of them together to simplify things.

-- <cite>Nutomic, A smarter person than me</cite>

It's one of those solutions that seems so simple that it should've been obvious from the outset. Though just because the
solution was simple in design, it doesn't mean that it was a simple thing to implement. I've been pulling my hair out
for a couple of months over this now, but I think I'm finally in a good place with it.

## The Solution

Here is a simplified model of the architecture as it currently exists:

{{<mermaid>}}
    flowchart TB
        subgraph Docker Compose
            subgraph PHP Apache container
                A([Apache])
            end
            subgraph MariaDB container
                M([MariaDB])
                A-.-M
            end
        end
        subgraph Filesystem
            UF(Uploaded files)
            A-.-|Volume mount|UF
            SC(SSL Certificate)
            HA(Habitat Application)
            A-.-|Volume mount|HA
            DB(Database)
            M-.-|Volume mount|DB
        end
        A-.-|Volume mount|SC
        C([Certbot])-.-SC
{{</mermaid>}}

### Ansible

First, we create the required directories:

```yaml
- name: Create the directory for which user uploaded files will live
  file:
    path: /opt/my-project/uploaded-files
    state: directory
    mode: 0755

- name: Create the directory for the database to be managed by MariaDB
  file:
    path: /opt/my-project/db
    state: directory
    mode: 0755

- name: Copy the Docker Compose instructions from the repository to a new directory
  copy:
    src: files/docker-compose
    dest: /opt/my-project/
    mode: 0755

- name: Download the repository for the Application
  get_url:
    url: "{{ application_repo_url }}/archive/refs/heads/{{ application_repo_version }}.zip"
    dest: /tmp
  register: application_archive

- name: Unarchive the application
  unarchive:
    src: "{{ application_archive.dest }}"
    dest: /tmp
    remote_src: yes
    list_files: yes
  register: application_unarchive

- name: Move application directory
  command: "mv /tmp/{{ application_unarchive.files[0] }}/App/ /opt/my-project/app"
```

Given that Habitat has an AGPL licence, the administrator must make the source code available to the end users. Making
this easy to adhere to is another thing I'd like to automate. Despite the application files already existing locally,
instead of uploading them from the local machine, I've decided here to use variables to download the application from
the public repository. These variables can later be used in the application to link to the public repository. It's also
a lot faster than using Ansible to copy each of the files.

Next up, we deal with automating the creation of an SSL certificate.

```yaml
- name: Generate a new SSL certificate
  shell: |
    certbot certonly --standalone --non-interactive --agree-tos --email {{ email_address }} -d {{ domain }}

- name: Create a cron job for certbot auto renewal
  cron:
    name: "Auto-renew letsencrypt certificate with certbot"
    minute: "0"
    hour: "1"
    day: "*"
    month: "*"
    weekday: "0"
    job: "certbot renew --post-hook 'docker compose -f /opt/my-project/docker-compose/docker-compose.yaml restart php-apache'"
    user: "root"
```

This checks the current status of the SSL certificate once a week, and if it's due to expire, it generates a new one and
then restarts the PHP Apache container.

The final step of the playbook is to run the Docker compose up:

```yaml
- name: Run docker compose up
  community.docker.docker_compose_v2:
    project_src: /opt/my-project/docker-compose
```

That's the outermost layer done! Onto the detail of that Docker Compose setup.

### Docker Compose

Firstly, let's look at the simpler container, MariaDB:

```yaml
  db:
    container_name: "my-project-mariadb"
    image: mariadb:latest
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      start_period: 10s
      interval: 10s
      timeout: 5s
      retries: 3
    environment:
      - MARIADB_ROOT_USER=root
      - MARIADB_ROOT_PASSWORD=${MARIADB_PASSWORD}
      - MARIADB_USER=${MARIADB_USER}
      - MARIADB_PASSWORD=${MARIADB_PASSWORD}
      - MARIADB_DATABASE=${MARIADB_DATABASE}
    volumes:
      - /opt/my-project/db:/var/lib/mysql:rw
    ports:
      - "3306:3306"
    restart: always
```

We attach the db directory created earlier as a volume, supply the credentials and create a health check for which the
Apache container can query.

Onto the Apache container:

```yaml
  php-apache:
    container_name: "my-project-php-apache"
    depends_on:
      db:
        condition: service_healthy
    build: ./php-apache
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 10
    environment:
      - MARIADB_USER=${MARIADB_USER}
      - MARIADB_PASSWORD=${MARIADB_PASSWORD}
      - MARIADB_DATABASE=${MARIADB_DATABASE}
    volumes:
      - /opt/my-project/app:/var/www/html
      - /opt/my-project/uploaded-files:/var/www/uploads
      - /etc/letsencrypt:/etc/letsencrypt
    ports:
      - "80:80"
      - "443:443"
    restart: always
```

Here we are forwarding the HTTP and HTTPS ports and mounting the application, uploaded files and letsencrypt directories
for the application to use. We are also sending it the same database credentials.

The configuration examples here lack a few extra details that I've added specifically for Habitat, you can always go and
[take a look at the repository](https://github.com/carlnewton/habitat) if you want to see more detail.
