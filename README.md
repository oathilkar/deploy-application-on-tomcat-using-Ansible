# Deploy-application-on-tomcat-using-Ansible

### Summary

- **Project Structure**: Organize your project with inventories, playbooks, and roles.
- **Ansible Playbooks**: Define tasks to install and configure Tomcat and deploy the application.
- **Roles**: Use roles to encapsulate tasks and templates for Tomcat installation and application deployment.
- **Templates**: Use Jinja2 templates for configuring Tomcat (`server.xml.j2`).

This setup provides a scalable approach to deploying applications on Tomcat using Ansible. Customize the configurations and tasks as needed for your application and environment requirements.

Deploying an application on Tomcat using Ansible involves setting up a project structure with playbooks, roles, and configuration files. 
Below, I'll guide you through creating a project to deploy an application on Apache Tomcat using Ansible.

### Project Structure

Create the following directory structure for your Ansible project:

```
ansible-tomcat-deployment/
├── inventories/
│   └── production/
│       └── hosts.ini       # Inventory file
├── playbooks/
│   ├── tomcat-install.yml   # Tomcat installation playbook
│   └── deploy-app.yml       # Playbook to deploy the application
└── roles/
    ├── tomcat/
    │   ├── tasks/
    │   │   └── main.yml    # Tomcat role tasks
    │   ├── templates/
    │   │   └── server.xml.j2  # Tomcat server.xml template
    │   ├── vars/
    │   │   └── main.yml     # Tomcat role variables
    │   └── meta/
    │       └── main.yml     # Tomcat role metadata
    └── deploy-app/
        ├── tasks/
        │   └── main.yml    # Deploy application tasks
        ├── files/
        │   └── sample.war  # Sample application WAR file (replace with your app)
        ├── vars/
        │   └── main.yml    # Deploy application variables
        └── meta/
            └── main.yml    # Deploy application metadata
```

### Step-by-Step Guide

#### 1. Inventory File

Create an inventory file (`hosts.ini`) under `inventories/production/`:

```ini
[tomcat_servers]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11

[tomcat_servers:vars]
ansible_ssh_user=username
ansible_ssh_pass=password
```

Replace `username` and `password` with your SSH username and password for the Tomcat servers.

#### 2. Tomcat Installation Playbook

Create a playbook (`tomcat-install.yml`) under `playbooks/` to install and configure Tomcat:

```yaml
---
- name: Install and configure Tomcat
  hosts: tomcat_servers
  become: true  # Run tasks with root privileges

  roles:
    - tomcat
```

#### 3. Tomcat Role

Create an Ansible role for Tomcat under `roles/tomcat/`:

**`roles/tomcat/meta/main.yml`**:

```yaml
---
dependencies: []
```

**`roles/tomcat/tasks/main.yml`**:

```yaml
---
- name: Install OpenJDK
  apt:
    name: openjdk-8-jdk
    state: present
  tags: [tomcat]

- name: Install Tomcat
  apt:
    name: tomcat8
    state: present
  tags: [tomcat]

- name: Configure Tomcat server.xml
  template:
    src: server.xml.j2
    dest: /etc/tomcat8/server.xml
  notify: restart tomcat
  tags: [tomcat]

- name: Ensure Tomcat service is enabled and started
  systemd:
    name: tomcat8
    enabled: yes
    state: started
  tags: [tomcat]

handlers:
  - name: restart tomcat
    service:
      name: tomcat8
      state: restarted
```

**`roles/tomcat/templates/server.xml.j2`**:

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <Valve className="org.apache.catalina.valves.AccessLogValve"
               directory="logs" prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>
```

#### 4. Deploy Application Playbook

Create a playbook (`deploy-app.yml`) under `playbooks/` to deploy the application to Tomcat:

```yaml
---
- name: Deploy application to Tomcat
  hosts: tomcat_servers
  become: true  # Run tasks with root privileges

  roles:
    - deploy-app
```

#### 5. Deploy Application Role

Create an Ansible role for deploying the application under `roles/deploy-app/`:

**`roles/deploy-app/meta/main.yml`**:

```yaml
---
dependencies: []
```

**`roles/deploy-app/tasks/main.yml`**:

```yaml
---
- name: Deploy WAR file to Tomcat webapps directory
  copy:
    src: files/sample.war  # Replace with your WAR file
    dest: /var/lib/tomcat8/webapps/sample.war
    owner: tomcat8
    group: tomcat8
    mode: '0644'
  notify: restart tomcat
  tags: [deploy-app]

handlers:
  - name: restart tomcat
    service:
      name: tomcat8
      state: restarted
```

Make sure to replace `files/sample.war` with the path to your actual WAR file that you want to deploy.

#### 6. Running the Playbooks

To deploy Tomcat and then deploy your application, run the following commands:

```bash
# Deploy Tomcat
ansible-playbook -i inventories/production/hosts.ini playbooks/tomcat-install.yml

# Deploy Application
ansible-playbook -i inventories/production/hosts.ini playbooks/deploy-app.yml
```
This concludes project to deploy .war on tomcat.
