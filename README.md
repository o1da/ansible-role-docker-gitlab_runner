docker_gitlab_runner
====================

Role which installs and registers GitLab Runner via Docker.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```yaml
---

- name: Default usage
  hosts: all
  vars:
    # GitLab server URL
    docker_gitlab_runner_server_url: https://gitlab.example.com
    # Registration token
    docker_gitlab_runner_reg_togen: mytoken
  roles:
    - docker
    - docker_gitlab_runner

- name: Example of how to run multiple runners on one host
  hosts: all
  vars:
    # GitLab server URL
    docker_gitlab_runner_server_url: https://gitlab.example.com
    # Registration token
    docker_gitlab_runner_reg_togen: mytoken
  roles:
    - roles: docker_gitlab_runner
      # Make sure each container has different name
      docker_gitlab_runner_dname: gitlab-runner1
    - roles: docker_gitlab_runner
      # Make sure each container has different name
      docker_gitlab_runner_dname: gitlab-runner2

- name: Example of how to customize the registration command
  hosts: all
  vars:
    # GitLab server URL
    docker_gitlab_runner_server_url: https://gitlab.example.com
    # Registration token
    docker_gitlab_runner_reg_togen: mytoken
    # Add list of tags to the runner
    docker_gitlab_runner_reg_opts__custom:
      tag-list: tag1,tag2
  roles:
    - docker
    - docker_gitlab_runner

- name: Example of how to use custom CA for the registration
  hosts: all
  vars:
    # Destination where to install the CA on the host
    ssl_cert_ca_file: /etc/ssl/certs/gitlab_ca.crt
    # Content of the CA
    ssl_cert_ca: 
    # GitLab server URL
    docker_gitlab_runner_server_url: https://gitlab.example.com
    # Registration token
    docker_gitlab_runner_reg_togen: mytoken
    # Add custom volume to mount into the container
    docker_gitlab_runner_volumes__custom:
      - /etc/ssl/certs/gitlab_ca.crt:/etc/gitlab-runner/certs/ca.crt
  roles:
    - docker
    - ssl_cert
    - docker_gitlab_runner
```


Role variables
--------------

Variables used by the role:

```yaml
# GitLab Runner Docker image
docker_gitlab_runner_rimg: gitlab/gitlab-runner

# GitLab Runner Docker image tag
docker_gitlab_runner_rimg_tag: latest

# GitLab Runner name
docker_gitlab_runner_rname: "{{ ansible_hostname }}"

# Docker image
docker_gitlab_runner_dimg: docker

# Docker image tag
docker_gitlab_runner_dimg_tag: latest

# Docker container name
docker_gitlab_runner_dname: gitlab-runner

# Gitlab server URL - must be set by the user
docker_gitlab_runner_server_url: null

# GitLab registration token - must be set by the user
docker_gitlab_runner_reg_token: null

# Docker socket path on the host
docker_gitlab_runner_sock_src: /var/run/docker.sock

# Docker socket path inside the container
docker_gitlab_runner_sock_dst: "{{ docker_gitlab_runner_sock_src }}"

# Default list of docker volumes
docker_gitlab_runner_volumes__default:
  - "{{ docker_gitlab_runner_sock_src }}:{{ docker_gitlab_runner_sock_dst }}"

# Custom list of docker volumes
docker_gitlab_runner_volumes__custom: []

# Final list of docker volumes
docker_gitlab_runner_volumes: "{{
  docker_gitlab_runner_volumes__default +
  docker_gitlab_runner_volumes__custom }}"

# Whether to force registration
docker_gitlab_runner_reg_force: no

# Default registration executor options
docker_gitlab_runner_reg_exec_opts__default:
  executor: docker
  docker-image: "{{ docker_gitlab_runner_dimg }}:{{ docker_gitlab_runner_dimg_tag }}"
  docker-tlsverify: no
  docker-disable-cache: yes
  docker-privileged: yes

# Custom registration executor options
docker_gitlab_runner_reg_exec_opts__custom: {}

# Final registration executor options
docker_gitlab_runner_reg_exec_opts: "{{
  docker_gitlab_runner_reg_exec_opts__default.update(
  docker_gitlab_runner_reg_exec_opts__custom) }}{{
  docker_gitlab_runner_reg_exec_opts__default }}"

# Default registration options
docker_gitlab_runner_reg_opts__default:
  non-interactive: ""
  name: "{{ docker_gitlab_runner_rname }}"
  url: "{{ docker_gitlab_runner_server_url }}"
  registration-token: "{{ docker_gitlab_runner_reg_token }}"

# Custom registration options
docker_gitlab_runner_reg_opts__custom: {}

# Final registration options
docker_gitlab_runner_reg_opts: "{{
  docker_gitlab_runner_reg_opts__default.update(
  docker_gitlab_runner_reg_exec_opts) }}{{
  docker_gitlab_runner_reg_opts__default.update(
  docker_gitlab_runner_reg_opts__custom) }}{{
  docker_gitlab_runner_reg_opts__default }}"
```


Dependencies
------------

- [`docker`](https://github.com/jtyr/ansible-docker) (optional)
- [`ssl_cert`](https://github.com/jtyr/ansible-ssl_cert) (optional)


License
-------

MIT


Author
------

Jiri Tyr
