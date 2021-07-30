# 23.4.1 Ansible Playbook

## Задача 

1. Написать свой Playbook, который будет устанавливать и запускать Docker на локальной машине.  
2. Написать свой Playbook, который будет запускать MySQL-сервер на локальной машине
3. написать свой Playbook, который будет выполнять установку и запуск веб-сервера nginx.  
Измените файл /usr/share/nginx/html/index.html, чтобы при обращении к веб-серверу он выдавал любой факт о машине, на которой запущен   (например, ее hostname или общее количество оперативной памяти). Зайдите в веб-браузере на эту машину и проверьте результат работы.

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

