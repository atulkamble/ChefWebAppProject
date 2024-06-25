# ChefWebAppProject
Here’s a detailed advanced Chef DevOps project, complete with code snippets and commands to help you automate the deployment and configuration of a web application stack using Chef.

### Project: Automated Deployment and Configuration with Chef

#### Components:
1. Chef Server Setup
2. Workstation Configuration
3. Node Configuration
4. Cookbooks and Recipes
5. Testing with Test Kitchen
6. Integration with CI/CD Tools
7. Monitoring and Reporting

### 1. Chef Server Setup
- **Install Chef Server:**
  Follow the official Chef Server installation guide:
  ```bash
  sudo apt-get update
  sudo apt-get install -y chef-server-core
  sudo chef-server-ctl reconfigure
  ```
- **Create an Admin User and Organization:**
  ```bash
  sudo chef-server-ctl user-create USER_NAME FIRST_NAME LAST_NAME EMAIL 'PASSWORD' --filename /path/to/USER_NAME.pem
  sudo chef-server-ctl org-create ORG_NAME "ORG_FULL_NAME" --association_user USER_NAME --filename /path/to/ORG_NAME-validator.pem
  ```

### 2. Workstation Configuration
- **Install ChefDK:**
  ```bash
  wget https://packages.chef.io/files/stable/chefdk/4.13.3/ubuntu/18.04/chefdk_4.13.3-1_amd64.deb
  sudo dpkg -i chefdk_4.13.3-1_amd64.deb
  ```
- **Configure Knife:**
  ```bash
  knife configure
  ```
  This will prompt you to provide the Chef Server URL, organization, and user details.

### 3. Node Configuration
- **Bootstrap Nodes:**
  ```bash
  knife bootstrap NODE_IP -x USER -P PASSWORD --sudo --node-name NODE_NAME
  ```

### 4. Cookbooks and Recipes

#### Nginx Cookbook
- **Generate Cookbook:**
  ```bash
  chef generate cookbook nginx
  ```
- **nginx/recipes/default.rb:**
  ```ruby
  package 'nginx' do
    action :install
  end

  service 'nginx' do
    action [:enable, :start]
  end

  template '/etc/nginx/sites-available/default' do
    source 'default.erb'
    notifies :reload, 'service[nginx]', :delayed
  end
  ```
- **nginx/templates/default/default.erb:**
  ```erb
  server {
      listen 80 default_server;
      listen [::]:80 default_server;

      server_name _;

      location / {
          proxy_pass http://localhost:3000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```
- **Upload Cookbook:**
  ```bash
  knife cookbook upload nginx
  ```

#### Application Cookbook
- **Generate Cookbook:**
  ```bash
  chef generate cookbook myapp
  ```
- **myapp/recipes/default.rb:**
  ```ruby
  package 'git'

  git '/srv/myapp' do
    repository 'https://github.com/your-repo/your-app.git'
    revision 'master'
    action :sync
  end

  execute 'install_dependencies' do
    command 'npm install'
    cwd '/srv/myapp'
    action :run
  end

  execute 'start_app' do
    command 'npm start &'
    cwd '/srv/myapp'
    action :run
  end
  ```
- **Upload Cookbook:**
  ```bash
  knife cookbook upload myapp
  ```

#### Database Cookbook
- **Generate Cookbook:**
  ```bash
  chef generate cookbook mysql
  ```
- **mysql/recipes/default.rb:**
  ```ruby
  package 'mysql-server' do
    action :install
  end

  service 'mysql' do
    action [:enable, :start]
  end

  execute 'create_database' do
    command 'mysql -e "CREATE DATABASE myapp;"'
    not_if 'mysql -e "SHOW DATABASES LIKE \'myapp\';"'
  end
  ```
- **Upload Cookbook:**
  ```bash
  knife cookbook upload mysql
  ```

### 5. Testing with Test Kitchen
- **Install Test Kitchen:**
  ```bash
  chef gem install kitchen
  chef gem install kitchen-vagrant
  chef gem install kitchen-inspec
  ```
- **.kitchen.yml Example:**
  ```yaml
  ---
  driver:
    name: vagrant

  provisioner:
    name: chef_zero

  platforms:
    - name: ubuntu-18.04

  suites:
    - name: default
      run_list:
        - recipe[nginx::default]
        - recipe[myapp::default]
        - recipe[mysql::default]
      attributes:
  ```
- **Run Test Kitchen:**
  ```bash
  kitchen converge
  kitchen verify
  kitchen destroy
  ```

### 6. Integration with CI/CD Tools

#### Jenkins Pipeline
- **Jenkinsfile Example:**
  ```groovy
  pipeline {
      agent any

      stages {
          stage('Checkout') {
              steps {
                  git 'https://github.com/your-repo/your-cookbook.git'
              }
          }
          stage('Test') {
              steps {
                  sh 'kitchen test'
              }
          }
          stage('Deploy') {
              steps {
                  sh 'knife cookbook upload your-cookbook'
                  sh 'knife ssh "name:NODE_NAME" "sudo chef-client"'
              }
          }
      }
  }
  ```

### 7. Monitoring and Reporting
- **Install and Configure Chef Automate:**
  Follow the [Chef Automate installation guide](https://docs.chef.io/automate/install/).
- **Configure Reporting:**
  ```bash
  chef-server-ctl enable-reporting
  ```
- **Set Up Node Monitoring:**
  Use tools like InSpec to write compliance profiles and run them on nodes.

This comprehensive project ensures that your infrastructure is automated, tested, and integrated with a CI/CD pipeline, while also being monitored and compliant.
