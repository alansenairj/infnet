# infnet - Projeto de Conclusão de Bloco : Arquitetura e Infraestrutura de Aplicações

Disponibilizar um IaaS com base no Openstack para Nuvem privada

Basicamente a solução Openstack utilizará a relação dos seguintes recursos físicos:

Item
Servidor Computação (Nova)
Servidor Controle
Servidor Armazenamento (Cinder)
Sistemas Operacionais
Versão Openstack

Descrição / Especificações
Workstation z 840
PC Core I7 / 16 GB RAM HD 500 GB
PC Core I7 / 16 GB RAM HD 2 TB
CentOS 7
RDO Triple-o Rocky Release


Premissas do projeto: 

Ambiente de desktops virtuais para criação de sites
Ambiente de produção separado de ambiente de testes
Armazenamento de seu banco de dados
Backup e redundância
Performance elástica das instâncias para dias de forte demanda de acesso
Balanceamento de carga para lidar com dias de forte demanda de acesso
Firewall e roteamento

Ambiente de criação compartilhado e com controle de versionamento
O gerenciamento da configuração será realizado por implementação da ferramenta Ansible
Hospedagem do Wordpress foram usadas 3 imagens diferentes. A imagem redis é um servidor de dicionário para armazenamento persistente, disponibilizado no repositório Docker em: https://hub.docker.com/_/redis/ e a outra imagem percona tem a finalidade de armazenar o banco de dados da aplicação, nesse caso o banco de dados mysql. https://hub.docker.com/_/percona/. A última imagem é Wordpress que contém a aplicação que servirá de ambiente de desenvolvimento do site. Essa aplicação roda nessa imagem através do servidos apache.

![image](https://user-images.githubusercontent.com/20565821/124002384-0c5dac80-d9ac-11eb-8fda-d2f0bdb37d42.png)

Calculo de dimensionamento de Recursos para as Instâncias: 

Em cada servidor executaríamos no máximo 72 VMs, calculado pela fórmula:
SOCKET ∗CORES ∗ T HREADS/MAX_vCPU ⇒ (2 ∗ 6 ∗ 12)/2 = 72
OBS: o hypervisor pode limitar esses recursos.

Precisaríamos de 30 cores para executar 60 VMs, calculado pela fórmula:
TOTALV MS ∗ MAX_vCPU ⇒60 ∗ 3 = 180

Precisaríamos de 8 servidores HP z840, calculado pela fórmula:
TOTAL_CORES/CORES_POR_SERVIDOR ⇒ 180/24 = 8 aprox.

Precisaríamos de 768 GB de RAM por servidores, calculado pela fórmula:
VMS_POR_SERVIDOR ∗ RAM_POR_VM ⇒ 72 ∗ 8 = 576 ∼ 768/8 = 96 GB por servidor

Porém a real carga e quantidade de máquinas virtuais por servidor dependem da
carga que vai ser executada, podendo variar para mais ou para menos.


Repositório do Projeto de Bloco de AIAPB

$ cd roles
$ ansible-galaxy init server
$ ansible-galaxy init php
$ ansible-galaxy init mysql
$ ansible-galaxy init wordpress

![image](https://user-images.githubusercontent.com/20565821/124002652-63638180-d9ac-11eb-936c-68ca0f0a96da.png)


Infra Openstack

![image](https://user-images.githubusercontent.com/20565821/124002970-c81edc00-d9ac-11eb-959c-15f277a61f82.png)

Abaixo os passos que devem ser considerados para realizar a ligação de maneira assertiva, pois existe muita dificuldade para que a mesma seja feita e a compreensão do cenário exige mais cuidado.

1- Consulte as interfaces ifconfig -a e instale o openssh-server54
2- Consulte as rotas netstat -r ou route e habilite o encaminhamento de pacotes.
3 - Atribua uma rota estática ou uma regra de iptable para o endereço da interface do
VMWare (vmnet1 ou vmnet8) mas o gateway da saída da rota tem que ser a sua
interface física (enp50s0) que tem contato com a sua LAN.
4 - Mude as políticas no security group pra permitir ssh e icmp
5- Aloque ou não o ip flutuante
6- Suba a instância e ping no router ou ping na instancia
7 - Baixe a chave privada no key do Openstack
8- Converta a chave para o formato open-ssh sem senha com puttygen (linha de
comando ou interface no Windows mesmo) ou através de um compartilhamento com o
samba. Um pendrive na máquina é mais prático também
9- Chmod 600 na chave convertida
10 - Ssh -i (caminho da sua chave pub convertida) .pub user@iphost




