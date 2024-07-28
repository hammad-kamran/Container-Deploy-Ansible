# Container Deploy Ansible

Ansible playbooks for deploying Docker containers: MySQL, Zabbix, Nagios, and Prometheus.

## Prerequisites

Before running the playbooks, ensure you have:

- **Ansible** installed on your control machine.
- **Docker** installed on all target machines.
- Target machines listed in your Ansible inventory file.

## Getting Started

1. **Clone the Repository:**
    ```bash
    git clone https://github.com/yourusername/container-deploy-ansible.git
    cd container-deploy-ansible
    ```

2. **Update the Inventory File:**
    Edit your Ansible inventory file (`/etc/ansible/hosts`) to include your target machines:
    ```ini
    [debian_agents]
    opsoura3 ansible_host=103.151.111.241 ansible_user=root
    opsoura4 ansible_host=103.151.111.237 ansible_user=root
    ```

3. **Run the Playbook:**
    ```bash
    ansible-playbook deploy_docker.yml
    ```

## Playbook Overview

The `deploy_docker.yml` playbook will:

- Pull and run MySQL container.
- Pull and run Zabbix container.
- Pull and run Nagios container.
- Pull and run Prometheus container.

Here is the `deploy_docker.yml` content for reference:

```yaml
- hosts: debian_agents
  become: yes
  tasks:
    - name: Pull MySQL Docker image
      community.docker.docker_image:
        name: mysql
        tag: latest
        source: pull

    - name: Run MySQL container
      community.docker.docker_container:
        name: mysql-server
        image: mysql:latest
        state: started
        env:
          MYSQL_ROOT_PASSWORD: "root_password"
          MYSQL_DATABASE: "zabbix"
          MYSQL_USER: "zabbix"
          MYSQL_PASSWORD: "zabbix_password"
        ports:
          - "3312:3306"

    - name: Pull Zabbix Docker image
      community.docker.docker_image:
        name: zabbix/zabbix-server-mysql
        tag: latest
        source: pull

    - name: Run Zabbix container
      community.docker.docker_container:
        name: zabbix-server
        image: zabbix/zabbix-server-mysql:latest
        state: started
        ports:
          - "8090:80"
          - "10058:10051"
        env:
          DB_SERVER_HOST: "mysql-server"
          MYSQL_USER: "zabbix"
          MYSQL_PASSWORD: "zabbix_password"
          MYSQL_DATABASE: "zabbix"

    - name: Pull Nagios Docker image
      community.docker.docker_image:
        name: jasonrivers/nagios
        tag: latest
        source: pull

    - name: Run Nagios container
      community.docker.docker_container:
        name: nagios-server
        image: jasonrivers/nagios:latest
        state: started
        ports:
          - "8091:80"

    - name: Pull Prometheus Docker image
      community.docker.docker_image:
        name: prom/prometheus
        tag: v2.30.3
        source: pull

    - name: Run Prometheus container
      community.docker.docker_container:
        name: prometheus-server
        image: prom/prometheus:v2.30.3
        state: started
        ports:
          - "8092:9090"
