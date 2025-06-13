# Projeto-de-Infraestrutura-como-C-digo-IaC-com-Vagrant-e-Ansible
Este repositório contém um projeto de Infraestrutura como Código (IaC) utilizando Vagrant para provisionamento de máquinas virtuais e Ansible para automação da configuração. O objetivo principal é demonstrar a implantação automatizada de um servidor web Nginx e a personalização de uma página index.html usando templates.

## Requisitos

Para executar este projeto, você precisará ter o seguinte software instalado em sua máquina de acordo com o sistema operacional.

* **VirtualBox:** Um software de virtualização gratuito e de código aberto.
    * [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* **Vagrant:** Ferramenta para construir e gerenciar ambientes de desenvolvimento virtuais.
    * [Download Vagrant](https://www.vagrantup.com/downloads)
* **Visual Code:** IDE Para Desenvolvimento.
    * [Download VsCode](https://code.visualstudio.com)
* **Git:** Sistema de controle de versão.
    * [Download Git](https://git-scm.com/downloads)


## Estrutura do Projeto

```
.
├── .vagrant/                 # Diretório interno do Vagrant (gerado automaticamente)
├── files/                    # Contém arquivos estáticos para o servidor web (CSS, JS, imagens, etc.)
│   ├── assets/
│   ├── code files/
│   └── index.html            # Cópia do index.html original (será sobrescrita pelo template após o desafio)
├── templates/                # Contém templates Jinja2 para o Ansible
│   └── index.html.j2         # Template da página web principal (para o desafio opcional)
├── playbook.yml              # Playbook Ansible principal para provisionamento
├── Vagrantfile               # Configuração do Vagrant para a máquina virtual
└── README.md                 # Este arquivo
```

## Configuração e Execução

Siga os passos abaixo para configurar e executar o ambiente virtual e implantar a aplicação web inicial.

### Clonar o Repositório

Primeiro, clone este repositório para sua máquina local usando Git:

```bash
git clone https://github.com/skvanderson/Projeto-de-Infraestrutura-como-C-digo-IaC-com-Vagrant-e-Ansible.git
cd Projeto-de-Infraestrutura-como-C-digo-IaC-com-Vagrant-e-Ansible
```

### Iniciar a Vangrantfile + Máquina Virtual

Dentro do diretório raiz do projeto (onde o Vagrantfile está), execute o comando Vagrant para provisionar a máquina virtual. Este processo fará o download da imagem base (se necessário) e executará o playbook.yml do Ansible.

Obs: O VirutalBox precisa já está instalado nessa etapa.

```bash
vagrant init
```

```bash
code .
```

Apague todas as informações que já estiverem no arquivo Vagrantfile, atualize as informações conforme abaixo:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/focal64"

    config.vm.network "forwarded_port", guest: 80, host: 8080

    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
      vb.name = "nginx - webserver"
    end

    config.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "playbook.yml"
    end
end
```
O arquivo Vagrantfile é um arquivo de definição que instrui o Vagrant sobre como a máquina virtual deve ser criada e configurada, utilizando o Ansible para realizar essa configuração.

Agora é necessário a criação do arquivo playbook.yml(Neste reposítório já está criado)

```
---
- hosts: all
  become: yes
  tasks:
    - name: Atualiza o cache do apt
      apt:
        update_cache: yes
      tags:
        - packages

    - name: Instala o Nginx
      apt:
        name: nginx
        state: present
      tags:
        - packages

    - name: Copia a página web para o diretório do Nginx
      copy:
        src: files/
        dest: /var/www/html
        owner: www-data
        group: www-data
        mode: '0644'
      notify:
        - Reiniciar Nginx

  handlers:
    - name: Reiniciar Nginx
      service:
        name: nginx
        state: restarted
```
Agora vamos iniciar o provisionamento da Marquina Virtual

```
vagrant up
```
Para verificar se tudo ocorreu bem!

```
http://localhost:8080
```
Resultado Esperado.

![image](https://github.com/user-attachments/assets/dba92fea-eceb-4549-9c17-2507bed8b65f)




