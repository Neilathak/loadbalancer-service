---

copyright:
  years: 2018
lastupdated: "2018-11-12"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Resolução de problemas de fornecimento do balanceador de carga
{: #load-balancer-provisioning-troubleshooting}

Este tópico fornece informações sobre problemas comuns que você pode encontrar ao criar uma
nova instância do IBM© Cloud Load Balancer.

## Endereços IP insuficientes em sua sub-rede
O IBM Cloud Load Balancer requer pelo menos dois endereços IP livres em sua sub-rede privada. Além disso, se for um balanceador de carga público e a opção **Conjunto de sistemas IBM** não for usada, pelo menos dois endereços IP livres também serão necessários em sua sub-rede pública. 

Siga as etapas abaixo para verificar se há IPs livres em uma sub-rede.

1. Acesse [Portal do Cliente ![Ícone de linkexterno](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com){:new_window} e navegue até a seção de sub-redes selecionando

**Rede > Gerenciamento de IP > Sub-redes**.

2. Clique na sub-rede na qual você deseja verificar se há IPs livres.

	<img src="images/subnet_list.png" alt="drawing" style="width: 600px;"/>
		
3. A página de detalhes da sub-rede selecionada mostra o status de todos os IPs nessa sub-rede.

## Problemas com firewalls em VLANs públicas e privadas
Consulte o tópico [Intervalo de IP do IBM Cloud](/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) para obter informações sobre a permissão de intervalos de IP por meio do firewall.
 
## Visualizando mensagens de erro do balanceador de carga
Para visualizar mensagens de erro de seu balanceador de carga, execute o procedimento a seguir:

1. Clique no balanceador de carga na página de lista para visualizar seus detalhes. 
2. Passe o mouse sobre o símbolo de erro ao lado do status do balanceador de carga.

<img src="images/lbaas_error_message.png" alt="drawing" style="width: 500px;"/>

Se o balanceador de carga estiver em um estado `ERROR`, ele não poderá ser recuperado e deverá ser excluído.
