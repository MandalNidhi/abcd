--- 
---

<h1 align="center"><b>Ansible playbook

--- 
---
 </u></b></h1>

<input value='Print' type='button' onclick='window.print()' />

## **Table of Contents** 

| S.No |Content|
|:----:|---|
|  1   | [Create a Ansible role for tomcat containe](#Create-a-Ansible-role-for-tomcat-container)|
|  2   | [List out the directory created under /home/nidhi/shiksha/ansible/roles/tomcat](#List-out-the-directory-created-under-home/nidhi/shiksha/ansible/roles/tomcat)|
|  3   | [Edit main.yml available in the tasks folder to define the tasks to be executed](#Edit-main.yml-available-in-the-tasks-folder-to-define-the-tasks-to-be-executed)|
|  4   | [Edit main.yml available in the vars folder to define the Variables to be executed](#Edit-main.yml-available-in-the-vars-folder-to-define-the-Variables-to-be-executed)|
|  5   | [Create sentenv.sh and tomcat.yml in the Files directory](#Create-sentenv.sh-and-tomcat.yml-in-the-Files-directory)|
|  6   | [Copy helloworld.war file into files Directory](#Copy-helloworld.war-file-into-files-Directory)|
|  7   | [Create a playbook to run the roles](#Create-a-playbook-to-run-the-roles)|
|  8   | [Run the Playbook using below mentioned command](#Run-the-Playbook-using-below-mentioned-command)|
|  9   | [Check application deployed status](#Check-application-deployed-status)|
| 10   | [Create a Ansible role for Grafana and Prometheus](#Create-a-Ansible-role-for-Grafana-and-Prometheus)|
| 11   | [List out the directory created](#List-out-the-directory-created)|
| 12   | [Edit main.yml available in the tasks folder to define the tasks to be executed](#Edit-main.yml-available-in-the-tasks-folder-to-define-the-tasks-to-be-executed)|
| 13   | [Edit main.yml available in the vars folder to define the Variables to be executed](#Edit-main.yml-available-in-the-vars-folder-to-define-the-Variables-to-be-executed)|
| 14   | [Create Prometheus.yml in the Files directory](#Create-Prometheus.yml-in-the-Files-directory)|
| 15   | [Create a playbook to run the roles](#Create-a-playbook-to-run-the-roles)|
| 16   | [Run the Playbook using below mentioned command](#Run-the-Playbook-using-below-mentioned-command)|
| 17   | [Container Status](#Container-Status)|
| 18   | [Open a Browser and check the targets](#Open-a-Browser-and-check-the-targets)|
| 19   | [Now open a Grafana and create a dashboard](#Now-open-a-Grafana-and-create-a-dashboard)|
| 20   | [Add a Data Source and Select Prometheus:](#Add-a-Data-Source-and-Select-Prometheus)|
| 21   | [Configure Prometheus Data Source](#Configure-Prometheus-Data-Source)|
| 22   | [Import a Dashboard](#Import-a-Dashboard)|
| 23   | [Copy Exported ID from Google and Paste ID and Complete Import](#Copy-Exported-ID-from-Google-and-Paste-ID-and-Complete-Import)|
| 24   | [Save the Imported Dashboard](#Save-the-Imported-Dashboard)|
| 25   | [Grafana Dashboard](#Grafana-Dashboard)|

---

## Create a Ansible role for tomcat container 
```
[nidhi@node1 ansible]$ sudo ansible-galaxy init /home/nidhi/shiksha/ansible/roles/tomcat
```
```Output
[sudo] password for nidhi: 
 - Role /home/nidhi/shiksha/ansible/roles/tomcat was created successfully
```
## List out the directory created under /home/nidhi/shiksha/ansible/roles/tomcat

```
[nidhi@node1 ansible]$ tree /home/nidhi/shiksha/ansible/roles/tomcat
```
```Output
/home/nidhi/shiksha/ansible/roles/tomcat
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```
## Edit main.yml available in the tasks folder to define the tasks to be executed

```
[nidhi@node1 ansible]$ sudo cat  /home/nidhi/shiksha/ansible/roles/tomcat/tasks/main.yml
```
```YAML
---
# tasks file for /home/nidhi/shiksha/ansible/roles/tomcat

- name: Install Podman Python bindings
  package:
    name: python3-podman
    state: present

- name: Pull the Tomcat container image
  containers.podman.podman_image:
    name: "{{ tomcat_image }}"
    tag: latest

- name: Create a directory for the WAR file if not exists
  file:
    path: /tmp
    state: directory

- name: Ensure the helloworld.war file exists in files directory
  stat:
    path: "{{ war_source_path }}"
  register: war_file

- name: Fail if the helloworld.war file does not exist
  fail:
    msg: "The file {{ war_source_path }} does not exist."
  when: not war_file.stat.exists

- name: Copy helloworld.war to /tmp directory
  copy:
    src: "{{ war_source_path }}"
    dest: "{{ war_dest_path }}"

- name: Download JMX Prometheus Java agent JAR
  get_url:
    url: "{{ jmx_agent_url }}"
    dest: "{{ jmx_agent_dest }}"

- name: Create setenv.sh for Tomcat
  copy:
    src: setenv.sh
    dest: "{{ setenv_dest }}"
    mode: '0755'

- name: Create tomcat.yml for JMX exporter
  copy:
    src: tomcat.yml
    dest: "{{ tomcat_yml_dest }}"

- name: Run Tomcat container with JMX exporter
  containers.podman.podman_container:
    name: "{{ tomcat_container_name }}"
    image: "{{ tomcat_image }}"
    state: started
    ports: "{{ tomcat_ports }}"
    volumes: "{{ container_volumes }}"

- name: Wait for Tomcat to start
  wait_for:
    host: localhost
    port: 8080
    delay: 10
    timeout: 120

- name: Verify Tomcat container is running
  command: podman ps --filter "{{ tomcat_status_filter }}" --format '{{ "{{" }}.Status{{ "}}" }}'
  register: container_status

- name: Debug container status
  debug:
    var: container_status.stdout

- name: Print Tomcat container logs for debugging
  command: podman logs {{ tomcat_container_name }}
  register: tomcat_logs
  ignore_errors: yes

- name: Debug Tomcat logs
  debug:
    var: tomcat_logs.stdout

- name: Verify sample application is accessible
  uri:
    url: "{{ tomcat_url }}"
    status_code: 200
```

## Edit main.yml available in the vars folder to define the Variables to be executed

```
[nidhi@node1 ansible]$ sudo cat  /home/nidhi/shiksha/ansible/roles/tomcat/vars/main.yml
```
```YAML
---
# vars file for /home/nidhi/shiksha/ansible/roles/tomcat

tomcat_image: "docker.io/library/tomcat:latest"
war_source_path: "{{ role_path }}/files/helloworld.war"
war_dest_path: "/tmp/helloworld.war"
jmx_agent_url: "https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar"
jmx_agent_dest: "/tmp/jmx_prometheus_javaagent-0.16.1.jar"
setenv_dest: "/tmp/setenv.sh"
tomcat_yml_dest: "/tmp/tomcat.yml"
tomcat_container_name: "tomcat_container"
tomcat_ports:
  - "8080:8080"
  - "9999:9999"
container_volumes:
  - "/tmp/helloworld.war:/usr/local/tomcat/webapps/helloworld.war"
  - "/tmp/jmx_prometheus_javaagent-0.16.1.jar:/usr/local/tomcat/bin/jmx_prometheus_javaagent-0.16.1.jar"
  - "/tmp/setenv.sh:/usr/local/tomcat/bin/setenv.sh"
  - "/tmp/tomcat.yml:/usr/local/tomcat/bin/tomcat.yml"
tomcat_url: "http://localhost:8080/helloworld/"
tomcat_status_filter: "name=tomcat_container"
```
## Create sentenv.sh and tomcat.yml in the Files directory 

```
[nidhi@node1 ansible]$ sudo cat /home/nidhi/shiksha/ansible/roles/tomcat/files/setenv.sh
```
```Bash
#!/bin/bash
JAVA_OPTS="$JAVA_OPTS -javaagent:/usr/local/tomcat/bin/jmx_prometheus_javaagent-0.16.1.jar=9999:/usr/local/tomcat/bin/tomcat.yml"
export JAVA_OPTS
```
- **Tomact.yml**

```
[nidhi@node1 ansible]$ sudo cat /home/nidhi/shiksha/ansible/roles/tomcat/files/tomcat.yml
```

```Yaml
---
lowercaseOutputLabelNames: true
lowercaseOutputName: true
rules:
- pattern: 'Catalina<type=GlobalRequestProcessor, name="(\w+-\w+)-(\d+)"><>(\w+):'
  name: tomcat_$3_total
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat global $3
  type: COUNTER
- pattern: 'Catalina<j2eeType=Servlet, WebModule=//([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), name=([-a-zA-Z0-9+/$%~_-|!.]*), J2EEApplication=none, J2EEServer=none><>(requestCount|maxTime|processingTime|errorCount):'
  name: tomcat_servlet_$3_total
  labels:
    module: "$1"
    servlet: "$2"
  help: Tomcat servlet $3 total
  type: COUNTER
- pattern: 'Catalina<type=ThreadPool, name="(\w+-\w+)-(\d+)"><>(currentThreadCount|currentThreadsBusy|keepAliveCount|pollerThreadCount|connectionCount):'
  name: tomcat_threadpool_$3
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat threadpool $3
  type: GAUGE
- pattern: 'Catalina<type=Manager, host=([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), context=([-a-zA-Z0-9+/$%~_-|!.]*)><>(processingTime|sessionCounter|rejectedSessions|expiredSessions):'
  name: tomcat_session_$3_total
  labels:
    context: "$2"
    host: "$1"
  help: Tomcat session $3 total
  type: COUNTER

```


## Copy helloworld.war file into files Directory

```
[nidhi@node1 ansible]$ sudo cp helloworld.war /home/nidhi/shiksha/ansible/roles/tomcat/files/
```

## Create a playbook to run the roles

```
[nidhi@node1 ansible]$ sudo cat deploy-tomcat.yml
```
```Yaml
---
- name: Deploy Tomcat container and deploy WAR file using Podman
  hosts: localhost  # Specify localhost as the host

  roles:
    - tomcat
```

## Run the Playbook using below mentioned command
```
[nidhi@node1 ansible]$ sudo ansible-playbook deploy-tomcat.yml
```

## Check application deployed status
![Alt text](unnamed%20(1).png)


## Create a Ansible role for Grafana and Prometheus

```
[nidhi@node1 ansible]$ ansible-galaxy init tomcat_monitoring
```
##  List out the directory created

```
[nidhi@node1 ansible]$ tree tomcat_monitoring/
````

```Output
tomcat_monitoring/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```

## Edit main.yml available in the tasks folder to define the tasks to be executed

```
[nidhi@node1 ansible]$ sudo cat tomcat_monitoring/tasks/main.yml
```

``` YAML
---
# tasks file for tomcat_monitoring

- name: Install Podman Python bindings
  package:
    name: python3-podman
    state: present

- name: Pull the Prometheus container image
  containers.podman.podman_image:
    name: "{{ prometheus_image }}"
    tag: "{{ prometheus_tag }}"

- name: Pull the Grafana container image
  containers.podman.podman_image:
    name: "{{ grafana_image }}"
    tag: "{{ grafana_tag }}"

- name: Create Prometheus configuration directory
  file:
    path: "{{ prometheus_config_dir }}"
    state: directory

- name: Create Prometheus configuration file
  copy:
    content: |
      global:
        scrape_interval: 15s

      scrape_configs:
        - job_name: 'tomcat'
          static_configs:
            - targets: ['{{ tomcat_target }}']
    dest: "{{ prometheus_config_file }}"

- name: Run Prometheus container
  containers.podman.podman_container:
    name: prometheus
    image: "{{ prometheus_image }}:{{ prometheus_tag }}"
    state: started
    ports:
      - "9090:9090"
    volumes:
      - "{{ prometheus_config_dir }}:/etc/prometheus"

- name: Run Grafana container
  containers.podman.podman_container:
    name: grafana
    image: "{{ grafana_image }}:{{ grafana_tag }}"
    state: started
    ports:
      - "3000:3000"

- name: Wait for Prometheus to start
  wait_for:
    host: localhost
    port: 9090
    delay: 10
    timeout: 120

- name: Wait for Grafana to start
  wait_for:
    host: localhost
    port: 3000
    delay: 10
    timeout: 120

- name: Print Prometheus container logs for debugging
  command: podman logs prometheus
  register: prometheus_logs
  ignore_errors: yes

- name: Debug Prometheus logs
  debug:
    var: prometheus_logs.stdout

- name: Print Grafana container logs for debugging
  command: podman logs grafana
  register: grafana_logs
  ignore_errors: yes

- name: Debug Grafana logs
  debug:
    var: grafana_logs.stdout

- name: Print instructions for accessing Grafana
  debug:
    msg: "Grafana is running at http://localhost:3000. Default username: admin, Default password: admin"
```

## Edit main.yml available in the vars folder to define the Variables to be executed

```
[nidhi@node1 ansible]$ sudo cat tomcat_monitoring/vars/main.yml
```
```YAML
---
# vars file for tomcat_monitoring

prometheus_image: "prom/prometheus"
prometheus_tag: "latest"
grafana_image: "grafana/grafana"
grafana_tag: "latest"
prometheus_config_dir: "/tmp/prometheus"
prometheus_config_file: "/tmp/prometheus/prometheus.yml"
tomcat_target: "localhost:9999"
```

## Create Prometheus.yml in the Files directory 

```
[nidhi@node1 ansible]$ sudo vim tomcat_monitoring/files/prometheus.yml
```

```YAML
mkdir -p tomcat_monitoring/files
cat <<EOF > tomcat_monitoring/files/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'tomcat'
    static_configs:
      - targets: ['localhost:9999']
EOF
```
## Create a playbook to run the roles

```
[nidhi@node1 ansible]$ sudo cat deploy-monitoring.yml
```
```YAML
---
- name: Monitor Tomcat application with Prometheus and Grafana
  hosts: localhost
  roles:
    - tomcat_monitoring
```

## Run the Playbook using below mentioned command

```
[nidhi@node1 ansible]$ sudo ansible-playbook deploy-monitoring.yml
```

## Container Status 

![Alt text](Screenshot%20from%202024-06-28%2014-27-32.png)

**Check Jmx metrics** 

```
[root@node1 ansible]# curl localhost:9999/metrics
```
![Alt text](Screenshot%20from%202024-07-08%2023-01-45.png)

## Open a Browser and check the targets

```
http://localhost:9090/targets?search=
```

![Alt text](unnamed%20(2).png)


## Now open a grafana and create a dashboard

```
http://localhost:3000/
```
```Bydefault 
password :- admin 
User :- admin
```

![Alt text](unnamed.png)



## Add a Data Source and Select Prometheus:

> - In the Grafana dashboard, navigate to the "Settings" menu (gear icon on the left sidebar).
>  - Click on "Data Sources" and then click on "Add your first data source."
>  - Choose Prometheus from the list of available data sources.

![Alt text](unnamed%20(3).png)


## Configure Prometheus Data Source
> In the settings for Prometheus, set the HTTP URL to: http://localhost:9090

![Alt text](unnamed%20(4).png)



## Import a Dashboard
> - Return to the Grafana dashboard and click on the "+" icon on the left sidebar.
>  - Select "Import" to import a new dashboard.

![Alt text](unnamed%20(5).png)



## Copy Exported ID from Google and Paste ID and Complete Import
> - Paste the exported ID in the designated field in the Grafana import dialog.
> - Follow the import process, confirming settings and making adjustments if needed.


![Alt text](unnamed%20(6).png)


## Save the Imported Dashboard
> Once the import is complete, save the dashboard. You can do this by clicking the "Save" icon and providing a name for your dashboard.

## Grafana Dashboard
![Alt text](unnamed%20(7).png)

<h1 align="center"><b>*****</u></b></h1>
