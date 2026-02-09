# Lab 08 - Tags, Limits e Estratégias

## Sobre este Lab

Executar partes específicas de um playbook com tags e limitar execução a servidores específicos.

## O que foi feito

- Criar playbook com múltiplas tags
- Testar execução seletiva com --tags e --skip-tags
- Testar tags especiais: always e never
- Limitar execução a grupos e hosts específicos

## Estrutura de Arquivos

```
lab-08/
├── inventory.ini
├── inventory-multi.ini   # Inventário com múltiplos hosts
├── tags.yml              # Playbook com tags
├── limit.yml             # Playbook para testar limit
└── README.md
```

## Conceitos que aprendi

**Tags em tasks**

```yaml
tasks:
  - name: Setup inicial
    file:
      path: /tmp/app
      state: directory
    tags:
      - setup
      - config

  - name: Deploy
    debug:
      msg: "Deploying..."
    tags:
      - deploy
```

**Tags especiais**

| Tag | Comportamento |
|-----|---------------|
| `always` | Sempre executa, mesmo com --tags |
| `never` | Nunca executa, a menos que chamada explicitamente |

```yaml
- name: Setup obrigatório
  file:
    path: /tmp/app
    state: directory
  tags:
    - setup
    - always    # Sempre roda

- name: Cleanup perigoso
  file:
    path: /tmp/app
    state: absent
  tags:
    - cleanup
    - never     # Só roda se chamar --tags cleanup
```

**Flags de tags**

| Flag | O que faz | Exemplo |
|------|-----------|---------|
| `--tags` | Executa só essas tags | `--tags "setup,deploy"` |
| `--skip-tags` | Executa tudo exceto essas | `--skip-tags test` |
| `--list-tags` | Lista tags do playbook | `--list-tags` |

**Limit - limitar hosts**

```bash
# Todos os hosts
ansible-playbook -i inventory.ini site.yml

# Só grupo webservers
ansible-playbook -i inventory.ini site.yml --limit webservers

# Só host específico
ansible-playbook -i inventory.ini site.yml --limit web1

# Múltiplos hosts
ansible-playbook -i inventory.ini site.yml --limit "web1,db1"

# Excluir host
ansible-playbook -i inventory.ini site.yml --limit 'all:!db1'
```

**Inventário com grupos**

```ini
[webservers]
web1 ansible_host=10.0.0.1
web2 ansible_host=10.0.0.2

[dbservers]
db1 ansible_host=10.0.0.3

[all:vars]
ansible_user=admin
```

**inventory_hostname**

Variável especial que contém o nome do host atual:

```yaml
- name: Mostrar host
  debug:
    msg: "Executando em: {{ inventory_hostname }}"
```

## Casos de uso

| Cenário | Comando |
|---------|---------|
| Só deploy sem testes | `--tags deploy` |
| Rodar tudo menos cleanup | `--skip-tags cleanup` |
| Testar em um host antes | `--limit web1` |
| Deploy só em webservers | `--limit webservers --tags deploy` |

## Comandos executados

```bash
# Executar tudo
ansible-playbook -i inventory.ini tags.yml

# Só setup e config
ansible-playbook -i inventory.ini tags.yml --tags "setup,config"

# Só deploy
ansible-playbook -i inventory.ini tags.yml --tags deploy

# Pular test
ansible-playbook -i inventory.ini tags.yml --skip-tags test

# Executar cleanup (tag never)
ansible-playbook -i inventory.ini tags.yml --tags cleanup

# Limit por grupo
ansible-playbook -i inventory-multi.ini limit.yml --limit webservers

# Limit por host
ansible-playbook -i inventory-multi.ini limit.yml --limit web1
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html
- https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html