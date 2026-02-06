# Lab 03 - Variáveis, Facts, Handlers e Condicionais

## Sobre este Lab

Uso de variáveis para parametrizar playbooks, facts para coletar informações do sistema, handlers para executar tasks condicionalmente, e when para criar condições.

## O que foi feito

- Definir e usar variáveis no playbook
- Acessar facts do sistema (SO, memória, IP, hostname)
- Criar handlers que só rodam quando notificados
- Usar condicionais para executar tasks seletivamente

## Estrutura de Arquivos

```
lab-03/
├── inventory.ini       # Lista de servidores
├── variaveis.yml       # Playbook demonstrando variáveis
├── facts.yml           # Playbook demonstrando facts
├── handlers.yml        # Playbook demonstrando handlers
├── condicionais.yml    # Playbook demonstrando when
└── README.md
```

## Conceitos que aprendi

**Variáveis no playbook**

```yaml
vars:
  nginx_port: 8080
  pacotes:
    - vim
    - curl
  mensagem: "Texto aqui"

tasks:
  - name: Usar variável
    debug:
      msg: "Porta: {{ nginx_port }}"
```

- `vars:` - Bloco onde define variáveis
- `{{ variavel }}` - Sintaxe Jinja2 para usar variável
- Sempre entre aspas no YAML

**Facts (informações automáticas)**

O Ansible coleta automaticamente no "Gathering Facts":

| Fact | O que contém |
|------|--------------|
| `ansible_distribution` | Nome do SO (Ubuntu, CentOS) |
| `ansible_distribution_version` | Versão do SO |
| `ansible_architecture` | Arquitetura (x86_64) |
| `ansible_memtotal_mb` | Memória total em MB |
| `ansible_default_ipv4.address` | IP principal |
| `ansible_hostname` | Nome da máquina |

Para ver todos os facts:

```bash
ansible -i inventory.ini local -m setup
```

**Handlers**

Tasks que só executam quando notificadas por outra task que mudou algo.

```yaml
tasks:
  - name: Copiar configuração
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Reiniciar Nginx    # Chama handler se mudar

handlers:
  - name: Reiniciar Nginx
    service:
      name: nginx
      state: restarted
```

Características:
- Handler só roda se a task que notificou tiver `changed`
- Roda apenas UMA vez no final, mesmo se notificado várias vezes
- Evita reinícios desnecessários

**Condicionais (when)**

Executar task apenas se condição for verdadeira:

```yaml
- name: Só no Ubuntu
  apt:
    name: htop
    state: present
  when: ansible_distribution == "Ubuntu"

- name: Só se tiver muita RAM
  debug:
    msg: "Muita memória!"
  when: ansible_memtotal_mb > 16000

- name: Múltiplas condições
  debug:
    msg: "Ubuntu 64 bits"
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_architecture == "x86_64"
```

- Não precisa de `{{ }}` no when
- Lista de condições = AND (todas devem ser verdadeiras)
- Task com condição falsa → `skipping`

**Módulo debug**

Mostra mensagens na tela. Não faz nada no servidor.

```yaml
# Mostrar mensagem
- debug:
    msg: "Texto aqui"

# Mostrar variável
- debug:
    var: nome_da_variavel

# Mostrar fact
- debug:
    msg: "IP: {{ ansible_default_ipv4.address }}"
```

**Módulo file com state: link**

Cria link simbólico (atalho):

```yaml
- name: Criar symlink
  file:
    src: /etc/nginx/sites-available/site    # Original
    dest: /etc/nginx/sites-enabled/site     # Link
    state: link
```

## Problemas encontrados

**Sudo pedindo senha:**

Usar flag `-K` para pedir senha:

```bash
ansible-playbook -i inventory.ini playbook.yml -K
```

**Porta 8080 ocupada pelo Docker:**

Mudamos para porta 9090 na variável `nginx_port`.

## Comandos executados

```bash
# Playbook de variáveis
ansible-playbook -i inventory.ini variaveis.yml

# Playbook de facts
ansible-playbook -i inventory.ini facts.yml

# Playbook de handlers (precisa sudo)
ansible-playbook -i inventory.ini handlers.yml -K

# Playbook de condicionais
ansible-playbook -i inventory.ini condicionais.yml -K
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html