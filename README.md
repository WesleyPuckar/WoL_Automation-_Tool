# 🚀 Wake-on-LAN (WoL) Automation Tool

![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

Esta ferramenta em PowerShell foi desenvolvida para automatizar a inicialização remota de estações de trabalho em rede local (LAN). É a solução ideal para recuperação de ambientes após quedas de energia ou para manutenção remota em parques computacionais.

## 📌 Índice
- [Tecnologias e Ferramentas](#️-tecnologias-e-ferramentas)
- [Como Funciona](#-como-funciona)
- [Configuração Obrigatória](#️-configuração-obrigatória)
- [Como Utilizar](#-como-utilizar)
- [Detalhamento do Código](#-detalhamento-do-código)
- [Segurança e Conformidade Corporativa](#️-segurança-e-conformidade-corporativa)
- [Justificativa Técnica](#-justificativa-técnica)
  
---
## 🛠️ Tecnologias e Ferramentas

Para o desenvolvimento e segurança deste projeto, foram utilizadas as seguintes tecnologias:

| Categoria | Tecnologia | Finalidade |
| :--- | :--- | :--- |
| **Linguagem** | ![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white) | Automação e interação com a stack Windows. |
| **Framework** | ![.NET](https://img.shields.io/badge/.NET-512BD4?style=flat-square&logo=dotnet&logoColor=white) | Uso da classe `UdpClient` para envio de pacotes de baixo nível. |
| **Segurança** | ![Infosec](https://img.shields.io/badge/Security-Hardening-red?style=flat-square&logo=looker&logoColor=white) | Implementação de diretrizes de conformidade corporativa. |
| **Ambiente** | ![Windows](https://img.shields.io/badge/Windows-0078D6?style=flat-square&logo=windows&logoColor=white) | Sistema operacional alvo e gerenciador de energia. |
| **Versionamento** | ![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white) | Controle de versão e documentação via Markdown. |

---

## ⚡ Como Funciona

O script utiliza o conceito de **Magic Packet**. Ele lê uma lista de endereços MAC e dispara um frame Ethernet via protocolo **UDP** para o endereço de broadcast da rede. 

> **O que é o Magic Packet?**
> É um frame que contém 6 bytes de valor `0xFF` seguidos por 16 repetições do endereço MAC do computador alvo. Ao detectar esse padrão, a placa de rede (NIC) sinaliza para a fonte de alimentação ligar o computador.

---

## 🛠️ Configuração Obrigatória

Para que o hardware consiga "ouvir" o pacote enquanto está desligado, siga os ajustes abaixo:

### 1. Hardware (BIOS/UEFI)
| Configuração | Ajuste | Descrição |
| :--- | :--- | :--- |
| **Wake on LAN/WLAN** | `LAN Only` | Permite que o sistema ligue via sinal de rede. |
| **Deep Sleep Control** | `Disabled` | Impede que a placa de rede seja totalmente desligada em S4/S5. |

**Guia Visual de Configuração:**

| Wake on LAN/WLAN | Deep Sleep Control |
| :---: | :---: |
| ![BIOS WoL](https://github.com/user-attachments/assets/f0b9cf1b-5d58-47cb-9de9-5c2554666901) | ![BIOS Deep Sleep](https://github.com/user-attachments/assets/2b5bce0b-f216-46ff-8730-c56398ca0754) |
| *Configuração restrita à rede cabeada.* | *Desativação do modo de economia agressiva.* |

### 2. Sistema Operacional (Windows)
* **Desativação do Fast Startup (Inicialização Rápida):** Ponto crítico! O Fast Startup coloca o kernel em hibernação híbrida, o que muitas vezes desliga os recursos de escuta da placa de rede.
* **Gerenciamento de Energia:** No *Gerenciador de Dispositivos*, as propriedades da placa Ethernet devem estar com a opção *"Permitir que este dispositivo acorde o computador"* marcada.

> [!NOTE]
> Após testes não foi necessário seguir com essa ativação dado a configuração correta na BIOS
---

## 📋 Como Utilizar

1. **Clone o repositório:**
   ```bash
   git clone (https://github.com/WesleyPuckar/WoL_Automation-_Tool.git)
   
2. **Prepare a lista de alvos Crie um arquivo chamado macs.txt na mesma pasta do script e insira um endereço MAC por linha**
   ```plaintext
   C0-47-0E-F2-15-BE
   A1-B2-C3-D4-E5-F6

3. **Execute o script: Abra o PowerShell e execute:**
   ```powershell
    .\wakePc.ps1

## 💻 Detalhamento do Código

O script segue um fluxo lógico estruturado para garantir a integridade do acionamento:

* **Leitura e Sanitização:** O script realiza a importação dos dados contidos no arquivo `macs.txt` e aplica um tratamento de strings para remover espaços ou caracteres especiais, garantindo que o endereço MAC esteja no formato correto para processamento.
* **Construção do Frame (Magic Packet):**
    * **Cabeçalho:** Criação de um prefixo de 6 bytes preenchidos com o valor hexadecimal `0xFF`.
    * **Conversão:** O endereço MAC alvo é convertido de hexadecimal para uma matriz de bytes.
    * **Concatenação:** O MAC convertido é repetido e concatenado 16 vezes logo após o cabeçalho, formando a estrutura padrão do Magic Packet.
* **Transmissão:** Utiliza a classe `.NET UdpClient` para realizar o disparo do pacote via protocolo UDP para o endereço de **Broadcast** da sub-rede (`255.255.255.255`) através da **porta 9**.

---

## 🛡️ Segurança e Conformidade Corporativa

Para que esta ferramenta seja utilizada em um ambiente de produção sem comprometer a segurança da rede, é importante destacar que o **Wake-on-LAN (WoL)** é um padrão industrial (IEEE 802.3) e não uma vulnerabilidade. 

Abaixo, detalhamos por que as configurações adotadas são seguras:

### 1. Impacto das Alterações de Hardware (BIOS/UEFI)
Uma preocupação comum é se permitir o acionamento remoto cria um "backdoor". A resposta é **não**, pelos seguintes motivos:
* **Apenas Energia, não Acesso:** O pacote WoL atua apenas no nível físico (Power Management). Ele sinaliza para a fonte ligar o hardware, mas **não ignora a autenticação do sistema operacional**. O computador ligará e parará na tela de login ou na criptografia de disco (BitLocker).
* **Deep Sleep Control:** Desativar o Deep Sleep apenas mantém a placa de rede em modo de escuta (*standby*). Ela não processa dados, não executa comandos e não possui endereço IP ativo enquanto a máquina está desligada; ela apenas reconhece o padrão binário do "Magic Packet".

| Configuração | É uma vulnerabilidade? | Justificativa Técnica |
| :--- | :--- | :--- |
| **WoL Enabled** | 🟢 Não | Exige que o atacante já esteja dentro da rede interna e saiba o MAC Address específico da máquina. |
| **Deep Sleep Disabled** | 🟢 Não | Apenas permite que a NIC (placa de rede) receba energia mínima para detectar o frame Ethernet. |

### 2. Segurança do "Magic Packet" (UDP Port 9)
O script utiliza o protocolo UDP, que é um protocolo sem conexão. 
* **Unidirecionalidade:** O script apenas "grita" na rede. Não há troca de mensagens, não há handshake e a máquina alvo não devolve nenhuma informação. Isso impede ataques de interceptação de dados durante o acionamento.
* **Filtro por Endereço Físico:** A placa de rede só reage se o pacote contiver o seu endereço MAC exato repetido 16 vezes. Ruídos na rede ou scanners de porta não ativam o recurso acidentalmente.

### 3. Por que desativar o Fast Startup?
Esta alteração no Windows é puramente operacional. O Fast Startup faz um desligamento híbrido que coloca os drivers em um estado de "dormência" que impede a escuta da rede. Desativá-lo **melhora a segurança** em alguns aspectos, pois garante que o kernel seja totalmente reiniciado, aplicando atualizações de segurança pendentes que muitas vezes não são instaladas em desligamentos rápidos.

### 4. Mitigações de Risco Recomendadas
Para garantir que a ferramenta seja vista como uma aliada da administração:
* **Acesso Restrito ao Script:** O script e o arquivo `macs.txt` devem estar em um diretório com permissões restritas (apenas para a equipe de TI/Suporte).
* **Segmentação:** Recomenda-se que o tráfego de broadcast seja limitado à VLAN de administração, evitando que usuários comuns tenham capacidade de enviar pacotes de acionamento.

> [!TIP]
> **Conclusão para Auditoria:** O uso desta ferramenta não altera a superfície de ataque (Attack Surface) do sistema operacional, uma vez que todas as barreiras de software (Firewall, Antivírus e Login) permanecem ativas imediatamente após o boot.
---

## 📝 Justificativa Técnica

A implementação desta ferramenta de automação visa a otimização operacional da equipe de TI, oferecendo os seguintes benefícios:

* **Redução do MTTR:** Diminui drasticamente o tempo médio de resposta (Mean Time To Repair) em incidentes críticos de infraestrutura, como quedas de energia.
* **Eficiência Logística:** Em cenários de prédios remotos ou unidades descentralizadas, o script permite que toda a frota de estações de trabalho seja reestabelecida em segundos.
* **Autonomia:** Elimina a necessidade de deslocamento físico dos técnicos ou a dependência de intervenção manual por parte de usuários locais para ligar os equipamentos.

---

**Desenvolvido por [WesleyPuckar]** 🚀
