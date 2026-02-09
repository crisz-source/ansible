# Lab 07 - Error Handling

## Sobre este Lab

Tratamento de erros em playbooks com block, rescue e always. Similar ao try/except/finally do Python.

## O que foi feito

- Demonstrar fluxo de erro com block/rescue/always
- Exemplo prático de backup e restore automático

## Estrutura de Arquivos

```
lab-07/
├── inventory.ini
├── error-handling.yml    # Demonstração básica
├── backup-restore.yml    # Exemplo prático
└── README.md
```

## Conceitos que aprendi

**Block, Rescue, Always**

```yaml
tasks:
  - name: Operação com tratamento de erro
    block:
      - name: Task 1
        # Se falhar, pula pro rescue
      - name: Task 2
        # Continua se Task 1 ok

    rescue:
      - name: Executado se block falhar
        # Cleanup, restaurar backup, notificar

    always:
      - name: Sempre executa
        # Log, finalização, independente do resultado
```

| Bloco | Quando executa |
|-------|----------------|
| `block` | Sempre (é o fluxo principal) |
| `rescue` | Apenas se algo no block falhar |
| `always` | Sempre, sucesso ou falha |

**Comparação com outras linguagens**

| Ansible | Python | JavaScript |
|---------|--------|------------|
| block | try | try |
| rescue | except | catch |
| always | finally | finally |

**Exemplo prático - Backup antes de modificar**

```yaml
block:
  - name: Fazer backup
    copy:
      src: "{{ config_file }}"
      dest: "{{ config_file }}.bak"
      remote_src: yes

  - name: Modificar config
    template:
      src: novo.conf.j2
      dest: "{{ config_file }}"

  - name: Testar config
    command: nginx -t

rescue:
  - name: Restaurar backup
    copy:
      src: "{{ config_file }}.bak"
      dest: "{{ config_file }}"
      remote_src: yes

  - name: Notificar falha
    debug:
      msg: "Falha! Backup restaurado."

always:
  - name: Reiniciar serviço
    service:
      name: nginx
      state: restarted
```

**remote_src: yes**

Indica que o arquivo fonte está no servidor remoto, não na máquina local onde o Ansible está rodando.

```yaml
copy:
  src: /tmp/arquivo.bak     # Arquivo no servidor remoto
  dest: /tmp/arquivo        # Destino no servidor remoto
  remote_src: yes           # Ambos estão no remoto
```

**PLAY RECAP com rescue**

```
localhost : ok=7  changed=4  unreachable=0  failed=0  skipped=0  rescued=1
```

- `rescued=1` indica que o rescue tratou 1 erro

## Casos de uso reais

| Cenário | Block | Rescue | Always |
|---------|-------|--------|--------|
| Deploy de app | Baixar, instalar, testar | Rollback versão anterior | Limpar arquivos temporários |
| Config de serviço | Backup, modificar, validar | Restaurar backup | Reiniciar serviço |
| Banco de dados | Iniciar transação, modificar | Rollback transação | Fechar conexão |
| Atualização de pacotes | Atualizar, testar | Reverter pacote | Enviar log |

## Comandos executados

```bash
# Demonstração básica
ansible-playbook -i inventory.ini error-handling.yml

# Exemplo prático
ansible-playbook -i inventory.ini backup-restore.yml
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_error_handling.html