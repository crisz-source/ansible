# Lab 05 - Roles

## Sobre este Lab

Organização de código Ansible em roles reutilizáveis. Equivalente aos módulos do Terraform.

## O que foi feito

- Criar estrutura de role para webserver
- Definir valores padrão em defaults
- Criar tasks, handlers e templates na role
- Usar a role em playbook
- Sobrescrever variáveis ao chamar a role

## Estrutura de Arquivos

```
lab-05/
├── inventory.ini
├── site.yml              # Playbook usando role com defaults
├── site-custom.yml       # Playbook sobrescrevendo variáveis
└── roles/
    └── webserver/
        ├── defaults/
        │   └── main.yml  # Valores padrão
        ├── tasks/
        │   └── main.yml  # Tasks principais
        ├── handlers/
        │   └── main.yml  # Handlers
        └── templates/
            ├── nginx.conf.j2
            └── index.html.j2
```

## Conceitos que aprendi

**O que é Role**

Forma de organizar código Ansible por responsabilidade. Cada role é independente e reutilizável.

```
roles/
├── nginx/      → Tudo de Nginx
├── docker/     → Tudo de Docker
└── app/        → Tudo da aplicação
```

**Estrutura de uma Role**

```
roles/nome_da_role/
├── tasks/main.yml      # Tasks (obrigatório)
├── handlers/main.yml   # Handlers
├── templates/          # Templates Jinja2
├── files/              # Arquivos estáticos
├── vars/main.yml       # Variáveis fixas (alta prioridade)
├── defaults/main.yml   # Valores padrão (baixa prioridade)
└── meta/main.yml       # Metadados e dependências
```

O mínimo necessário é `tasks/main.yml`.

**defaults vs vars**

| Pasta | Prioridade | Pode sobrescrever? |
|-------|------------|-------------------|
| `defaults/` | Baixa | Sim, facilmente |
| `vars/` | Alta | Difícil, precisa de extra-vars |

Use `defaults/` para valores que o usuário pode querer mudar.
Use `vars/` para valores internos que não devem mudar.

**Usar role no playbook**

```yaml
---
- name: Configurar servidor
  hosts: webservers
  become: yes
  roles:
    - webserver
```

**Sobrescrever variáveis**

```yaml
---
- name: Configurar servidor customizado
  hosts: webservers
  become: yes
  roles:
    - role: webserver
      vars:
        http_port: 9090
        server_name: meusite.local
```

**Prefixo nas tasks**

Quando usa role, as tasks aparecem com prefixo:

```
TASK [webserver : Instalar Nginx]
TASK [webserver : Copiar configuracao]
```

Facilita identificar de qual role veio cada task.

## Comparação com Terraform

| Terraform | Ansible |
|-----------|---------|
| Module | Role |
| `variables.tf` | `defaults/main.yml` |
| `main.tf` | `tasks/main.yml` |
| `source = "./modules/x"` | `roles: - x` |

## Comandos executados

```bash
# Estrutura da role
mkdir -p roles/webserver/{tasks,handlers,templates,defaults}

# Executar com defaults
ansible-playbook -i inventory.ini site.yml -K
curl localhost

# Executar com variáveis customizadas
ansible-playbook -i inventory.ini site-custom.yml -K
curl localhost:9090
```

## Resultado

**Com defaults (porta 80):**
```html
<title>localhost</title>
<p>Porta: 80</p>
```

**Com vars customizadas (porta 9090):**
```html
<title>meusite.local</title>
<p>Porta: 9090</p>
```

Mesma role, configurações diferentes.

## Documentação consultada

- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html