# Lab 06 - Ansible Vault (Secrets)

## Sobre este Lab

Criptografar dados sensíveis (senhas, chaves, tokens) com Ansible Vault. Essencial para DevSecOps.

## O que foi feito

- Criar arquivo criptografado com secrets
- Visualizar e editar arquivo criptografado
- Usar secrets em playbook
- Automatizar com arquivo de senha

## Estrutura de Arquivos

```
lab-06/
├── inventory.ini
├── secrets.yml           # Arquivo criptografado
├── usar-secrets.yml      # Playbook que usa os secrets
└── README.md
```

## Conceitos que aprendi

**O que é Vault**

Ferramenta do Ansible para criptografar dados sensíveis. O arquivo fica criptografado no repositório, só quem tem a senha consegue ler.

```
Sem Vault:  db_password: "minhasenha123"     → Qualquer um vê
Com Vault:  $ANSIBLE_VAULT;1.1;AES256...     → Criptografado
```

**Comandos do Vault**

| Comando | O que faz |
|---------|-----------|
| `ansible-vault create arquivo.yml` | Cria arquivo criptografado |
| `ansible-vault view arquivo.yml` | Visualiza conteúdo |
| `ansible-vault edit arquivo.yml` | Edita arquivo |
| `ansible-vault encrypt arquivo.yml` | Criptografa arquivo existente |
| `ansible-vault decrypt arquivo.yml` | Descriptografa arquivo |
| `ansible-vault rekey arquivo.yml` | Muda a senha |

**Usar secrets no playbook**

```yaml
---
- name: Usar secrets
  hosts: local
  vars_files:
    - secrets.yml    # Arquivo criptografado

  tasks:
    - name: Usar variável do vault
      debug:
        msg: "Usuario: {{ db_user }}"
```

**Executar com senha interativa**

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

**Executar com arquivo de senha**

```bash
# Criar arquivo de senha
echo "minhasenha" > ~/.vault_pass
chmod 600 ~/.vault_pass

# Executar
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

**Filtros Jinja2 úteis com secrets**

```yaml
# Mostrar tamanho sem revelar conteúdo
msg: "Senha tem {{ db_password | length }} caracteres"

# Mostrar apenas início
msg: "API Key: {{ api_key[:6] }}..."
```

**Permissões de arquivo**

```yaml
- name: Criar config com permissões restritas
  copy:
    content: |
      DB_PASS={{ db_password }}
    dest: /tmp/config.env
    mode: '0600'    # Só dono lê/escreve
```

## Boas práticas

| Faça | Não faça |
|------|----------|
| Versionar secrets.yml (criptografado) | Versionar .vault_pass |
| Adicionar .vault_pass no .gitignore | Commitar senhas em texto plano |
| Usar chmod 600 no arquivo de senha | Deixar senha com permissões abertas |
| Usar variáveis de ambiente no CI/CD | Hardcodar senhas no playbook |

**.gitignore recomendado:**

```
.vault_pass
*.retry
```

## Uso em CI/CD

No GitHub Actions, GitLab CI, etc:

1. Guarde a senha do vault como secret do CI
2. No pipeline, crie o arquivo temporário:

```yaml
# GitHub Actions exemplo
- name: Create vault password file
  run: echo "${{ secrets.VAULT_PASS }}" > .vault_pass

- name: Run playbook
  run: ansible-playbook site.yml --vault-password-file .vault_pass

- name: Cleanup
  run: rm .vault_pass
```

## Comandos executados

```bash
# Criar arquivo criptografado
ansible-vault create secrets.yml

# Ver conteúdo
ansible-vault view secrets.yml

# Executar pedindo senha
ansible-playbook -i inventory.ini usar-secrets.yml --ask-vault-pass

# Executar com arquivo de senha
ansible-playbook -i inventory.ini usar-secrets.yml --vault-password-file ~/.vault_pass
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/vault_guide/index.html
- https://docs.ansible.com/ansible/latest/cli/ansible-vault.html