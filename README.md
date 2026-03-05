# Análise de Incidente de Segurança em Windows – Investigação Prática no TryHackMe

Platform: TryHackMe  
Category: Blue Team / Incident Response  
Tools Used: PowerShell, Event Viewer, Registry Analysis, Task Scheduler  

# Windows Incident Investigation

Nesta sala do **TryHackMe**, fui desafiada a conduzir uma investigação detalhada de um incidente de segurança em um ambiente Windows comprometido. O objetivo principal era analisar um sistema totalmente infectado por malware, identificar sinais de comprometimento e extrair informações críticas sobre o incidente.

Durante a exploração, apliquei diversas técnicas de **análise de logs**, **PowerShell**, **comandos do CMD** e investigação de **configurações do sistema**, sempre documentando cada passo para garantir uma abordagem metódica e replicável. Este relatório reflete a narrativa da investigação, demonstrando o raciocínio por trás de cada comando e cada descoberta.

<img width="813" height="299" alt="image" src="https://github.com/user-attachments/assets/3fa388cf-b662-42fa-b1c1-9357f86c4bd7" />

# Abordagem Inicial e Coleta de Informações

A primeira etapa do processo foi a **coleta de informações básicas sobre o sistema**. Com o comando: `winver` pude confirmar a versão do Windows e o ano de instalação, obtendo contexto essencial para a investigação. Entender o ambiente é crítico para qualquer análise de incidentes, pois algumas vulnerabilidades e comportamentos maliciosos podem variar de acordo com a versão do sistema. Já o comando `net user` permitiu listar todas as contas registradas, fornecendo uma visão sobre possíveis contas comprometidas ou suspeitas.

<img width="510" height="136" alt="image" src="https://github.com/user-attachments/assets/f550d3d2-2429-4b52-8934-0f9f7e33d2ed" />
<img width="806" height="234" alt="image" src="https://github.com/user-attachments/assets/120f90e6-da95-44e9-86be-aa44b33e5a21" />

### **Acessos ao sistema**

**Pergunta:** Quando foi a última vez que John acessou o sistema? Para obter informações sobre o último logon do usuário **John**, utilizei o comando:`net user John` com isso, pude confirmar de forma confiável quando John acessou o sistema pela última vez, um passo fundamental para rastrear a atividade de contas e identificar possíveis acessos não autorizados.

<img width="677" height="420" alt="image" src="https://github.com/user-attachments/assets/827cfe06-f7be-4835-93f4-d2259d79d60a" />

### Conexão de rede ao iniciar o sistema

**Pergunta:** A qual endereço IP o sistema se conecta quando é iniciado pela primeira vez? Iniciei a investigação verificando o arquivo **Hosts**, onde notei diversos sites bloqueados, mas a informação buscada não estava presente ali. Isso indicou que algum processo era executado **antes do Hosts ser carregado**. Seguindo essa lógica, acessei o **Registro do Windows** (`regedit`) e naveguei até: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run lá, identifiquei o arquivo **UpdateSvc**, responsável por inicializar processos automaticamente. Ao inspecionar este arquivo, descobri o **endereço IP ao qual a máquina estava se conectando**, permitindo determinar a origem da comunicação inicial e evidenciar possíveis tentativas de redirecionamento malicioso.

<img width="806" height="187" alt="image" src="https://github.com/user-attachments/assets/0b27a0bc-cd2a-4e29-b65c-b2dc1c9ab885" />

### **Permissões de conta**

**Pergunta:** Quais outras contas possuíam privilégios de administrador? Para analisar quais contas tinham privilégios administrativos no sistema, utilizei o **PowerShell** com o comando: `Get-LocalGroupMember -Group "Administrators”`  consegui uma lista com todos os usuários pertencentes ao grupo **Administrators**, permitindo identificar contas com acesso privilegiado. Esta análise é essencial para compreender a superfície de ataque do sistema e avaliar quais usuários poderiam potencialmente executar ações críticas ou maliciosas.

<img width="691" height="151" alt="image" src="https://github.com/user-attachments/assets/49ceef22-997a-452a-8598-84529d29c5f8" />

### **Tarefa agendada maliciosa**

**Pergunta:** Qual o nome da tarefa agendada maliciosa? Após perceber que o **Task Manager** não estava respondendo, utilizei o **PowerShell** para continuar a investigação. Com o comando: Get-ScheduledTask, pude identificar uma tarefa suspeita que **não tinha sido criada pela Microsoft**, confirmando o nome da tarefa maliciosa que estava sendo executada no sistema.

<img width="873" height="220" alt="image" src="https://github.com/user-attachments/assets/b891be50-f8c2-41ec-a006-02492593ce7e" />

### **Arquivo executado e porta de escuta**

**Pergunta:** Qual arquivo a tarefa tentava executar diariamente e em qual porta estava escutando localmente? Analisei o **Task Scheduler**, verificando todas as tarefas agendadas diariamente. Dentro da tarefa **Clean File System**, identifiquei o arquivo configurado para execução no **arranque da máquina** e a porta local em que ele estava escutando, confirmando a atividade maliciosa automatizada.

<img width="778" height="378" alt="image" src="https://github.com/user-attachments/assets/1f880b75-94da-4b43-b064-7e3b4763c6df" />

### **Último login de Jenny e data do acordo**

**Pergunta:** Quando Jenny fez login pela última vez e em que data ocorreu o acordo? Retornei ao **PowerShell** e utilizei: `net user Jenny` e com isso, consegui obter o histórico de logons da usuária, respondendo ambas as perguntas com precisão, incluindo datas e horários relevantes.

<img width="636" height="409" alt="image" src="https://github.com/user-attachments/assets/19981299-8101-4c57-952b-76970c51379b" />

### **Privilégios especiais atribuídos**

**Pergunta:** Durante a invasão, em que momento o Windows atribuiu privilégios especiais a um novo usuário? Acessei o **Event Viewer** em: Windows Logs > Security  e filtrei pelo **Event ID 4672** e pela data do evento (3/02/2019), completando com os segundos fornecidos pelo site. Abrindo os detalhes do log, identifiquei exatamente quando o Windows concedeu privilégios administrativos ao usuário recém-criado.

<img width="1081" height="215" alt="image" src="https://github.com/user-attachments/assets/4cabf3cd-dd76-4cb0-a541-3ea97cb6f9bb" />

### **Ferramenta utilizada para roubo de senhas**

**Pergunta:** Qual ferramenta foi usada para obter as senhas do Windows? Durante a análise do disco local **C:\TMP**, encontrei o arquivo **MIM.OUT**. A estrutura e os dados contidos nele correspondiam ao comportamento do **Mimikatz**, e haviam passwords reveladas no arquivo, confirmando que essa foi a ferramenta utilizada pelos atacantes para extrair credenciais.

<img width="943" height="458" alt="image" src="https://github.com/user-attachments/assets/45ae413f-4c01-4cc9-a067-8fcdc277f14c" />
<img width="675" height="181" alt="image" src="https://github.com/user-attachments/assets/179a7d3d-3256-40f4-90da-c060cb2c0c8c" />

### **Endereço IP dos servidores de C2 e DNS envenenado**

**Pergunta:** Qual era o endereço IP dos servidores externos de controle e comando dos atacantes?Verifique se houve envenenamento de DNS. Qual site foi alvo do ataque? Investiguei o arquivo **Hosts** localizado em: `C:\Windows\System32\drivers\etc\hosts` lá, encontrei o **IP 76.32.97.132**, identificando o servidor de comando e controle. Além disso, o envenenamento de DNS estava direcionando solicitações para o **site google.com**, confirmando o alvo do ataque.

<img width="980" height="335" alt="image" src="https://github.com/user-attachments/assets/d79963e6-5323-48c7-84fa-d7e99195e7e6" />

### **Última porta aberta pelo atacante**

**Pergunta:** Qual foi a última porta aberta pelo atacante? Acesseio o **Firewall do Windows** em **Inbound Rules** e examinei as regras ativas. A primeira regra listada, **Allow outside connections for development**, revelou que a porta **1337** foi aberta pelo atacante, sendo a última porta utilizada para acesso externo.

<img width="801" height="107" alt="image" src="https://github.com/user-attachments/assets/ca1d1c2b-442e-4114-a26a-12e31886fbcb" />
<img width="833" height="117" alt="image" src="https://github.com/user-attachments/assets/32376410-f093-441a-8082-1ce4972d07c7" />

### **Extensão do shell carregado**

**Pergunta:** Qual era o nome da extensão do shell carregado através do site do servidor? Explorei o diretório **inetpub** para analisar os arquivos servidos pelo servidor web. Identifiquei arquivos com a extensão **.jsp (Jakarta Server Pages)**, que correspondia ao shell carregado, confirmando a tecnologia utilizada pelos atacantes.

<img width="843" height="162" alt="image" src="https://github.com/user-attachments/assets/d5a636ef-bdc5-4877-9e24-bda8fff8abe6" />

## **Conclusão**

Durante esta investigação, demonstrei a aplicação de **conhecimentos em análise de sistemas Windows**, incluindo a interpretação de logs, auditoria de contas de usuário, inspeção de tarefas agendadas e análise de conectividade de rede. Cada etapa foi guiada por raciocínio crítico, utilizando comandos do **PowerShell**, **CMD**, inspeção de registros no **Event Viewer** e análise de arquivos de configuração, mostrando capacidade de identificar indicadores de comprometimento e atividades maliciosas de forma estruturada e metodológica.

Além disso, a investigação evidenciou o uso de **ferramentas especializadas**, como o **Mimikatz** para extração de credenciais, e técnicas de persistência, como tarefas agendadas e shells web (.jsp). A correlação entre acessos de usuários, atividades de rede, manipulação de DNS e portas abertas demonstrou a compreensão de todo o **ciclo de ataque**, conectando cada evidência para reconstruir o incidente de forma clara e fundamentada. Este relatório reflete tanto o **domínio técnico** quanto a capacidade de aplicar metodologia investigativa em cenários reais de segurança.

## Skills Demonstrated
- Windows Log Analysis
- PowerShell Investigation
- Scheduled Task Analysis
- Registry Forensics
- DNS Poisoning Detection
- Credential Dumping Identification


