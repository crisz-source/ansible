# Lab 02 - Playbooks Básicos

## Sobre este Lab

Playbooks para instalar pacotes, gerenciar serviços e copiar arquivos.

## O que foi feito

- Instalar pacote htop com apt
- Instalar e gerenciar serviço Nginx
- Copiar página HTML customizada
- Verificar acesso via módulo uri

## Estrutura de Arquivos

```
lab-02/
├── inventory.ini         # Lista de servidores
├── instalar-pacote.yml   # Playbook para instalar htop
├── gerenciar-servico.yml # Playbook para Nginx
├── copiar-arquivo.yml    # Playbook para copiar HTML
├── index.html            # Página customizada
└── README.md
```

## Conceitos que aprendi

**become: yes**

Executa como root (equivalente ao sudo). Necessário para instalar pacotes e gerenciar serviços.

```yaml
- name: Playbook com privilégios
  hosts: local
  become: yes   # ou become: true
```

**Módulo apt**

Gerencia pacotes no Ubuntu/Debian.

```yaml
- name: Instalar pacote
  apt:
    name: htop
    state: present       # Garante instalado
    update_cache: yes    # Roda apt update antes
    cache_valid_time: 3600  # Não atualiza se fez há menos de 1h
```

States do apt:
- `present` - Instala se não tiver
- `absent` - Remove se tiver
- `latest` - Instala/atualiza para versão mais recente

**Módulo service**

Gerencia serviços do systemd.

```yaml
- name: Gerenciar serviço
  service:
    name: nginx
    state: started    # Garante rodando
    enabled: yes      # Inicia no boot
```

States do service:
- `started` - Garante rodando (inicia se parado)
- `stopped` - Garante parado (para se rodando)
- `restarted` - Reinicia sempre
- `reloaded` - Recarrega configuração

**Módulo copy**

Copia arquivos para o servidor.

```yaml
- name: Copiar arquivo
  copy:
    src: index.html           # Arquivo local
    dest: /var/www/html/      # Destino no servidor
    owner: www-data           # Dono do arquivo
    group: www-data           # Grupo do arquivo
    mode: '0644'              # Permissões
```

**Módulo uri**

Faz requisições HTTP.

```yaml
- name: Verificar página
  uri:
    url: http://localhost
    return_content: yes   # Retorna o conteúdo da página
  register: resposta

- name: Mostrar conteúdo
  debug:
    msg: "{{ resposta.content }}"
```

**register**

Salva a saída de uma task em variável para uso posterior.

```yaml
- name: Executar comando
  shell: htop --version
  register: versao

- name: Mostrar
  debug:
    msg: "{{ versao.stdout }}"
```

Atributos disponíveis:
- `stdout` - Saída padrão como string
- `stdout_lines` - Saída como lista de linhas
- `rc` - Return code (0 = sucesso)
- `changed` - Se a task mudou algo

**ignore_errors**

Continua o playbook mesmo se a task falhar.

```yaml
- name: Parar serviço que pode não existir
  service:
    name: apache2
    state: stopped
  ignore_errors: yes
```

**changed_when**

Controla quando uma task é marcada como changed.

```yaml
- name: Comando que só lê informação
  shell: cat /etc/hostname
  register: hostname
  changed_when: false   # Nunca marca como changed
```

## Problema encontrado

O Nginx não iniciou porque o Apache estava usando a porta 80. Solução: parar o Apache antes de iniciar o Nginx.

```bash
sudo systemctl stop apache2
```

Ou adicionar task no playbook:

```yaml
- name: Parar Apache
  service:
    name: apache2
    state: stopped
  ignore_errors: yes
```

## Comandos executados

```bash
# Instalar htop
ansible-playbook -i inventory.ini instalar-pacote.yml

# Instalar e gerenciar Nginx
ansible-playbook -i inventory.ini gerenciar-servico.yml

# Copiar página customizada
ansible-playbook -i inventory.ini copiar-arquivo.yml

# Testar
curl localhost
```

## Documentação consultada

- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
- https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html