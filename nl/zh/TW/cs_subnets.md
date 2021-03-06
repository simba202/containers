---

copyright:
  years: 2014, 2018
lastupdated: "2018-09-10"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}


# 配置叢集的子網路
{: #subnets}

將子網路新增至 Kubernetes 叢集，為負載平衡器服務變更可用的可攜式公用或專用 IP 位址的儲存區。
{:shortdesc}

## 叢集的預設 VLAN、子網路及 IP
{: #default_vlans_subnets}

在建立叢集期間，叢集工作者節點及預設子網路會自動連接至 VLAN。

### VLAN
{: #vlans}

當您建立叢集時，叢集的工作者節點會自動連接至 VLAN。VLAN 會配置一組工作者節點及 Pod，彷彿它們已連接至相同的實體佈線，並提供一個通道，在工作者與 Pod 之間進行連線。

<dl>
<dt>免費叢集的 VLAN</dt>
<dd>在免費叢集裡，叢集的工作者節點依預設會連接至 IBM 擁有的公用 VLAN 及專用 VLAN。由於 IBM 控制了 VLAN、子網路及 IP 位址，所以您無法建立多區域叢集，或將子網路新增至您的叢集，只能使用 NodePort 服務來公開您的應用程式。</dd>
<dt>標準叢集的 VLAN</dt>
<dd>在標準叢集裡，第一次在區域中建立叢集時，會在您的 IBM Cloud 基礎架構 (SoftLayer) 帳戶中，自動為您在該區域中佈建公用 VLAN 及專用 VLAN。針對您在該區域建立的每個後續叢集，您可以重複使用相同的公用及專用 VLAN，因為多個叢集可以共用 VLAN。</br></br>您可以將工作者節點同時連接至公用 VLAN 及專用 VLAN，或僅連接至專用 VLAN。如果您只想要將工作者節點連接至專用 VLAN，則可以使用現有專用 VLAN 的 ID，或[建立專用 VLAN](/docs/cli/reference/ibmcloud/cli_vlan.html#ibmcloud-sl-vlan-create)，並在建立叢集期間使用該 ID。</dd></dl>

若要查看每一個區域中針對您帳戶佈建的 VLAN，請執行 `ibmcloud ks vlans <zone>`。若要查看某一個叢集佈建所在的 VLAN，請執行 `ibmcloud ks cluster-get<cluster_name_or_ID> --showResources`，並找出 **Subnet VLANs** 區段。

**附註**：
* 如果一個叢集具有多個 VLAN，相同的 VLAN 上具有多個子網路，或多個區域叢集，您必須針對 IBM Cloud 基礎架構 (SoftLayer) 帳戶啟用 [VLAN Spanning](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)，讓您的工作者節點可在專用網路上彼此通訊。若要執行此動作，您需要**網路 > 管理網路 VLAN Spanning** [基礎架構許可權](cs_users.html#infra_access)，或者您可以要求帳戶擁有者啟用它。若要檢查是否已啟用 VLAN Spanning，請使用 `ibmcloud s vlan-spanning` [指令](/docs/containers/cs_cli_reference.html#cs_vlan_spanning_get)。如果您使用 {{site.data.keyword.BluDirectLink}}，則必須改用[虛擬路由器功能 (VRF)](/docs/infrastructure/direct-link/subnet-configuration.html#more-about-using-vrf)。若要啟用 VRF，請聯絡 IBM Cloud 基礎架構 (SoftLayer) 客戶業務代表。
* IBM Cloud 基礎架構 (SoftLayer) 管理在您於區域中建立第一個叢集時所自動供應的 VLAN。如果您讓 VLAN 變成未使用（例如移除 VLAN 中的所有工作者節點），IBM Cloud 基礎架構 (SoftLayer) 會收回此 VLAN。之後，如果您需要新建的 VLAN，請[與 {{site.data.keyword.Bluemix_notm}} 支援中心聯絡](/docs/infrastructure/vlans/order-vlan.html#order-vlans)。

### 子網路及 IP 位址
{: #subnets_ips}

除了工作者節點及 Pod 外，子網路也會自動佈建在 VLAN 上。子網路可藉由將 IP 位址指派給您的叢集元件，為它們提供網路連線功能。

下列子網路會自動佈建在預設公用及專用 VLAN 上：

**公用 VLAN 子網路**
* 主要公用子網路會決定在建立叢集期間指派給工作者節點的公用 IP 位址。相同 VLAN 上的多個叢集可以共用一個主要公用子網路。
* 可攜式公用子網路只會連結至一個叢集，並提供該叢集 8 個公用 IP 位址。3 個 IP 會保留給 Softlayer 功能使用。1 個 IP 由預設公用 Ingress ALB 使用，而 4 個 IP 可以用來建立公用負載平衡器網路服務。可攜式公用 IP 是永久性的固定 IP 位址，可以用來透過網際網路存取負載平衡器服務。如果公用負載平衡器需要超過 4 個 IP，請參閱[新增可攜式 IP 位址](#adding_ips)。

**專用 VLAN 子網路**
* 主要專用子網路會決定在建立叢集期間指派給工作者節點的專用 IP 位址。相同 VLAN 上的多個叢集可以共用一個主要專用子網路。
* 可攜式專用子網路只會連結至一個叢集，並提供該叢集 8 個專用 IP 位址。3 個 IP 會保留給 Softlayer 功能使用。1 個 IP 由預設專用 Ingress ALB 使用，而 4 個 IP 可以用來建立專用負載平衡器網路服務。可攜式專用 IP 是永久性的固定 IP 位址，可以用來透過網際網路存取負載平衡器服務。如果專用負載平衡器需要超過 4 個 IP，請參閱[新增可攜式 IP 位址](#adding_ips)。

若要查看您帳戶中佈建的所有子網路，請執行 `ibmcloud ks subnets`。若要查看連結至某一個叢集的可攜式公用及可攜式專用子網路，您可以執行 `ibmcloud ks cluster-get<cluster_name_or_ID> --showResources`，並找出 **Subnet VLANs** 區段。

**附註**：在 {{site.data.keyword.containerlong_notm}} 中，VLAN 的限制為 40 個子網路。如果您達到此限制，首先請查看您是否可以[重複使用 VLAN 中的子網路來建立新的叢集](#custom)。如果您需要新的 VLAN，請[與 {{site.data.keyword.Bluemix_notm}} 支援中心聯絡](/docs/infrastructure/vlans/order-vlan.html#order-vlans)，進行訂購。然後，[建立叢集](cs_cli_reference.html#cs_cluster_create)，而叢集使用這個新的 VLAN。

<br />


## 使用自訂或現有的子網路來建立叢集
{: #custom}

建立標準叢集時，會自動為您建立子網路。不過，您可以不要使用自動佈建的子網路，而使用 IBM Cloud 基礎架構 (SoftLayer) 帳戶中的現有可攜式子網路，或重複使用已刪除叢集裡的子網路。
{:shortdesc}

使用此選項，可在移除及建立叢集時保留穩定的靜態 IP 位址，或訂購更大的 IP 位址區塊。

**附註：**可攜式公用 IP 位址為按月收費。如果您在佈建叢集之後移除可攜式公用 IP 位址，則仍須支付一個月的費用，即使您只是短時間使用也是一樣。

開始之前，
- [將 CLI 的目標設為](cs_cli_install.html#cs_cli_configure)您的叢集。
- 若要重複使用來自您不再需要之叢集的子網路，請刪除不需要的叢集。請立即建立新的叢集，因為如果您未重複使用子網路，這些子網路會在 24 小時內遭到刪除。

   ```
   ibmcloud ks cluster-rm <cluster_name_or_ID>
   ```
   {: pre}

若要使用 IBM Cloud 基礎架構 (SoftLayer) 組合中的現有子網路來搭配自訂防火牆規則或可用的 IP 位址，請執行下列動作：

1. 取得您要使用之子網路的 ID，以及子網路所在 VLAN 的 ID。

    ```
    ibmcloud ks subnets
    ```
    {: pre}

    在此輸出範例中，子網路 ID 是 `1602829`，而 VLAN ID 是 `2234945`：
    ```
    Getting subnet list...
    OK
    ID        Network             Gateway          VLAN ID   Type      Bound Cluster
    1550165   10.xxx.xx.xxx/26    10.xxx.xx.xxx    2234947   private
    1602829   169.xx.xxx.xxx/28   169.xx.xxx.xxx   2234945   public
    ```
    {: screen}

2. 使用您所識別的 VLAN ID 來[建立叢集](cs_clusters.html#clusters_cli)。包含 `--no-subnet` 旗標，以防止自動建立新的可攜式公用 IP 子網路及新的可攜式專用 IP 子網路。

    ```
    ibmcloud ks cluster-create --zone dal10 --machine-type b2c.4x16 --no-subnet --public-vlan 2234945 --private-vlan 2234947 --workers 3 --name my_cluster
    ```
    {: pre}
    對於 `--zone` 旗標，如果您不記得 VLAN 所在的區域，可以執行 `ibmcloud ks vlans <zone>` 來檢查 VLAN 是否位於特定區域中。
    {: tip}

3.  驗證已建立叢集。**附註：**訂購工作者節點機器，並在您的帳戶中設定及佈建叢集，最多可能需要 15 分鐘。

    ```
    ibmcloud ks clusters
    ```
    {: pre}

    完全佈建叢集後，**狀況**會變更為 `deployed`。

    ```
    Name         ID                                   State      Created          Workers   Zone   Version
    mycluster    aaf97a8843a29941b49a598f516da72101   deployed   20170201162433   3         dal10      1.10.7
    ```
    {: screen}

4.  檢查工作者節點的狀態。

    ```
    ibmcloud ks workers <cluster_name_or_ID>
    ```
    {: pre}

    在繼續進行下一個步驟之前，工作者節點必須就緒。**狀況**會變更為 `normal`，而**狀態**是 `Ready`。

    ```
    ID                                                  Public IP        Private IP     Machine Type   State      Status   Zone   Version
    prod-dal10-pa8dfcc5223804439c87489886dbbc9c07-w1    169.xx.xxx.xxx   10.xxx.xx.xxx  free           normal     Ready    dal10      1.10.7
    ```
    {: screen}

5.  指定子網路 ID，以將子網路新增至叢集。當您讓子網路可供叢集使用時，系統會為您建立 Kubernetes ConfigMap，其中包含您可以使用的所有可用可攜式公用 IP 位址。如果子網路 VLAN 所在的區域裡沒有 Ingress ALB 存在，則會自動使用一個可攜式公用 IP 位址及一個可攜式專用 IP 位址，來建立該區域的公用及專用 ALB。您可以使用來自該子網路的所有其他可攜式公用及專用 IP 位址，為您的應用程式建立負載平衡器服務。

    ```
    ibmcloud ks cluster-subnet-add mycluster 807861
    ```
    {: pre}

6. **重要事項**：若要為位於相同 VLAN 之不同子網路上的工作者節點，啟用它們之間的通訊，您必須[在相同 VLAN 上的子網路之間啟用遞送](#subnet-routing)。

<br />


## 管理現有的可攜式 IP 位址
{: #managing_ips}

依預設，藉由[建立負載平衡器服務](cs_loadbalancer.html)，即可使用 4 個可攜式公用 IP 位址及 4 個可攜式專用 IP 位址，將單一應用程式公開至公用或專用網路。若要建立負載平衡器服務，您必須具有至少 1 個可用且類型正確的可攜式 IP 位址。您可以檢視可用的可攜式 IP 位址，或釋放已使用的可攜式 IP 位址。

### 檢視可用的可攜式公用 IP 位址
{: #review_ip}

若要列出您叢集裡的所有可攜式 IP 位址，包括已使用及可用的可攜式 IP 位址，您可以執行：

```
kubectl get cm ibm-cloud-provider-vlan-ip-config -n kube-system -o yaml
```
{: pre}

若只要列出可用來建立負載平衡器的可攜式公用 IP 位址，您可以使用下列步驟：

開始之前，[請為您要使用的叢集設定環境定義。](cs_cli_install.html#cs_cli_configure)

1.  建立名為 `myservice.yaml` 的 Kubernetes 服務配置檔，並且定義類型為 `LoadBalancer` 且具有虛擬負載平衡器 IP 位址的服務。下列範例使用 IP 位址 1.1.1.1 作為負載平衡器 IP 位址。

    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: myservice
      name: myservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: myservice
      sessionAffinity: None
      type: LoadBalancer
      loadBalancerIP: 1.1.1.1
    ```
    {: codeblock}

2.  在叢集裡建立服務。

    ```
    kubectl apply -f myservice.yaml
    ```
    {: pre}

3.  檢查服務。

    ```
    kubectl describe service myservice
    ```
    {: pre}

    **附註：**建立此服務失敗，因為 Kubernetes 主節點在 Kubernetes ConfigMap 中找不到指定的負載平衡器 IP 位址。當您執行這個指令時，可以看到錯誤訊息以及叢集的可用公用 IP 位址清單。

    ```
    Error on cloud load balancer a8bfa26552e8511e7bee4324285f6a4a for service default/myservice with UID 8bfa2655-2e85-11e7-bee4-324285f6a4af: Requested cloud provider IP 1.1.1.1 is not available. The following cloud provider IP addresses are available: <list_of_IP_addresses>
    ```
    {: screen}

<br />


### 釋放已使用的 IP 位址
{: #free}

您可以刪除正在使用可攜式 IP 位址的負載平衡器服務，以釋放已使用的可攜式 IP 位址。
{:shortdesc}

開始之前，[請為您要使用的叢集設定環境定義。](cs_cli_install.html#cs_cli_configure)

1.  列出叢集裡可用的服務。

    ```
    kubectl get services
    ```
    {: pre}

2.  移除使用公用或專用 IP 位址的負載平衡器服務。

    ```
    kubectl delete service <service_name>
    ```
    {: pre}

<br />


## 新增可攜式 IP 位址
{: #adding_ips}

依預設，藉由[建立負載平衡器服務](cs_loadbalancer.html)，即可使用 4 個可攜式公用 IP 位址及 4 個可攜式專用 IP 位址，將單一應用程式公開至公用或專用網路。若要建立超過 4 個公用或 4 個專用負載平衡器，您可以將網路子網路新增至叢集來取得更多的可攜式 IP 位址。

**附註：**
* 當您讓子網路可供叢集使用時，會使用這個子網路的 IP 位址來進行叢集網路連線。若要避免 IP 位址衝突，請確定一個子網路只搭配使用一個叢集。請不要同時將一個子網路用於多個叢集或 {{site.data.keyword.containerlong_notm}} 以外的其他用途。
* 可攜式公用 IP 位址是按月計費。如果您在佈建子網路之後移除可攜式公用 IP 位址，則仍須支付一個月的費用，即使您只是短時間使用。

### 透過訂購更多子網路來新增可攜式 IP
{: #request}

您可以在 IBM Cloud 基礎架構 (SoftLayer) 帳戶中建立新的子網路，並讓它可供您指定的叢集使用，而為負載平衡器服務取得更多的可攜式 IP。
{:shortdesc}

開始之前，請先將 [CLI 的目標](cs_cli_install.html#cs_cli_configure)設為您的叢集。

1. 佈建新的子網路。

    ```
    ibmcloud ks cluster-subnet-create <cluster_name_or_id> <subnet_size> <VLAN_ID>
    ```
    {: pre}

    <table>
    <caption>瞭解這個指令的元件</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="構想圖示"/> 瞭解這個指令的元件</th>
    </thead>
    <tbody>
    <tr>
    <td><code>cluster-subnet-create</code></td>
    <td>為叢集佈建子網路的指令。</td>
    </tr>
    <tr>
    <td><code><em>&lt;cluster_name_or_id&gt;</em></code></td>
    <td>將 <code>&lt;cluster_name_or_id&gt;</code> 取代為叢集的名稱或 ID。</td>
    </tr>
    <tr>
    <td><code><em>&lt;subnet_size&gt;</em></code></td>
    <td>將 <code>&lt;subnet_size&gt;</code> 取代為您要從可攜式子網路新增的 IP 位址數目。接受值為 8、16、32 或 64。<p>**附註：**當您新增子網路的可攜式 IP 位址時，會使用三個 IP 位址來建立叢集內部網路。您無法將這三個 IP 位址用於應用程式負載平衡器，或是用它們來建立負載平衡器服務。例如，如果您要求八個可攜式公用 IP 位址，則可以使用其中的五個將您的應用程式公開給大眾使用。</p> </td>
    </tr>
    <tr>
    <td><code><em>&lt;VLAN_ID&gt;</em></code></td>
    <td>將 <code>&lt;VLAN_ID&gt;</code> 取代為您要在其上配置可攜式公用或專用 IP 位址之公用或專用 VLAN 的 ID。您必須選取現有工作者節點所連接的公用或專用 VLAN。若要檢閱工作者節點的公用或專用 VLAN，請執行 <code>ibmcloud ks worker-get &lt;worker_id&gt;</code> 指令。<子網路佈建於 VLAN 所在的同一個區域中。</td>
    </tr>
    </tbody></table>

2. 驗證已順利建立子網路並將其新增至叢集。子網路 CIDR 列在 **Subnet VLANs** 區段中。

    ```
    ibmcloud ks cluster-get --showResources <cluster_name_or_ID>
    ```
    {: pre}

    在此輸出範例中，第二個子網路已新增至 `2234945` 公用 VLAN：
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR          Public   User-managed
    2234947   10.xxx.xx.xxx/29     false    false
    2234945   169.xx.xxx.xxx/29    true     false
    2234945   169.xx.xxx.xxx/29    true     false
    ```
    {: screen}

3. **重要事項**：若要為位於相同 VLAN 之不同子網路上的工作者節點，啟用它們之間的通訊，您必須[在相同 VLAN 上的子網路之間啟用遞送](#subnet-routing)。

<br />


### 利用使用者管理的子網路來新增可攜式專用 IP
{: #user_managed}

您可以讓來自內部部署網路的子網路可供指定的叢集使用，為負載平衡器服務取得更多的專用可攜式 IP。
{:shortdesc}

需求：
- 使用者管理的子網路只能新增至專用 VLAN。
- 子網路字首長度限制為 /24 到 /30。例如，`169.xx.xxx.xxx/24` 指定 253 個可用的專用 IP 位址，而 `169.xx.xxx.xxx/30` 指定 1 個可用的專用 IP 位址。
- 子網路中的第一個 IP 位址必須用來作為子網路的閘道。

開始之前：
- 配置進出外部子網路的網路資料流量遞送。
- 確認您在內部部署資料中心網路閘道與專用網路 Virtual Router Appliance 或您叢集裡執行的 strongSwan VPN 服務之間，具有 VPN 連線功能。如需相關資訊，請參閱[設定 VPN 連線功能](cs_vpn.html)。

若要從內部部署網路新增子網路，請執行下列動作：

1. 檢視叢集專用 VLAN 的 ID。找出 **Subnet VLANs** 區段。在 **User-managed** 欄位中，識別具有 _false_ 的 VLAN ID。

    ```
    ibmcloud ks cluster-get --showResources <cluster_name>
    ```
    {: pre}

    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR       Public   User-managed
    2234947   10.xxx.xx.xxx/29  false    false
    2234945   169.xx.xxx.xxx/29 true     false
    ```
    {: screen}

2. 將外部子網路新增至您的專用 VLAN。可攜式專用 IP 位址會新增至叢集的 ConfigMap。

    ```
    ibmcloud ks cluster-user-subnet-add <cluster_name> <subnet_CIDR> <VLAN_ID>
    ```
    {: pre}

    範例：

    ```
    ibmcloud ks cluster-user-subnet-add mycluster 10.xxx.xx.xxx/24 2234947
    ```
    {: pre}

3. 驗證已新增使用者提供的子網路。**User-managed** 欄位為 _true_。

    ```
    ibmcloud ks cluster-get --showResources <cluster_name>
    ```
    {: pre}

    在此輸出範例中，第二個子網路已新增至 `2234947` 專用 VLAN：
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR       Public   User-managed
    2234947   10.xxx.xx.xxx/29  false    false
    2234945   169.xx.xxx.xxx/29 true     false
    2234947   10.xxx.xx.xxx/24  false    true
    ```
    {: screen}

4. [在相同 VLAN 上的子網路之間啟用遞送](#subnet-routing)。

5. 新增[專用負載平衡器服務](cs_loadbalancer.html)或啟用[專用 Ingress ALB](cs_ingress.html#private_ingress)，以透過專用網路存取您的應用程式。若要使用來自您新增之子網路中的專用 IP 位址，您必須指定來自子網路 CIDR 中的 IP 位址。否則，會從 IBM Cloud 基礎架構 (SoftLayer) 子網路或專用 VLAN 上使用者提供的子網路中，隨機選擇一個 IP 位址。

<br />


## 管理子網路遞送
{: #subnet-routing}

如果一個叢集具有多個 VLAN，相同的 VLAN 上具有多個子網路，或多個區域叢集，您必須針對 IBM Cloud 基礎架構 (SoftLayer) 帳戶啟用 [VLAN Spanning](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)，讓您的工作者節點可在專用網路上彼此通訊。若要執行此動作，您需要**網路 > 管理網路 VLAN Spanning** [基礎架構許可權](cs_users.html#infra_access)，或者您可以要求帳戶擁有者啟用它。若要檢查是否已啟用 VLAN Spanning，請使用 `ibmcloud s vlan-spanning` [指令](/docs/containers/cs_cli_reference.html#cs_vlan_spanning_get)。如果您使用 {{site.data.keyword.BluDirectLink}}，則必須改用[虛擬路由器功能 (VRF)](/docs/infrastructure/direct-link/subnet-configuration.html#more-about-using-vrf)。若要啟用 VRF，請聯絡 IBM Cloud 基礎架構 (SoftLayer) 客戶業務代表。

檢閱下列也需要 VLAN Spanning 的情境。

### 在相同 VLAN 上的主要子網路之間啟用遞送
{: #vlan-spanning}

當您建立叢集時，結尾為 `/26` 的子網路會佈建在預設專用主要 VLAN 上。專用主要子網路可以為最多 62 個工作者節點提供 IP。
{:shortdesc}

大型叢集，或相同 VLAN 上單一地區中的數個較小叢集，都可能會超出這項 62 個工作者節點的限制。達到 62 個工作者節點的限制時，會在相同 VLAN 中訂購第二個專用主要子網路。

若要確保相同 VLAN 上這些主要子網路中的工作者節點可以通訊，您必須開啟 VLAN Spanning。如需指示，請參閱[啟用或停用 VLAN Spanning](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)。

若要檢查是否已啟用 VLAN Spanning，請使用 `ibmcloud s vlan-spanning` [指令](cs_cli_reference.html#cs_vlan_spanning_get)。
{: tip}

### 管理閘道應用裝置的子網路遞送
{: #vra-routing}

建立叢集時，會在連接叢集的 VLAN 上訂購可攜式公用及可攜式專用子網路。這些子網路會提供 Ingress 及負載平衡器網路服務的 IP 位址。

不過，如果您具有現有的路由器應用裝置，例如 [Virtual Router Appliance (VRA)](/docs/infrastructure/virtual-router-appliance/about.html#about)，則不會在路由器上配置剛新增的可攜式子網路，這些子網路來自叢集連接到的那些 VLAN。若要使用 Ingress 或負載平衡器網路服務，您必須藉由[啟用 VLAN Spanning](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)，確保網路裝置可以在相同 VLAN 上的不同子網路之間遞送。

若要檢查是否已啟用 VLAN Spanning，請使用 `ibmcloud s vlan-spanning` [指令](cs_cli_reference.html#cs_vlan_spanning_get)。
{: tip}
