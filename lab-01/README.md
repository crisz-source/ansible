# Lab 01 - Conceitos, Instalação e Primeiro Playbook

## Sobre este Lab

Introdução ao Ansible: instalação, inventário, comandos ad-hoc e primeiro playbook.

## O que foi feito

- Instalação do Ansible
- Criação de inventário local
- Comandos ad-hoc (ping, shell, file, copy)
- Primeiro playbook com múltiplas tasks

## Estrutura de Arquivos

```
lab-01/
├── inventory.ini         # Lista de servidores
├── primeiro-playbook.yml # Primeiro playbook
└── README.md
```

## Conceitos que aprendi

**Ansible vs Terraform**

| Terraform | Ansible |
|-----------|---------|
| Cria infraestrutura | Configura o que roda dentro |
| HCL | YAML |
| State file | Sem state (idempotente por design) |
| Providers (AWS, Azure) | Módulos (apt, copy, docker) |

**Componentes do Ansible**

- Control Node: Máquina onde Ansible está instalado
- Managed Nodes: Servidores que serão configurados
- Inventory: Arquivo com lista de servidores
- Module: Unidade de trabalho (apt, file, copy, etc)
- Playbook: Arquivo YAML com tarefas a executar
- Task: Uma ação dentro do playbook

**Inventário básico**

```ini
[local]
localhost ansible_connection=local
```

- `[local]` - Nome do grupo
- `localhost` - Nome do servidor
- `ansible_connection=local` - Conexão direta sem SSH

**Comando ad-hoc**

```bash
ansible -i inventory.ini local -m ping
```

- `-i inventory.ini` - Arquivo de inventário
- `local` - Grupo de servidores
- `-m ping` - Módulo a executar

**Estrutura de playbook**

```yaml
---
- name: Descrição do playbook
  hosts: grupo_do_inventario
  tasks:
    - name: Descrição da tarefa
      modulo:
        parametro: valor
```

**Idempotência**

Rodar o mesmo playbook várias vezes não quebra nada. O Ansible verifica o estado atual antes de agir:

- Se já está como desejado → `ok` (não faz nada)
- Se precisa mudar → `changed` (executa a mudança)

## Comandos executados

```bash
# Instalar Ansible
sudo apt update
sudo apt install ansible -y
ansible --version

# Testar conexão
ansible -i inventory.ini local -m ping

# Comandos ad-hoc
ansible -i inventory.ini local -m shell -a "uname -a"
ansible -i inventory.ini local -m file -a "path=/tmp/teste.txt state=touch"
ansible -i inventory.ini local -m copy -a "content='texto' dest=/tmp/teste.txt"
ansible -i inventory.ini local -m file -a "path=/tmp/teste.txt state=absent"

# Executar playbook
ansible-playbook -i inventory.ini primeiro-playbook.yml
```

## PLAY RECAP explicado

```
localhost : ok=4  changed=2  unreachable=0  failed=0  skipped=0
```

- `ok` - Tarefas executadas com sucesso
- `changed` - Tarefas que mudaram algo
- `unreachable` - Servidores inacessíveis
- `failed` - Tarefas que falharam
- `skipped` - Tarefas puladas (por condição)

## Documentação consultada

- https://docs.ansible.com/ansible/latest/getting_started/index.html
- https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html