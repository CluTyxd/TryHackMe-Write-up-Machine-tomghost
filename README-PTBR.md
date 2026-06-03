# Tomghost - Write-up TryHackMe

## Enumeração

O primeiro passo foi realizar o reconhecimento do alvo utilizando o Nmap para identificar serviços expostos e possíveis vetores de ataque.
<br><br>
<img width="942" height="482" alt="image" src="https://github.com/user-attachments/assets/3ab912d0-562e-4dd8-84e7-2976ca6e7ded" />
<br><br>
A varredura revelou diversas portas abertas, incluindo SSH na porta 22, um serviço HTTP executando Apache Tomcat na porta 8080 e o serviço AJP13 na porta 8009. A presença do AJP chamou atenção imediatamente como um possível vetor de ataque devido a vulnerabilidades conhecidas que afetam o Apache Tomcat.

## Enumeração Web

Com a varredura inicial concluída, o próximo passo foi investigar a aplicação web exposta na porta 8080.
<br><br>
<img width="1299" height="820" alt="image" src="https://github.com/user-attachments/assets/c1de1ecd-7e4f-4001-b1b0-2c2677f58692" />
<br><br>
Ao acessar o serviço, foi exibida a página padrão do Apache Tomcat, confirmando a versão em uso e fornecendo informações adicionais sobre o ambiente.

Foram realizadas tentativas de acesso às interfaces Manager App, Host Manager e Server Status.
<br><br>
<img width="1913" height="482" alt="image" src="https://github.com/user-attachments/assets/b55520ad-7262-48f7-9dd6-595d8982c4e8" />
<br><br>
No entanto, o acesso a esses endpoints administrativos estava restrito, exigindo uma investigação mais aprofundada da instalação do Tomcat.

## Pesquisa de Vulnerabilidades

Considerando o serviço AJP exposto e a versão identificada do Tomcat, foi realizada uma pesquisa para identificar vulnerabilidades publicamente conhecidas que afetassem o alvo.

A investigação confirmou a presença de uma vulnerabilidade associada ao protocolo AJP, conhecida como Ghostcat (CVE-2020-1938), que permite leitura arbitrária de arquivos sob determinadas condições.

Os detalhes da vulnerabilidade confirmaram que o conector AJP exposto poderia ser explorado para acessar arquivos sensíveis da instalação do Tomcat.
<br><br>
<img width="793" height="289" alt="image" src="https://github.com/user-attachments/assets/36ed4c63-ae26-4aac-85fc-1a86028195c0" />
<br><br>
## Explorando o Ghostcat

Após confirmar a vulnerabilidade, o Metasploit foi iniciado para simplificar o processo de exploração.
<br><br>
<img width="783" height="422" alt="image" src="https://github.com/user-attachments/assets/174a5aec-0044-4dc6-8fbd-cbff25c5444f" />
<br><br>
O módulo Ghostcat foi selecionado e configurado contra o alvo.
<br><br>
<img width="931" height="38" alt="image" src="https://github.com/user-attachments/assets/8b33b173-ad2d-4076-a518-88467cd32909" />
<br><br>
A exploração foi bem-sucedida e permitiu acesso a arquivos internos da aplicação, revelando credenciais armazenadas na configuração do sistema.
<br><br>
<img width="658" height="507" alt="image" src="https://github.com/user-attachments/assets/b7db2ac0-36d6-488b-b126-076400025c09" />
<br><br>
## Acesso Inicial

Durante a fase de enumeração, o serviço SSH já havia sido identificado na porta 22. As credenciais recuperadas foram então testadas na autenticação SSH.
<br><br>
<img width="772" height="386" alt="image" src="https://github.com/user-attachments/assets/52b3218e-ac90-476f-84c8-9416f833f60a" />
<br><br>
As credenciais eram válidas, fornecendo acesso inicial ao sistema como o usuário `skyfuck`.

## Enumeração de Usuários

Após obter acesso ao sistema, os diretórios pessoais dos usuários foram explorados com o objetivo de identificar contas adicionais e arquivos sensíveis.
<br><br>
<img width="519" height="158" alt="image" src="https://github.com/user-attachments/assets/b495cceb-f009-4f99-bdc4-ecf4f0af0432" />
<br><br>
Durante esse processo, a flag de usuário foi localizada dentro do diretório pessoal de Merlin.

## Enumeração para Escalação de Privilégios

Diversas verificações de escalonamento de privilégios foram realizadas, incluindo enumeração de binários SUID e análise das permissões sudo. Inicialmente, nenhum caminho óbvio para escalonamento foi identificado.
<br><br>
<img width="589" height="273" alt="image" src="https://github.com/user-attachments/assets/36205c6a-a0a9-4434-b2c7-83aed0c58f7c" />
<br><br>
Como resultado, a atenção foi direcionada para arquivos localizados no diretório pessoal do usuário Skyfuck.

## Descoberta da Chave PGP

Dois arquivos chamaram atenção imediatamente: `credential.pgp` e `tryhackme.asc`.
<br><br>
<img width="271" height="39" alt="image" src="https://github.com/user-attachments/assets/9aa11ba1-a88f-41fd-8ad7-96be54dacfbe" />
<br><br>
Ao inspecionar o arquivo `tryhackme.asc`, foi identificada uma chave privada PGP.
<br><br>
<img width="587" height="474" alt="image" src="https://github.com/user-attachments/assets/1b553f5d-d697-485a-ba4d-f46740a3c611" />
<br><br>
A chave foi copiada para a máquina atacante para análise posterior.

## Quebrando a Senha da Chave PGP

O hash da chave PGP foi extraído para preparação do processo de quebra de senha.
<br><br>
<img width="365" height="85" alt="image" src="https://github.com/user-attachments/assets/6befe03f-0688-4e6a-b8d4-308f1e36b223" />
<br><br>
O hash extraído foi quebrado com sucesso utilizando o John the Ripper, revelando a senha que protegia a chave privada.
<br><br>
<img width="925" height="233" alt="image" src="https://github.com/user-attachments/assets/8a76ada6-c890-4054-aa4d-95ad25cc4df9" />
<br><br>
## Descriptografando as Credenciais Armazenadas

Retornando à máquina alvo, a chave privada foi importada para o GPG.
<br><br>
<img width="654" height="166" alt="image" src="https://github.com/user-attachments/assets/d66bce59-23b6-4497-9279-03022e2a5f22" />
<br><br>
A verificação confirmou que a importação foi realizada com sucesso e que a chave pertencia ao usuário da TryHackMe.

<br><br>
<img width="463" height="108" alt="image" src="https://github.com/user-attachments/assets/425f5e68-3382-4480-8136-ef2816799176" />
<br><br>

Utilizando a senha recuperada, o arquivo criptografado `credential.pgp` foi descriptografado.
<br><br>
<img width="714" height="195" alt="image" src="https://github.com/user-attachments/assets/868ad0d1-a26a-40c6-93a9-f25fddadf363" />
<br><br>
O conteúdo descriptografado revelou credenciais pertencentes ao usuário `merlin`.

## Movimentação para o Usuário Merlin

Com as credenciais válidas recuperadas, foi possível alternar da conta `skyfuck` para a conta `merlin`.
<br><br>
<img width="563" height="60" alt="image" src="https://github.com/user-attachments/assets/11734069-f262-413e-89e9-627e987143fe" />
<br><br>
Isso forneceu acesso a um novo contexto de usuário e abriu novas oportunidades para escalonamento de privilégios.

## Configuração Incorreta do Sudo

A enumeração das permissões sudo revelou uma configuração incorreta crítica.
<br><br>
<img width="728" height="114" alt="image" src="https://github.com/user-attachments/assets/7318263c-7d5e-4c8b-b5b0-25ea6b06de54" />
<br><br>
O usuário `merlin` tinha permissão para executar o binário `zip` como root sem a necessidade de fornecer senha.

## Escalação de Privilégios via GTFOBins

Uma pesquisa no GTFOBins revelou que o binário `zip` poderia ser abusado para obter um shell privilegiado.
<br><br>
<img width="879" height="271" alt="image" src="https://github.com/user-attachments/assets/c6cbd872-9e27-484d-88d3-f31556d6342d" />
<br><br>
A execução da técnica documentada permitiu obter com sucesso um shell com privilégios de root.
<br><br>
<img width="643" height="125" alt="image" src="https://github.com/user-attachments/assets/83091ff9-43d3-46a4-8787-844b20151557" />
<br><br>
## Acesso Root

Com privilégios de root obtidos, foi possível navegar até o diretório do usuário root.
<br><br>
<img width="399" height="96" alt="image" src="https://github.com/user-attachments/assets/a242d9be-ca02-40a2-a0db-f0124e33e5f1" />
<br><br>
A flag de root foi recuperada com sucesso, concluindo a máquina.

## Principais Aprendizados

* Enumeração com Nmap
* Enumeração de Apache Tomcat
* Enumeração do Protocolo AJP
* Ghostcat (CVE-2020-1938)
* Exploração com Metasploit
* Vulnerabilidade de Divulgação de Arquivos
* Coleta de Credenciais
* Acesso via SSH
* Enumeração em Linux
* Análise de Chaves PGP
* John the Ripper
* Descriptografia com GPG
* Movimentação entre Usuários
* Enumeração de Permissões Sudo
* GTFOBins
* Escalação de Privilégios
* Comprometimento Total do Sistema

## Recomendações de Segurança

Para mitigar os problemas identificados durante esta avaliação, as seguintes medidas de segurança devem ser implementadas:

* Manter o Apache Tomcat atualizado para uma versão não afetada pela CVE-2020-1938 (Ghostcat).
* Desabilitar o conector AJP caso ele não seja necessário para a aplicação.
* Restringir o acesso ao serviço AJP por meio de regras de firewall e segmentação de rede.
* Evitar o armazenamento de credenciais em texto puro dentro de arquivos de configuração.
* Implementar políticas de senha fortes e credenciais únicas para todos os usuários.
* Proteger informações sensíveis utilizando soluções seguras de gerenciamento de segredos.
* Auditar regularmente diretórios de usuários em busca de arquivos sensíveis e credenciais.
* Utilizar senhas robustas para proteger chaves privadas PGP.
* Restringir o acesso SSH apenas a usuários autorizados e utilizar autenticação por chave sempre que possível.
* Revisar periodicamente as permissões sudo e aplicar o princípio do menor privilégio.
* Evitar conceder privilégios de execução como root a binários que possam ser utilizados para obtenção de shell.
* Realizar avaliações de segurança e auditorias periódicas para identificar vulnerabilidades e configurações incorretas antes que possam ser exploradas.

## Conclusão

Tomghost foi uma excelente máquina para praticar enumeração de aplicações web, pesquisa de vulnerabilidades, coleta de credenciais e escalonamento de privilégios em Linux.

A cadeia de ataque começou com a identificação de uma instância Apache Tomcat exposta e de um conector AJP acessível. A investigação levou à descoberta e exploração da vulnerabilidade Ghostcat (CVE-2020-1938), permitindo a divulgação de informações sensíveis e a recuperação de credenciais válidas.

Após obter acesso inicial via SSH, uma nova etapa de enumeração revelou credenciais criptografadas protegidas por uma chave privada PGP. Através da extração e quebra da senha da chave, foi possível descriptografar as credenciais armazenadas e realizar a movimentação para o usuário `merlin`.

A etapa final consistiu na identificação de uma configuração incorreta do sudo que permitia a execução do binário `zip` como root. Utilizando uma técnica documentada no GTFOBins, foi possível obter acesso administrativo completo e comprometer totalmente a máquina.

De forma geral, esta sala proporcionou uma excelente experiência prática com exploração da vulnerabilidade Ghostcat, recuperação de credenciais, análise de chaves PGP, enumeração em Linux, técnicas de escalonamento de privilégios e a importância de configurações seguras e controles de acesso adequados.
