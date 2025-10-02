# Projeto de Infraestrutura como Código (IaC) com Vagrant e Ansible

Este repositório contém um projeto de Infraestrutura como Código (IaC) utilizando Vagrant para provisionamento de máquinas virtuais e Ansible para automação da configuração. O objetivo é demonstrar a implantação automatizada de um servidor web Nginx servindo o site completo "Mundo Invertido" baseado em Stranger Things.

## 🎯 Objetivo

Criar uma infraestrutura completamente automatizada que permita:
- Provisionar uma máquina virtual Ubuntu com um único comando
- Configurar automaticamente um servidor web Nginx
- Implantar uma aplicação web estática completa
- Demonstrar boas práticas de Infraestrutura como Código
- Servir como base para projetos mais complexos

## 📋 Requisitos

1. **VirtualBox** - Software de virtualização
- [Download aqui](https://www.virtualbox.org/wiki/Downloads)
- Versão recomendada: 6.1 ou superior

2. **Vagrant** - Gerenciador de ambientes virtuais
- [Download aqui](https://www.vagrantup.com/downloads)
- Versão recomendada: 2.2 ou superior

3. **Git** - Controle de versão
- [Download aqui](https://git-scm.com/downloads)

4. **Editor de Texto (Opcional)** 
- Visual Studio Code: [Download aqui](https://code.visualstudio.com)
- Ou qualquer editor de sua preferência

```
virtualbox --version
```
```
vagrant --version
```
```
git --version
```   
   
## 🏗️ Estrutura do Projeto


```
.
│
├── .vagrant/                         # Diretório interno do Vagrant (NÃO MODIFICAR)
├── files/                            # Arquivos do site
│   ├── index.html                    # Página principal do Mundo Invertido
│   └── assets/                       # Recursos do site
│       ├── css/
│       │   └── style.css             # Estilos principais
│       ├── js/
│       │   ├── main.js               # JavaScript principal
│       │   ├── firebase/
│       │   │   └── app.js            # Configuração Firebase
│       │   └── hellfire-club.js      # Script do Clube Hellfire
│       ├── images/
│       │   ├── banner/
│       │   │   └── logo.svg          # Logo do site
│       │   └── content/
│       │       ├── invertedworld.png
│       │       ├── serieimage01.png
│       │       ├── serieimage02.png
│       │       └── serieimage03.png
│       └── musics/
│           ├── normal-world.mpeg
│           └── inverted-world.mpeg
├── Vagrantfile                       # Configuração da máquina virtual
├── playbook.yml                      # Script de automação Ansible
└── README.md                         # Este arquivo
```

## Configuração e Execução

Siga os passos abaixo para configurar e executar o ambiente virtual e implantar a aplicação web inicial.

### Clonar o Repositório

Primeiro, clone este repositório para sua máquina local usando Git:

### 1. Clonar o Repositório

```bash
git clone https://github.com/skvanderson/Projeto-de-Infraestrutura-como-C-digo-IaC-com-Vagrant-e-Ansible.git
cd Projeto-de-Infraestrutura-como-C-digo-IaC-com-Vagrant-e-Ansible
```

### 2. Verificar os Arquivos do Projeto

Antes de iniciar, verifique se você tem todos os arquivos necessários:

### Listar a estrutura de arquivos
```
ls -la
```
Deverá ver algo como: Vagrantfile  playbook.yml  files/  README.md

### 3. Entender os Arquivos de Configuração

### Vagrantfile

Este arquivo define como a máquina virtual será criada:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Define a imagem base do Ubuntu 20.04
  config.vm.box = "ubuntu/focal64"
  
  # Configura o redirecionamento de porta
  # A porta 80 da VM será acessível como 8080 no host
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  # Configurações do VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"        # 1GB de RAM
    vb.name = "nginx-webserver"  # Nome da VM
  end
  
  # Provisionamento com Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"  # Script de configuração
    ansible.verbose = "v"              # Modo verboso para logs detalhados
  end
end
```
### playbook.yml
Este arquivo automatiza a configuração do servidor:

```
---
- hosts: all
  become: yes  # Executa comandos como superusuário
  
  tasks:
    # Atualiza a lista de pacotes disponíveis
    - name: Atualiza o cache do apt
      apt:
        update_cache: yes
        cache_valid_time: 3600

    # Instala o servidor web Nginx
    - name: Instala o Nginx
      apt:
        name: nginx
        state: present

    # Remove a página padrão do Nginx
    - name: Remove página padrão do Nginx
      file:
        path: /var/www/html/index.nginx-debian.html
        state: absent

    # Copia todos os arquivos do site para o servidor
    - name: Copia TODO o conteúdo web para o diretório do Nginx
      ansible.builtin.copy:
        src: files/                    # Origem: pasta files/ local
        dest: /var/www/html/           # Destino: diretório web do Nginx
        owner: www-data                # Proprietário: usuário do Nginx
        group: www-data                # Grupo: grupo do Nginx
        mode: '0644'                   # Permissões: leitura para todos
        force: yes                     # Sobrescreve arquivos existentes

    # Garante que o Nginx está rodando
    - name: Habilita e inicia o serviço Nginx
      systemd:
        name: nginx
        enabled: yes      # Inicia automaticamente no boot
        state: started    # Inicia o serviço agora

    # Testa se o site está funcionando
    - name: Verifica se o Nginx está rodando
      uri:
        url: http://localhost
        return_content: yes
      register: nginx_status
      failed_when: nginx_status.status != 200
```
### 4: Iniciar o Provisionamento

Agora execute o comando principal:
```
vagrant up
```
Para verificar se tudo ocorreu bem!
Durante o processo, você verá logs como:

```
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
PLAY [all] *********************************************************************
TASK [Gathering Facts] *********************************************************
ok: [default]
TASK [Atualiza o cache do apt] *************************************************
changed: [default]
TASK [Instala o Nginx] *********************************************************
changed: [default]
...
PLAY RECAP *********************************************************************
default: ok=7 changed=4 unreachable=0 failed=0

```

```
O que acontece quando você executa vagrant up:

📥 Download da imagem Ubuntu (apenas na primeira vez)
🖥️ Criação da VM no VirtualBox
🔧 Configuração de rede e port forwarding
⚙️ Execução do Ansible que irá:

Atualizar o sistema
Instalar o Nginx
Configurar o servidor web
Copiar os arquivos do site
Iniciar o serviço

Tempo estimado: 5-10 minutos na primeira execução.

http://localhost:8080
```

### Passo 5: Verificar se Deu Certo

Se você ver failed=0 no final, significa que tudo deu certo! ✅
Abra seu navegador e visite:

```
http://localhost:8080
```
Resultado Esperado.
Você deve ver o site "Mundo Invertido" completo, com:

🎨 Design temático de Stranger Things

- 🎵 Música de fundo
- 🖼️ Galeria de imagens
- 📝 Formulário de inscrição
- 🌙 Botão para alternar entre temas claro/escuro

![image](https://github.com/user-attachments/assets/dba92fea-eceb-4549-9c17-2507bed8b65f)

### Comandos Úteis para Gerenciamento

Comandos Básicos do Vagrant
- Verificar se o Nginx está rodando na VM
```
vagrant ssh -- systemctl status nginx
```
- Parar a VM
```
vagrant halt
```
- Reiniciar a VM
```
vagrant reload
```
- Acessar a VM via SSH
```
vagrant ssh
```
- Ver status da VM
```
vagrant status
```

## Destruir a VM (CUIDADO: remove tudo)
```
vagrant destroy
```
Apenas executar o provisionamento novamente
```
vagrant provision
```

### Comandos de Verificação

Verificar se o Nginx está rodando na VM
```
vagrant ssh -- systemctl status nginx
```
Testar o site via linha de comando
```
vagrant ssh -- curl http://localhost
```
Verificar os arquivos copiados
```
vagrant ssh -- ls -la /var/www/html/
```
Ver logs de erro do Nginx
```
vagrant ssh -- tail -f /var/log/nginx/error.log
```

### Personalização e Modificações

Modificando o Site

### 1. Edite os arquivos na pasta files/

- files/index.html - Conteúdo principal
- files/assets/css/style.css - Estilos
- files/assets/js/ - Scripts JavaScript

### 2. Reprovisione a VM:

```
vagrant provision
```

### 3. Como Alterar a Configuração do Servidor

Modificar porta: Edite o Vagrantfile:
```
config.vm.network "forwarded_port", guest: 80, host: 8080  # Mude 8080 para outra porta
```
Aumentar memória: Edite o Vagrantfile:
```
vb.memory = "2048"  # Mude para 2GB
```

## Adicionar Novos Pacotes
Edite o playbook.yml e adicione novas tasks:

```
- name: Instala um novo pacote
  apt:
    name: nome-do-pacote
    state: present
```
### Possiveis Problemas

Porta 8080 já está em uso

Solução
```
# Encontrar qual processo está usando a porta
sudo lsof -i :8080

# Ou mudar a porta no Vagrantfile para 8081
config.vm.network "forwarded_port", guest: 80, host: 8081
```

Erro no provisionamento do Ansible

Solução
```
# Ver logs detalhados
vagrant provision --debug

# Ou destruir e recriar
vagrant destroy -f
vagrant up
```

VM não inicia

# Verificar se o VirtualBox está instalado
virtualbox --version

Solução
```
# Verificar status
vagrant status

# Recriar a VM
vagrant destroy -f
vagrant up
```

Site não carrega CSS/JS

```
# Verificar se os arquivos foram copiados
vagrant ssh -- ls -la /var/www/html/assets/

# Verificar permissões
vagrant ssh -- ls -la /var/www/html/

# Reprovisionar
vagrant provision
```


📝 Licença

Este projeto é para fins educacionais e demonstração de conceitos de Infraestrutura como Código.

Desenvolvido com Vagrant + Ansible + Nginx
