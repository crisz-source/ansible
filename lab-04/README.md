# Lab 04 - Templates e Loops

## Sobre este Lab

Uso de templates Jinja2 para gerar arquivos dinâmicos e loops para repetir tasks com valores diferentes.

## O que foi feito

- Criar template HTML com variáveis Jinja2
- Gerar página dinâmica com informações do servidor
- Instalar múltiplos pacotes com loop
- Criar múltiplos diretórios com loop
- Iterar sobre lista de dicionários

## Estrutura de Arquivos

```
lab-04/
├── inventory.ini           # Lista de servidores
├── template.yml            # Playbook demonstrando templates
├── loops.yml               # Playbook demonstrando loops
├── templates/
│   └── index.html.j2       # Template HTML
└── README.md
```

## Conceitos que aprendi

**Jinja2**

Linguagem de template usada pelo Ansible para processar variáveis em arquivos.

| Sintaxe | O que faz | Exemplo |
|---------|-----------|---------|
| `{{ }}` | Imprime variável | `{{ nome }}` |
| `{% %}` | Lógica (if, for) | `{% if x %}...{% endif %}` |
| `{# #}` | Comentário | `{# ignorado #}` |

**Módulo template vs copy**

| Módulo | O que faz |
|--------|-----------|
| `copy` | Copia arquivo como está |
| `template` | Processa variáveis Jinja2 antes de copiar |

```yaml
- name: Gerar arquivo a partir de template
  template:
    src: templates/config.j2    # Template local
    dest: /etc/app/config       # Destino no servidor
    owner: root
    group: root
    mode: '0644'
```

**Template com variáveis**

Arquivo `templates/index.html.j2`:

```html
<h1>{{ titulo }}</h1>
<p>Servidor: {{ ansible_hostname }}</p>
<p>IP: {{ ansible_default_ipv4.address }}</p>
```

O Ansible substitui `{{ variavel }}` pelo valor real.

**Loops básicos**

Repetir task para cada item de uma lista:

```yaml
vars:
  pacotes:
    - vim
    - curl
    - wget

tasks:
  - name: Instalar pacotes
    apt:
      name: "{{ item }}"
      state: present
    loop: "{{ pacotes }}"
```

- `loop:` - Lista de itens
- `{{ item }}` - Item atual da iteração

**Loop com lista inline**

```yaml
- name: Criar diretorios
  file:
    path: "/tmp/{{ item }}"
    state: directory
  loop:
    - app
    - logs
    - config
```

**Loop com lista de dicionários**

```yaml
vars:
  usuarios:
    - nome: dev1
      shell: /bin/bash
    - nome: dev2
      shell: /bin/zsh

tasks:
  - name: Mostrar usuarios
    debug:
      msg: "{{ item.nome }} usa {{ item.shell }}"
    loop: "{{ usuarios }}"
```

- `{{ item.nome }}` - Acessa atributo do dicionário

**Condicionais em templates**

```jinja2
{% if ambiente == "prod" %}
DEBUG=false
{% else %}
DEBUG=true
{% endif %}
```

**Loops em templates**

```jinja2
Servidores:
{% for server in servidores %}
- {{ server }}
{% endfor %}
```

## Observação: Cowsay

Se o pacote `cowsay` estiver instalado, o Ansible usa ele na saída. Para desativar:

```bash
ANSIBLE_NOCOWS=1 ansible-playbook playbook.yml
```

## Comandos executados

```bash
# Template
ansible-playbook -i inventory.ini template.yml -K
curl localhost/info.html

# Loops
ansible-playbook -i inventory.ini loops.yml -K
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html
- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html
- https://jinja.palletsprojects.com/en/3.1.x/templates/