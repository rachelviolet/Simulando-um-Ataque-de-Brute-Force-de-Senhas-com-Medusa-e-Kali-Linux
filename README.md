# Laboratório de Cibersegurança: Enumeração de Serviços e Auditoria de Senhas

## Visão Geral

Este repositório documenta um exercício controlado de cibersegurança focado em enumeração de serviços, auditoria de senhas e validação de credenciais contra sistemas intencionalmente vulneráveis. O laboratório foi realizado em um ambiente virtual isolado usando Kali Linux e Metasploitable 2.

O objetivo foi praticar uma metodologia repetível para identificar serviços expostos, enumerar usuários, realizar auditorias de senhas com o Medusa e validar credenciais descobertas nos serviços FTP, DVWA e SMB.

## Aviso Ético

Este trabalho foi realizado exclusivamente em um ambiente de laboratório controlado e isolado, usando máquinas virtuais intencionalmente vulneráveis. As técnicas apresentadas aqui estão documentadas apenas para fins educacionais e de portfólio. Não use estas técnicas contra sistemas que você não possui ou para os quais não possui autorização escrita explícita de teste.

## Ambiente do Laboratório

| Componente | Detalhes |
|---|---|
| Hipervisor | VirtualBox |
| VM atacante | Kali Linux |
| VM alvo | Metasploitable 2 |
| Modo de rede | Adaptador Host-Only |
| IP alvo | `192.168.56.102` |

O adaptador de rede Host-Only foi usado para manter o tráfego do laboratório isolado da rede do host e de redes externas.

## Ferramentas Utilizadas

- VirtualBox
- Kali Linux
- Metasploitable 2
- Medusa
- enum4linux
- smbclient
- ftp
- Listas de palavras personalizadas de nomes de usuário e senhas

## Metodologia

### 1. Configuração do Laboratório

1. Instalei o VirtualBox.
2. Importei o Kali Linux e o Metasploitable 2.
3. Configurei ambas as VMs para usar um adaptador de rede Host-Only.
4. Iniciei ambas as máquinas virtuais e confirmei a conectividade.

### 2. Preparação das Listas de Palavras

Listas de palavras personalizadas foram criadas no Kali Linux para apoiar a auditoria de senhas. As listas foram ordenadas intencionalmente para que a credencial `msfadmin` aparecesse mais adiante no arquivo, permitindo observar como a ordenação da lista de palavras afeta o tempo de auditoria.

```bash
echo -e "username\nuser\nadmin\ntest\ntest2\nroot\nsudo\nmsfadmin" > user.txt
echo -e "password\npass\npass123\n12345\n123456\n12345678\nmsfadmin\nroot\nadmin\n123\nqwerty" > pass.txt
```

### 3. Auditoria de Senhas no FTP

O Medusa foi usado para realizar uma auditoria de senhas contra o serviço FTP no alvo Metasploitable 2.

```bash
medusa -h 192.168.56.102 -U user.txt -P pass.txt -t 2 -M ftp
```

A auditoria identificou credenciais FTP válidas: `msfadmin:msfadmin`.

A credencial foi então validada conectando ao serviço FTP:

```bash
ftp 192.168.56.102
```

**Resultado:** Autenticação FTP bem-sucedida usando as credenciais descobertas.

### 4. Auditoria de Login no DVWA

O Medusa foi usado com o módulo HTTP para auditar o formulário de login do DVWA.

```bash
medusa -h 192.168.56.102 -U user.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m 'FAIL=Login failed' -t 6
```

A auditoria identificou credenciais válidas do DVWA: `admin:password`.

**Resultado:** Autenticação bem-sucedida no DVWA usando as credenciais descobertas.

### 5. Enumeração SMB e Auditoria de Senhas

A enumeração SMB foi realizada com o `enum4linux` para identificar usuários e compartilhamentos.

```bash
enum4linux -a 192.168.56.102
```

Os resultados da enumeração forneceram informações úteis sobre usuários, incluindo a conta `msfadmin`. Em seguida, foi realizada uma auditoria de senhas contra o SMB usando o Medusa.

```bash
medusa -h 192.168.56.102 -U user.txt -P pass.txt -M smbnt -t 2 -T 50
```

As credenciais SMB foram validadas com o `smbclient`:

```bash
smbclient -L \\192.168.56.102 -U msfadmin
```

A senha `msfadmin` foi usada para autenticar.

**Resultado:** Autenticação SMB bem-sucedida e listagem de compartilhamentos usando as credenciais descobertas.

## Resumo dos Achados

| Serviço | Método de enumeração / auditoria | Credencial descoberta | Validação |
|---|---|---|---|
| FTP | Módulo FTP do Medusa | `msfadmin:msfadmin` | `ftp 192.168.56.102` |
| DVWA | Módulo HTTP do Medusa | `admin:password` | Formulário de login do DVWA |
| SMB | `enum4linux`, módulo SMB do Medusa | `msfadmin:msfadmin` | `smbclient -L` |

## Principais Aprendizados

- Isolar sistemas vulneráveis com um adaptador de rede Host-Only reduz o risco durante exercícios de laboratório.
- A ordenação e o tamanho da lista de palavras afetam diretamente a duração da auditoria de senhas.
- A enumeração é um precursor crítico para ataques baseados em credenciais; a enumeração SMB identificou nomes de usuário válidos que apoiaram auditorias posteriores.
- Credenciais padrão ou fracas continuam sendo um risco significativo em sistemas mal configurados ou intencionalmente vulneráveis.
- Validar as credenciais descobertas contra o serviço real confirma os achados e reduz falsos positivos.
- Documentação clara de comandos, resultados e metodologia é essencial para testes de segurança repetíveis.

## Propósito do Repositório

Este repositório demonstra familiaridade prática com:

- Configuração de laboratório virtual e isolamento de rede
- Enumeração de serviços
- Auditoria de senhas com Medusa
- Validação de credenciais em FTP, HTTP/DVWA e SMB
- Documentação e relatórios de segurança

> Observação: O IP alvo `192.168.56.102` é específico desta instância do laboratório e pode variar em outros ambientes.
