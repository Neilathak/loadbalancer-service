---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-07"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}

# API 参考
{: #api-reference}

SoftLayer® 应用程序编程接口 (API) 是开发接口，为开发者和系统管理员提供与 SoftLayer 后端系统的直接交互。
{:shortdesc}

SoftLayer API (SLAPI) 在客户门户网站中提供许多功能。通常情况下，如果可在客户门户网站中进行交互，那么也可以在 API 中运行交互。这样，因为您可以通过编程方式在 API 中与 SoftLayer 环境的所有部分进行交互，所以您可以使用 API 来自动执行任务。

SoftLayer API (SLAPI) 是远程过程调用系统。每个调用都涉及向 API 端点发送数据以及接收所返回的结构化数据。通过 SLAPI 发送和接收数据时所使用的格式将取决于您所选择的 API 实现。SLAPI 当前使用 SOAP、XML-RPC 或 REST 进行数据传输。

有关 SoftLayer API、IBM© Cloud Load Balancer 服务 API 的更多信息，请参阅 SoftLayer 开发网络中的以下资源：
* [SoftLayer API 入门 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/article/getting-started/){: new_window}
* [SoftLayer_Product_Package API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Product_Package/){: new_window}
* [SoftLayer_Network_LBaaS_LoadBalancer API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_LoadBalancer/){: new_window}
* [SoftLayer_Network_LBaaS_Listener API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_Listener/){: new_window}
* [SoftLayer_Network_LBaaS_Member API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_Member/){: new_window}
* [SoftLayer_Network_LBaaS_HealthMonitor API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_HealthMonitor/){: new_window}
* [SoftLayer_Network_LBaaS_SSLCipher API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_SSLCipher/){: new_window}
* [SoftLayer_Network_LBaaS_L7Policy API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_L7Policy/){: new_window}
* [SoftLayer_Network_LBaaS_L7Rule API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_L7Rule/){: new_window}
* [SoftLayer_Network_LBaaS_L7Pool API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_L7Pool/){: new_window}
* [SoftLayer_Network_LBaaS_L7Member API ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://softlayer.github.io/reference/services/SoftLayer_Network_LBaaS_L7Member/){: new_window}

以下示例使用 [softlayer-python ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://github.com/softlayer/softlayer-python){: new_window} 客户机。

按如下所示为每个示例设置 API 用户名和 API 密钥，以防您的环境中未配置 `~/.softlayer` 文件。

```py
client = SoftLayer.Client(username='set me', api_key='set me')
```

## 创建负载均衡器
以下示例会检索“Load Balancer As A Service (LBaaS)”包的包标识、子网标识和价格，构建订单数据以及下订单/验证订单。

```py
import SoftLayer
from pprint import pprint

class LBaaSExample():
    def __init__(self):
        self.client = SoftLayer.Client()

    def get_package_id(self, pkg_name):
        """Returns the packageId for the LBaaS"""
        _filter = {"name":{"operation": pkg_name}}

        pkg_list = self.client['Product_Package'].getAllObjects(filter=_filter)

        return pkg_list[0]['id']


    def get_item_prices(self, pkg_id):
        """Returns the standard prices"""

        item_list = self.client['Product_Package'].getItems(id=pkg_id)
        prices = []

        for item in item_list:
            price_id = [p['id'] for p in item['prices']
                            if not p['locationGroupId']][0]
            prices.append(price_id)

        return prices

    def get_subnet_id(self, datacenter):
        """Find and returns the first subnet in the datacenter"""

        _filter = {"privateSubnets": {
                        "networkVlan": {
                            "primaryRouter": {
                                "datacenter": {
                                    "regions": { "keyname": { "operation": datacenter}}
                                }
                            },
                            "type": { "keyName": { "operation": "STANDARD" } }
                        },
                        "routingTypeKeyName": {"operation": "PRIMARY" }
                    }
                }

        subnets = self.client['Account'].getPrivateSubnets(filter=_filter)

        return subnets[0]['id']

    def order_lbaas(self, pkg_name, datacenter, name, desc, protocols,
                    subnet_id=None, public=False, verify=False):
        """Allows to order a Load Balancer"""

        package_id = self.get_package_id(pkg_name)
        prices = self.get_item_prices(package_id)

        # Find and select a subnet id if it was not specified.
        if subnet_id is None:
            subnet_id = self.get_subnet_id(datacenter)

        # Build the configuration of the order
        orderData = {
            'complexType': 'SoftLayer_Container_Product_Order_Network_LoadBalancer_AsAService',
            'name': name,
            'description': desc,
            'location': datacenter,
            'packageId': package_id,
            'useHourlyPricing': True,       # Required since LBaaS is an hourly service            
            'prices': [{'id': price_id} for price_id in prices],
            'protocolConfigurations': protocols,
            'subnets': [{'id': subnet_id}]
        }

        try:
            # If verify=True it will check your order for errors.
            # It will order the lbaas if False.
            if verify:
                response = self.client['Product_Order'].verifyOrder(orderData)
            else:
                response = self.client['Product_Order'].placeOrder(orderData)

            return response
        except SoftLayer.SoftLayerAPIError as e:            
            print("Unable to place the order: %s, %s" % (e.faultCode, e.faultString))


if __name__ == "__main__":
    lbaas = LBaaSExample()

    package_name = 'Load Balancer As A Service (LBaaS)'
    location = 'MEXICO'
    name = 'My-LBaaS-name'
    description = 'A description sample'

    # Set False for private network
    is_public = True

    protocols = [        
        {
            "backendPort": 80,
            "backendProtocol": "HTTP",
            "frontendPort": 8080,
            "frontendProtocol": "HTTP",
            "loadBalancingMethod": "ROUNDROBIN",    # ROUNDROBIN, LEASTCONNECTION, WEIGHTED_RR
            "maxConn": 1000
        }
    ]

    # remove verify=True to place the order
    receipt = lbaas.order_lbaas(package_name, location, name, description,
                                protocols, public=is_public, verify=True)

    pprint(receipt)
```
{: codeblock}

## 检索有关负载均衡器的详细信息
### 列出所有 Load Balancer
```py
import SoftLayer
from prettytable import PrettyTable

class LBaasExample():
    def __init__(self):
        client = SoftLayer.Client()
        self.lbaas_service = client['Network_LBaaS_LoadBalancer']

    def get_list(self, dc=None):
        _filter = None
        lbaas_list = None

        # Use filters if datacenter is set
        if dc:
            _filter = {"datacenter":{"name":{"operation": dc}}}

        try:
            # Retrieve load balancer objects
            lbaas_list = self.lbaas_service.getAllObjects(filter=_filter)
        except SoftLayer.SoftLayerAPIError as e:            
            print("Unable to get the LBaaS list: %s, %s" % (e.faultCode, e.faultString))

        return lbaas_list


if __name__ == "__main__":
    table = PrettyTable(['ID','UUID','Name', 'Description',
                         'Address', 'Type', 'Location', 'Status'])

    # To find all lbaas in Mexico
    datacenter = "mex01"

    lbaas = LBaasExample()
    # remove dc=datacenter to retrieve all the load balancers in the account
    lbaas_list = lbaas.get_list(dc=datacenter)

    # add lbaas data to the table
    for i in lbaas_list:
        isPublic = "Public" if i['isPublic'] == 1 else "Private"
        table.add_row([i['id'], i['uuid'], i['name'], i['description'], i['address'],
                       isPublic,i['datacenter']['longName'],i['operatingStatus']])

    print (table)
```
{: codeblock}

### 检索特定 Load Balancer 的详细信息
```py
import SoftLayer
from pprint import pprint

# Your load balancer UUID
uuid = 'set me'
# mask to retrieve the load balancer's listeners and healthMonitors
_mask = "mask[listeners, healthMonitors]"

# Create the api client
client = SoftLayer.Client()
lbaas_service = client['Network_LBaaS_LoadBalancer']

try:
    # Retrieve a specific load balancer object
    details = lbaas_service.getLoadBalancer(uuid, mask=_mask)
    pprint(details)
except SoftLayer.SoftLayerAPIError as e:            
    print("Unable to retrieve LBaaS details: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

## 更新负载均衡器
### 添加成员
```py
import SoftLayer
from pprint import pprint

# Your load balancer UUID
uuid = 'set me'

# Update with the correct IP addresses
serverInstances = [
    { "privateIpAddress": "10.131.11.46", "weight": 80 },
    { "privateIpAddress": "10.131.11.6" } # Default weight=50
]

# Create the api client
client = SoftLayer.Client()
member_service = client['Network_LBaaS_Member']

_mask = "mask[members]"

try:
    response = member_service.addLoadBalancerMembers(uuid, serverInstances, mask=_mask)
    pprint(response)
except SoftLayer.SoftLayerAPIError as e:            
    print("Unable to add members: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

### 添加协议
```py
import SoftLayer
from pprint import pprint

# Your load balancer UUID
uuid = 'set me'

# New protocols to add
protocolConfigurations = [
    {
        "backendPort": 1350,
        "backendProtocol": "TCP",
        "frontendPort": 1450,
        "frontendProtocol": "TCP",
        "loadBalancingMethod": "WEIGHTED_RR",    # ROUNDROBIN, LEASTCONNECTION, WEIGHTED_RR
        "maxConn": 500,
        "sessionType": "SOURCE_IP"
    },
    {
        "backendPort": 1200,
        "backendProtocol": "HTTP",
        "frontendPort": 1150,
        "frontendProtocol": "HTTP",
        "loadBalancingMethod": "ROUNDROBIN",    # ROUNDROBIN, LEASTCONNECTION, WEIGHTED_RR
        "maxConn": 1000,
        "sessionType": "SOURCE_IP"
    }
]

# Create the api client
client = SoftLayer.Client()
listener_service = client['Network_LBaaS_Listener']

_mask = "mask[listeners]"

try:
    response = listener_service.updateLoadBalancerProtocols(uuid, protocolConfigurations, mask=mask)
    pprint(response)
except SoftLayer.SoftLayerAPIError as e:            
    print("Unable to add protocols: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

## 取消负载均衡器
```py
import SoftLayer

# Your load balancer UUID
uuid = 'set me'

# Create the api client
client = SoftLayer.Client()
lbaas_service = client['Network_LBaaS_LoadBalancer']

try:
    result = lbaas_service.cancelLoadBalancer(uuid)

    if result:
        print("The cancellation request is accepted")
except SoftLayer.SoftLayerAPIError as e:            
    print("Failed to cancel load balancer: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

## 查看负载均衡器的监视度量值
### 获取 HTTP 流量的吞吐量
```py
import SoftLayer
from pprint import pprint

# UUID of load balancer
uuid = 'set me'

# Avaiable options: Throughput, ActiveConnections, and ConnectionRate.
metric_name = 'Throughput'

# Options are: 1hour, 6hours, 12hours, 24hour, 1week, or 2weeks
time_range = '1hour'

# UUID of the listener whose throughput your requesting
protocol_uuid = 'set me'

# Create the api client
client = SoftLayer.Client()
lbaas_service = client['Network_LBaaS_LoadBalancer']

try:
    # Remove protocol_uuid to retrieve the traffic metrics of the entire load balancer
    time_series = lbaas_service.getListenerTimeSeriesData(uuid, metric_name,
                                                          time_range, protocol_uuid)

    pprint(time_series)
except SoftLayer.SoftLayerAPIError as e:            
    print("Unable to retrieve the traffic: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

## 第 7 层 API

### 创建多个 L7 策略和 L7 规则
```py
import SoftLayer
from pprint import pprint

# UUID of the listener
listener_uuid = "set me"

# Build the rules
l7RuleArray1 = [
    {
        "type": "HEADER",
        "comparisonType": "EQUAL_TO",
        "key": "headerkey",
        "value": "header_key3",
        "invert": 0
    },
    {
        "type": "PATH",
        "comparisonType": "STARTS_WITH",
        "value": "/secret_key",
        "invert": 0
    }
]

# Bulk policies and rules configuration
policies_rules = [
    {
        "l7Policy": {
            "name": "traf_test1",
            "action": "REDIRECT_URL",
            "priority":101,
            "redirectUrl": "http://example.com"
        },
     "l7Rules": l7RuleArray1
    }
]

client = SoftLayer.Client()
networkLBaaSL7PolicyService = client['SoftLayer_Network_LBaaS_L7Policy']

try:
    result = networkLBaaSL7PolicyService.addL7Policies(listener_uuid, policies_rules)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to addL7Policies: %s, %s " % (e.faultCode, e.faultString))
```
{: codeblock}

### 更新第 7 层策略
```py
import SoftLayer
from pprint import pprint

# The ID of policy you want to update
networkLBaaSL7PolicyId = 11111111

# Update policy configuration by specifying the variable name and value , e.g. "<name>": "<value>"
policyConfiguration = {
    "name": "<value>"
}

client = SoftLayer.Client()
networkLBaaSL7PolicyService = client['SoftLayer_Network_LBaaS_L7Policy']

try:
    result = networkLBaaSL7PolicyService.editObject(policyConfiguration, id=networkLBaaSL7PolicyId)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to update the Layer 7 policy: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

### 将规则添加到第 7 层策略 
```py
import SoftLayer
from pprint import pprint

# UUID of the Layer 7 policy
policyUuid = "set me"

# New rules to add
rules = [
    {
        "type": "FILE_TYPE",
        "comparisonType": "CONTAINS",
        "value": "some_value",
        "invert": 1
    },
    {
        "type": "PATH",
        "comparisonType": "EQUAL_TO",
        "value": "some_value",
        "invert": 0
    }
]

client = SoftLayer.Client()
networkLBaaSL7RuleService = client['SoftLayer_Network_LBaaS_L7Rule']

try:
    result = networkLBaaSL7RuleService.addL7Rules(policyUuid, rules)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to add rules to a Layer 7 policy: %s, %s " % (e.faultCode, e.faultString))
```
{: codeblock}

### 更新连接到同一个第 7 层策略的多个第 7 层规则 
```py
# For nice debug output:
import SoftLayer
from pprint import pprint

# UUID of the Layer 7 policy
policyUuid = "set me"

# New rules to add
rules = [
    {
        'uuid': '<UUID of the L7 Rule being updated>',
        'type': 'FILE_TYPE',
        'comparisonType': 'CONTAINS',
        'value': 'some_newvalue',
        'invert': 1
    },
    {
        'uuid': '<UUID of the L7 Rule being updated>',
        'type': 'PATH',
        'comparisonType': 'EQUAL_TO',
        'value': 'some_newvalue2',
        'invert': 0
    }
]

client = SoftLayer.Client()
networkLBaaSL7RuleService = client['SoftLayer_Network_LBaaS_L7Rule']

try:
    result = networkLBaaSL7RuleService.updateL7Rules(policyUuid, rules)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to update multiple Layer 7 Rules attached to the same Layer 7 policy: %s, %s"
          % (e.faultCode, e.faultString))
```
{: codeblock}

### 创建具有服务器、运行状况监视和会话亲缘关系的第 7 层池
```py
import SoftLayer
from pprint import pprint

# UUID of load balancer to be updated
loadBalancerUuid = "set me"

# Layer 7 pool to be added
l7Pool = {
    'name': 'image_pool',
    'protocol': 'HTTP',  # only supports HTTP
    'loadBalancingAlgorithm': 'ROUNDROBIN'
}

# Layer 7 Backend servers to be added
l7Members = [
    {
        'address': '10.131.11.46',
        'port': 80,
        'weight': 10
    },
    {
        'address': '10.130.188.130',
        'port': 81,
        'weight': 11
    }
]

# Layer 7 Health monitor to be added
l7HealthMonitor = {
    'interval': 10,
    'timeout': 5,
    'maxRetries': 3,
    'urlPath': '/'
}

# Layer 7 session affinity to be added. Only supports SOURCE_IP as of now
l7SessionAffinity = {
    'type': 'SOURCE_IP'
}

client = SoftLayer.Client()
networkLBaaSL7PoolService = client['SoftLayer_Network_LBaaS_L7Pool']

try:
    result = networkLBaaSL7PoolService.createL7Pool(loadBalancerUuid, l7Pool, l7Members,
                                                    l7HealthMonitor, l7SessionAffinity)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:    
    print("Unable to create a Layer 7 Pool with servers: %s, %s"
          % (e.faultCode, e.faultString))
```
{: codeblock}

### 更新第 7 层池以及运行状况监视和会话亲缘关系
```py
import SoftLayer
from pprint import pprint

# UUID of L7 pool to be updated
l7PoolUuid = 'set me'

# New Layer 7 pool values to be updated
l7Pool = {'loadBalancingAlgorithm': 'LEASTCONNECTION'}

# New Layer 7 Health monitor values to be updated
l7HealthMonitor = {'urlPath': '/index'}

# New Layer 7 session affinity values to be updated.
# If not given it deletes the existing session affinity
# If given and session affinity doesn't exist, it creates one.
l7SessionAffinity = {'type': 'SOURCE_IP'}

client = SoftLayer.Client()
networkLBaaSL7PoolService = client['SoftLayer_Network_LBaaS_L7Pool']

try:
    result = networkLBaaSL7PoolService.updateL7Pool(l7PoolUuid, l7Pool,
                                                    l7HealthMonitor, l7SessionAffinity)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to update a Layer 7 pool along with health monitoring and session affinity: %s, %s"
          % (e.faultCode, e.faultString))
```
{: codeblock}

### 将服务器添加到第 7 层池
```py
import SoftLayer
from pprint import pprint

# UUID of the L7 pool to which members should be added.
l7PoolUuid = 'set me'

# Backend servers to be added
memberInstances = [
    {
        'address': '10.131.11.46',
        'port': 80,
        'weight': 10
    },
    {
        'address': '10.130.188.130',
        'port': 81,
        'weight': 11
    }
]

client = SoftLayer.Client()
networkLBaaSL7MemberService = client['SoftLayer_Network_LBaaS_L7Member']

try:
    result = networkLBaaSL7MemberService.addL7PoolMembers(l7PoolUuid, memberInstances)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to add servers to a Layer 7 Pool: %s, %s" % (e.faultCode, e.faultString))
```
{: codeblock}

### 更新属于第 7 层池的服务器
```py
import SoftLayer
from pprint import pprint

# UUID of the L7 pool who's member we need to update
l7PoolUuid = 'set me'

# Backend servers to be added
members = [
    {
        'uuid': '1e433123-ceae-4bbd-a4e3-2539ceb8b46f',  # UUID of the member to be updated.
        'address': '10.73.67.83',
        'port': 90,
        'weight': 10
    },
    {
        'uuid': 'bd3524db-26c8-4662-982e-04a68a420fba',
        'address': '10.73.67.83',
        'port': 91,
        'weight': 11
    }
]

client = SoftLayer.Client()
networkLBaaSL7MemberService = client['SoftLayer_Network_LBaaS_L7Member']

try:
    result = networkLBaaSL7MemberService.updateL7PoolMembers(l7PoolUuid, members)
    pprint(result)
except SoftLayer.SoftLayerAPIError as e:
    print("Unable to update severs belonging to a Layer 7 pool: %s, %s"
          % (e.faultCode, e.faultString))
```
