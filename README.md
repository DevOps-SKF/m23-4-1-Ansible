# 23.4.1 Ansible Playbook

## Задача 

1. Написать свой Playbook, который будет устанавливать и запускать Docker на локальной машине.  
2. Написать свой Playbook, который будет запускать MySQL-сервер на локальной машине
3. написать свой Playbook, который будет выполнять установку и запуск веб-сервера nginx.  
Измените файл /usr/share/nginx/html/index.html, чтобы при обращении к веб-серверу он выдавал любой факт о машине, на которой запущен   (например, ее hostname или общее количество оперативной памяти). 

## Решение

### 1 - Установка Docker

Изначально все было в Vagrantfile, сразу же устанавливающий и Kubernetes.  
Переношу в Ansible, оставляя containerd и прочие настройки, чтобы при необходимости поверх можно было поставить Kubernetes.  

    export DEBIAN_FRONTEND=noninteractive
    sudo apt-get -qq update
    sudo apt-get -y remove docker docker-engine docker.io containerd runc
    sudo apt-get autoremove -y

    sudo apt-get -qq install -y apt-transport-https ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo \
        "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get -qq update
    sudo apt-get -qq install -y docker-ce=#{DOCKERVER} docker-ce-cli=#{DOCKERVER} containerd.io
    
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    
    sed '/\[plugins.\"io.containerd.grpc.v1.cri\".containerd.runtimes.runc.options\]/a\            SystemdCgroup = true' /etc/containerd/config.toml
    
    cat <<EOF | sudo tee /etc/docker/daemon.json
    {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
    }
    EOF
    sudo systemctl restart containerd
    sudo systemctl enable docker.service --now
    sudo usermod -aG docker $(whoami)

Запуск:  
`ansible-playbook -i hosts docker.yml`  

Затем, если нужен и Kubernetes,  
`ansible-playbook -i hosts k8s.yml`

### 2 - Установка mysql

Запуск:  
`ansible-playbook -i hosts mysql.yml`  

На будущее создается дополнительный пользователь для доступа с любого хоста по паролю. Но bind_address остается localhost  
root подключается без ввода пароля благодаря /root/.my.cnf  
Сгенерированные пароли выводятся в процессе выполнения. Полезно их сохранить :)  

    mysql> select host,user,authentication_string,plugin from mysql.user;
    +-----------+------------------+------------------------------------+-----------------------+
    | host      | user             | authentication_string              | plugin                |
    +-----------+------------------+------------------------------------+-----------------------+
    | %         | anton            | *303F6D0AEF83E461186B279DC7D431B8D | mysql_native_password |
    | localhost | debian-sys-maint | $A$005$4<V|Q?J=}Ud*-WiuU28ziHUxS8F | caching_sha2_password |
    | localhost | mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALID | caching_sha2_password |
    | localhost | mysql.session    | $A$005$THISISACOMBINATIONOFINVALID | caching_sha2_password |
    | localhost | mysql.sys        | $A$005$THISISACOMBINATIONOFINVALID | caching_sha2_password |
    | localhost | root             |                                    | auth_socket           |
    +-----------+------------------+------------------------------------+-----------------------+
    6 rows in set (0.00 sec)

