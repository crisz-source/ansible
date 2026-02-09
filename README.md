# Ansible - Configuração do Servidor

Playbook que configura a VM criada pelo Terraform, instalando Docker e rodando a aplicação.

## O que faz

1. Atualiza pacotes do sistema
2. Instala Docker CE
3. Instala SDK Docker para Python
4. Copia página HTML customizada
5. Sobe container Nginx na porta 80

## Arquivos

| Arquivo | Descrição |
|---------|-----------|
| `inventory.ini` | Gerado automaticamente pelo Terraform com IP da VM |
| `playbook.yml` | Tasks de configuração do servidor |
| `templates/index.html.j2` | Página da aplicação com variáveis dinâmicas |

## Pré-requisitos

- Ansible >= 2.10
- Infraestrutura criada pelo Terraform (VM acessível via SSH)
- Chave SSH em `../terraform/ssh_key`

## Como Executar

```bash
ansible-playbook -i inventory.ini playbook.yml
```

## Variáveis

Definidas no playbook:

| Variável | Valor | Descrição |
|----------|-------|-----------|
| `app_name` | minha-app | Nome do container |
| `app_port` | 80 | Porta exposta |

## Módulos Utilizados

- `apt` - Instalar pacotes
- `apt_key` - Adicionar chave GPG
- `apt_repository` - Adicionar repositório
- `service` - Gerenciar serviços
- `file` - Criar diretórios
- `template` - Copiar arquivos com variáveis
- `docker_container` - Gerenciar containers

## Resultado

Container Nginx rodando com página customizada exibindo:

- Hostname da VM
- IP interno
- Sistema operacional
- Data do deploy
- Stack utilizada
