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


# Configuración de clústeres y nodos trabajadores
{: #clusters}
Cree clústeres y añada nodos trabajadores para aumentar la capacidad del clúster en {{site.data.keyword.containerlong}}. ¿Aún se está familiarizando? Pruebe la [guía de aprendizaje para la creación de un clúster de Kubernetes](cs_tutorials.html#cs_cluster_tutorial).
{: shortdesc}

## Creación de clústeres con la GUI
{: #clusters_ui}

La finalidad del clúster de Kubernetes es definir un conjunto de recursos, nodos, redes y dispositivos de almacenamiento que mantengan la alta disponibilidad de las apps. Para poder desplegar una app, debe crear un clúster y establecer las definiciones de los nodos trabajadores en dicho clúster.
{:shortdesc}

**Antes de empezar**
* Debe tener una [cuenta de {{site.data.keyword.Bluemix_notm}}](https://console.bluemix.net/registration/) de prueba, pago según uso o de suscripción.
* Decida entre un [clúster gratuito o estándar](cs_why.html#cluster_types).
* Si crea un clúster estándar, puede [estimar el coste con la calculadora de precios ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://console.bluemix.net/pricing/configure/iaas/containers-kubernetes). **Nota**: Debe añadir una **Red dedicada (1 subred con 8 IP públicas portátiles)** para cada clúster a la estimación. Además, el coste del ancho de banda de red de salida varía según el uso y es un cargo adicional aparte de los cargos de la calculadora de precios.

Para personalizar completamente su clúster y añadir, entre otras características, la versión de API, la zona o el aislamiento de hardware, cree un clúster estándar.
{: tip}

**Para crear un clúster gratuito**

Puede utilizar 1 clúster gratuito para familiarizarse con el funcionamiento de {{site.data.keyword.containerlong_notm}}. Con los clústeres gratuitos puede aprender la terminología, completar una guía de aprendizaje y familiarizarse con el sistema antes de dar el salto a los clústeres estándares de nivel de producción. No se preocupe, sigue disponiendo de un clúster gratuito, aunque tenga una cuenta de Pago según uso o de Suscripción. **Nota**: los clústeres gratuitos tienen un período de vida de 30 días. Transcurrido este periodo, el clúster caduca y el clúster y sus datos se suprimen. {{site.data.keyword.Bluemix_notm}} no hace copia de seguridad de los datos suprimidos y no se pueden restaurar. Asegúrese de realizar una copia de seguridad de los datos importantes.

1. En el catálogo, seleccione **{{site.data.keyword.containerlong_notm}}**.

2. Seleccione una región en la que desea desplegar el clúster. **Nota**: No se pueden crear clústeres gratuitos en las regiones de EE.UU. este o AP norte y las zonas correspondientes.

3. Seleccione el plan de clúster **Gratuito**.

4. Asigne un nombre al clúster. El nombre debe empezar por una letra, puede contener letras, números, guiones (-) y debe tener 35 caracteres como máximo. El nombre del clúster y la región en la que el clúster se despliega forman el nombre de dominio completo para el subdominio de Ingress. Para garantizar que el subdominio de Ingress es exclusivo dentro de una región, el nombre del clúster podría ser truncado y añadírsele un valor aleatorio dentro del nombre de dominio de Ingress.


5. Pulse **Crear clúster**. De forma predeterminada, se crea una agrupación de nodos trabajadores con un nodo trabajador. Verá el progreso del despliegue del nodo trabajador en el separador **Nodos trabajadores**. Cuando finalice el despliegue, podrá ver que el clúster está listo en el separador **Visión general**.

    Si se cambia el nombre de dominio o el ID exclusivo asignado durante la creación, se impide que el nodo maestro de Kubernetes gestione el clúster.
    {: tip}

</br>

**Para crear un clúster estándar**

1. En el catálogo, seleccione **{{site.data.keyword.containerlong_notm}}**.

2. Seleccione una región en la que desea desplegar el clúster. Para obtener el mejor rendimiento, seleccione la región físicamente más cercana. Tenga en cuenta que, si selecciona una zona que está fuera de su país, es posible que necesite autorización legal para que se puedan almacenar datos.

3. Seleccione el plan de clúster **Estándar**. Con un clúster estándar, obtiene acceso a características como, por ejemplo, varios nodos trabajadores para obtener un entorno de alta disponibilidad.

4. Especifique los detalles de la zona.

    1. Seleccione la disponibilidad **Una sola zona** o **Multizona**. En un clúster multizona, el nodo maestro se despliega en una zona con soporte multizona y los recursos del clúster se distribuyen entre varias zonas. Las opciones pueden verse limitadas por región.

    2. Seleccione las zonas específicas en las que desea alojar el clúster. Debe seleccionar al menos 1 zona, pero puede seleccionar tantas como desee. Si selecciona más de 1 zona, los nodos trabajadores se distribuyen entre las zonas que elija, lo que le proporciona una mayor disponibilidad. Si selecciona solo 1 zona, puede [añadir zonas a su clúster](#add_zone) después de crearlo.

    3. Seleccione una VLAN pública (opcional) y una VLAN privada (obligatorio) en la cuenta de la infraestructura de IBM Cloud (SoftLayer). Los nodos trabajadores se comunican entre sí a través de la VLAN privada. Para comunicarse con el nodo maestro de Kubernetes, debe configurar la conectividad pública para el nodo trabajador.  Si no tiene una VLAN pública o privada en esta zona, déjelo en blanco. Se crea automáticamente una VLAN privada y una pública. Si ya tiene VLAN y no especifica una VLAN pública, tenga en cuenta la posibilidad de configurar un cortafuegos, como por ejemplo [Virtual Router Appliance](/docs/infrastructure/virtual-router-appliance/about.html#about). Puede utilizar la misma VLAN para varios clústeres.
        **Nota**: Si los nodos trabajadores únicamente se configuran con una VLAN privada, debe configurar una solución alternativa para la conectividad de red. Para obtener más información, consulte [Planificación de redes de clúster solo privado](cs_network_cluster.html#private_vlan).

5. Configure la agrupación de nodos trabajadores predeterminada. Las agrupaciones de nodos trabajadores son grupos de nodos trabajadores que comparten la misma configuración. Siempre puede añadir más agrupaciones de nodos trabajadores a su clúster posteriormente.

    1. Seleccione un tipo de aislamiento de hardware. La opción virtual se factura por hora, y la nativa se factura mensualmente.

        - **Virtual - Dedicado**: Los nodos trabajadores están alojados en una infraestructura que está dedicada a su cuenta. Sus recursos físicos están completamente aislados.

        - **Virtual - Compartido**: Los recursos de infraestructura, como por ejemplo el hipervisor y el hardware físico, están compartidos entre usted y otros clientes de IBM, pero cada nodo trabajador es accesible sólo por usted. Aunque esta opción es menos costosa y suficiente en la mayoría de los casos, es posible que desee verificar los requisitos de rendimiento e infraestructura con las políticas de la empresa.

        - **Nativo**: Los servidores nativos se facturan de forma mensual y su suministro se realiza mediante interacción manual con la infraestructura de IBM Cloud (SoftLayer), por lo que puede tardar más de un día laborable en realizarse. Los servidores nativos son más apropiados para aplicaciones de alto rendimiento que necesitan más recursos y control de host. Para clústeres que ejecutan la versión 1.9 o posterior de Kubernetes, también puede optar por habilitar [Trusted Compute](cs_secure.html#trusted_compute) para verificar que los nodos trabajadores no se manipulan de forma indebida. Trusted Compute está disponible para determinados tipos de máquinas nativas. Por ejemplo, los distintos tipos de GPU `mgXc` no dan soporte a Trusted Compute. Si no habilita la confianza durante la creación del clúster pero desea hacerlo posteriormente, puede utilizar el [mandato](cs_cli_reference.html#cs_cluster_feature_enable) `ibmcloud ks feature-enable`. Una vez que habilita la confianza, no puede inhabilitarla posteriormente.

        Asegúrese de que desea suministrar una máquina nativa. Puesto que se factura mensualmente, si cancela la operación inmediatamente tras realizar un pedido por error, se le cobrará el mes completo.
        {:tip}

    2. Seleccione un tipo de máquina. El tipo de máquina define la cantidad de memoria, espacio de disco y CPU virtual que se configura en cada nodo trabajador y que está disponible para todos los contenedores. Los tipos de máquinas nativas y virtuales varían según la zona en la que se despliega el clúster. Después de crear el clúster, puede añadir distintos tipos de máquina añadiendo un nodo trabajador o una agrupación al clúster.

    3. Especifique el número de nodos trabajadores que necesita en el clúster. El número de nodos trabajadores que especifique se replica entre el número de zonas que ha seleccionado. Esto significa que si tiene 2 zonas y selecciona 3 nodos trabajadores, se suministran 6 nodos y en cada zona residen 3 nodos.

6. Dele un nombre exclusivo al clúster. **Nota**: si se cambia el nombre de dominio o el ID exclusivo asignado durante la creación, se impide que el nodo maestro de Kubernetes gestione el clúster.

7. Elija la versión del servidor de API de Kubernetes para el nodo maestro del clúster.

8. Pulse **Crear clúster**. Se crea una agrupación de nodos trabajadores con el número de nodos trabajadores que ha especificado. Verá el progreso del despliegue del nodo trabajador en el separador **Nodos trabajadores**. Cuando finalice el despliegue, podrá ver que el clúster está listo en el separador **Visión general**.

**¿Qué es lo siguiente?**

Cuando el clúster esté activo y en ejecución, puede realizar las siguientes tareas:

-   Distribuya los nodos trabajadores entre varias zonas [añadiendo una zona al clúster](#add_zone).
-   [Instalar las CLI para empezar a trabajar con el clúster.](cs_cli_install.html#cs_cli_install)
-   [Desplegar una app en el clúster.](cs_app.html#app_cli)
-   [Configure su propio registro privado en {{site.data.keyword.Bluemix_notm}} para almacenar y compartir imágenes de Docker con otros usuarios. ](/docs/services/Registry/index.html)
- Si tiene varias VLAN para un clúster, varias subredes en la misma VLAN o un clúster multizona, debe habilitar la [expansión de VLAN](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning) para la cuenta de infraestructura de IBM Cloud (SoftLayer) para que los nodos trabajadores puedan comunicarse entre sí en la red privada. Para llevar a cabo esta acción, necesita el [permiso de la infraestructura](cs_users.html#infra_access) **Red > Gestionar expansión de VLAN de la red**, o bien puede solicitar al propietario de la cuenta que lo habilite. Para comprobar si la expansión de VLAN ya está habilitada, utilice el [mandato](/docs/containers/cs_cli_reference.html#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get`. Si utiliza {{site.data.keyword.BluDirectLink}}, en su lugar debe utilizar una [función de direccionador virtual (VRF)](/docs/infrastructure/direct-link/subnet-configuration.html#more-about-using-vrf). Para habilitar la VRF, póngase en contacto con el representante de cuentas de la infraestructura de IBM Cloud (SoftLayer).
- Si tiene un cortafuegos, es posible que tenga que [abrir los puertos necesarios](cs_firewall.html#firewall) para utilizar los mandatos `ibmcloud`, `kubectl` o `calicotl`, para permitir el tráfico de salida desde el clúster o para permitir el tráfico de entrada para los servicios de red.
-  Clústeres con Kubernetes versión 1.10 o posterior: controle quién puede crear pods en el clúster con [políticas de seguridad de pod](cs_psp.html).

<br />


## Creación de clústeres con la CLI
{: #clusters_cli}

La finalidad del clúster de Kubernetes es definir un conjunto de recursos, nodos, redes y dispositivos de almacenamiento que mantengan la alta disponibilidad de las apps. Para poder desplegar una app, debe crear un clúster y establecer las definiciones de los nodos trabajadores en dicho clúster.
{:shortdesc}

Antes de empezar:
- Debe poseer una [cuenta de {{site.data.keyword.Bluemix_notm}}](https://console.bluemix.net/registration/) de pago según uso o de suscripción configurada para el [acceso al portafolio (SoftlLayer) de la infraestructura de IBM Cloud](cs_troubleshoot_clusters.html#cs_credentials). Puede crear un clúster gratuito para probar algunas de las funcionalidades durante 30 días, o crear clústeres estándares totalmente personalizables con el aislamiento de hardware que elija.
- [Asegúrese de que tiene los permisos necesarios mínimos en la infraestructura de IBM Cloud (SoftLayer) para suministrar un clúster estándar](cs_users.html#infra_access).
- Instale la CLI de {{site.data.keyword.Bluemix_notm}} y el [plugin de {{site.data.keyword.containerlong_notm}}](cs_cli_install.html#cs_cli_install).
- Si tiene varias VLAN para un clúster, varias subredes en la misma VLAN o un clúster multizona, debe habilitar la [expansión de VLAN](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning) para la cuenta de infraestructura de IBM Cloud (SoftLayer) para que los nodos trabajadores puedan comunicarse entre sí en la red privada. Para llevar a cabo esta acción, necesita el [permiso de la infraestructura](cs_users.html#infra_access) **Red > Gestionar expansión de VLAN de la red**, o bien puede solicitar al propietario de la cuenta que lo habilite. Para comprobar si la expansión de VLAN ya está habilitada, utilice el [mandato](/docs/containers/cs_cli_reference.html#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get`. Si utiliza {{site.data.keyword.BluDirectLink}}, en su lugar debe utilizar una [función de direccionador virtual (VRF)](/docs/infrastructure/direct-link/subnet-configuration.html#more-about-using-vrf). Para habilitar la VRF, póngase en contacto con el representante de cuentas de la infraestructura de IBM Cloud (SoftLayer).
- Decida entre un [clúster gratuito o estándar](cs_why.html#cluster_types). Con su clúster gratuito, dispone de un número limitado de días para probar las características de Kubernetes. Cree un clúster estándar para obtener más ventajas y controlar el rendimiento del clúster.
- Si crea un clúster estándar, puede [estimar el coste con la calculadora de precios ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://console.bluemix.net/pricing/configure/iaas/containers-kubernetes). **Nota**: Debe añadir una **Red dedicada (1 subred con 8 IP públicas portátiles)** para cada clúster a la estimación. Además, el coste del ancho de banda de red de salida varía según el uso y es un cargo adicional aparte de los cargos de la calculadora de precios.

Para crear un clúster:

1.  Inicie la sesión en la CLI de {{site.data.keyword.Bluemix_notm}}.

    1.  Inicie una sesión y escriba sus credenciales de {{site.data.keyword.Bluemix_notm}} cuando se le solicite.

        ```
        ibmcloud login
        ```
        {: pre}

        **Nota:** si tiene un ID federado, utilice `ibmcloud login --sso` para iniciar la sesión en la CLI de {{site.data.keyword.Bluemix_notm}}. Especifique su nombre de usuario y utilice el URL proporcionado en la salida de la CLI para recuperar el código de acceso de un solo uso. Sabe tiene un ID federado cuando el inicio de sesión falla sin el `--sso` y se lleva a cabo correctamente con la opción `--sso`.

    2. Si tiene varias cuentas de {{site.data.keyword.Bluemix_notm}}, seleccione la cuenta donde desea crear el clúster de Kubernetes.

    3.  Si desea crear o acceder a clústeres de Kubernetes en una región distinta de la región de {{site.data.keyword.Bluemix_notm}} seleccionada anteriormente, ejecute `ibmcloud ks region-set`.

3.  Cree un clúster. **Nota**: los clústeres estándar se pueden crear en cualquier región y zona disponible. No se pueden crear clústeres gratuitos en las regiones de EE.UU. este o AP norte y las zonas correspondientes, y no se puede seleccionar la zona.

    1.  **Clústeres estándares**: revise las zonas que están disponibles. Las zonas que se muestran dependen de la región de {{site.data.keyword.containerlong_notm}} en la que ha iniciado la sesión.

        **Nota**: para distribuir el clúster entre zonas, debe crear el clúster en una [zona con soporte multizona](cs_regions.html#zones).

        ```
        ibmcloud ks zones
        ```
        {: pre}

    2.  **Clústeres estándares**: Elija una zona y revise los tipos de máquinas disponibles en dicha zona. El tipo de máquina especifica los hosts de cálculo físicos o virtuales que están disponibles para cada nodo trabajador.

        -  Consulte el campo **Tipo de servidor** para elegir máquinas virtuales o físicas (nativas).
        -  **Virtual**: Las máquinas virtuales se facturan por horas y se suministran en hardware compartido o dedicado.
        -  **Físico**: Los servidores nativos se facturan de forma mensual y su suministro se realiza mediante interacción manual con la infraestructura de IBM Cloud (SoftLayer), por lo que puede tardar más de un día laborable en realizarse. Los servidores nativos son más apropiados para aplicaciones de alto rendimiento que necesitan más recursos y control de host.
        - **Máquinas físicas con Trusted Compute**: Para clústeres nativos que ejecutan la versión 1.9 o posterior de Kubernetes, también puede optar por habilitar [Trusted Compute](cs_secure.html#trusted_compute) para verificar que los nodos trabajadores nativos no se manipulan de forma indebida. Trusted Compute está disponible para determinados tipos de máquinas nativas. Por ejemplo, los distintos tipos de GPU `mgXc` no dan soporte a Trusted Compute. Si no habilita la confianza durante la creación del clúster pero desea hacerlo posteriormente, puede utilizar el [mandato](cs_cli_reference.html#cs_cluster_feature_enable) `ibmcloud ks feature-enable`. Una vez que habilita la confianza, no puede inhabilitarla posteriormente.
        -  **Tipos de máquina**: para decidir qué tipo de máquina desplegar, revise el núcleo, la memoria y las combinaciones de almacenamiento del [hardware del nodo trabajador disponible](cs_clusters_planning.html#shared_dedicated_node). Después de crear el clúster, puede añadir distintos tipos de máquina física o virtual mediante la [adición de una agrupación de nodos trabajadores](#add_pool).

           Asegúrese de que desea suministrar una máquina nativa. Puesto que se factura mensualmente, si cancela la operación inmediatamente tras realizar un pedido por error, se le cobrará el mes completo.
           {:tip}

        ```
        ibmcloud ks machine-types <zone>
        ```
        {: pre}

    3.  **Clústeres estándares**: Compruebe si ya existe una VLAN pública y privada en la infraestructura de IBM Cloud (SoftLayer) para esta cuenta.

        ```
        ibmcloud ks vlans <zone>
        ```
        {: pre}

        ```
        ID        Name                Number   Type      Router
        1519999   vlan   1355     private   bcr02a.dal10
        1519898   vlan   1357     private   bcr02a.dal10
        1518787   vlan   1252     public   fcr02a.dal10
        1518888   vlan   1254     public    fcr02a.dal10
        ```
        {: screen}

        Si ya existen una VLAN pública y privada, anote los direccionadores correspondientes. Los direccionadores de VLAN privadas siempre
empiezan por <code>bcr</code> (back-end router, direccionador de fondo) y los direccionadores de VLAN públicas siempre
empiezan por <code>fcr</code> (direccionador frontal). Al crear un clúster especificando las VLAN privadas y públicas, deben coincidir el número y la combinación de letras después de dichos prefijos. En la salida de ejemplo, se puede utilizar cualquier VLAN privada con cualquier VLAN pública porque todos los direccionadores incluyen `02a.dal10`.

        Los nodos trabajadores los debe conectar a una VLAN privada y, opcionalmente, puede conectarlos a una VLAN pública. **Nota**: Si los nodos trabajadores únicamente se configuran con una VLAN privada, debe configurar una solución alternativa para la conectividad de red. Para obtener más información, consulte [Planificación de redes de clúster solo privado](cs_network_cluster.html#private_vlan).

    4.  **Clústeres estándares y gratuitos**: Ejecute el mandato `cluster-create`. Puede elegir un clúster gratuito, que incluye un nodo trabajador configurado con 2 vCPU y 4 GB de memoria que se suprime de forma automática después de 30 días. Cuando se crea un clúster estándar, de forma predeterminada, los discos del nodo trabajador están cifrados, su hardware se comparte entre varios clientes de IBM y se factura por horas de uso. </br>Ejemplo para un clúster estándar. Especifique las opciones del clúster:

        ```
        ibmcloud ks cluster-create --zone dal10 --machine-type b2c.4x16 --hardware <shared_or_dedicated> --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID> --workers 3 --name <cluster_name> --kube-version <major.minor.patch> [--disable-disk-encrypt][--trusted]
        ```
        {: pre}

        Ejemplo para un clúster gratuito. Especifique el nombre del clúster:

        ```
        ibmcloud ks cluster-create --name my_cluster
        ```
        {: pre}

        <table>
        <caption>Componentes de cluster-create</caption>
        <thead>
        <th colspan=2><img src="images/idea.png" alt="Icono Idea"/> Descripción de los componentes de este mandato</th>
        </thead>
        <tbody>
        <tr>
        <td><code>cluster-create</code></td>
        <td>El mandato para crear un clúster en la organización de {{site.data.keyword.Bluemix_notm}}.</td>
        </tr>
        <tr>
        <td><code>--zone <em>&lt;zone&gt;</em></code></td>
        <td>**Clústeres estándares**: Sustituya <em>&lt;zone&gt;</em> por el ID de zona de {{site.data.keyword.Bluemix_notm}} donde desea crear el clúster. Las zonas disponibles dependen de la región de {{site.data.keyword.containerlong_notm}} en la que ha iniciado la sesión.<br></br>**Nota**: los nodos trabajadores de clúster se despliegan en esta zona. Para distribuir el clúster entre zonas, debe crear el clúster en una [zona con soporte multizona](cs_regions.html#zones). Después de que se haya creado el clúster, puede [añadir una zona al clúster](#add_zone).</td>
        </tr>
        <tr>
        <td><code>--machine-type <em>&lt;machine_type&gt;</em></code></td>
        <td>**Clústeres estándares**: Elija un tipo de máquina. Puede desplegar los nodos trabajadores como máquinas virtuales en hardware dedicado o compartido, o como máquinas físicas en servidores nativos. Los tipos de máquinas físicas y virtuales varían según la zona en la que se despliega el clúster. Para obtener más información, consulte la documentación del [mandato](cs_cli_reference.html#cs_machine_types) `ibmcloud ks machine-type`. Para los clústeres gratuitos, no tiene que definir el tipo de máquina.</td>
        </tr>
        <tr>
        <td><code>--hardware <em>&lt;shared_or_dedicated&gt;</em></code></td>
        <td>**Clústeres estándares, sólo virtuales**: El nivel de aislamiento del hardware del nodo trabajador. Utilice el valor dedicated para tener recursos físicos disponibles dedicados solo a usted, o shared para permitir que los recursos físicos se compartan con otros clientes de IBM. El valor predeterminado es shared. Este valor es opcional para clústeres estándares y no está disponible para clústeres gratuitos.</td>
        </tr>
        <tr>
        <td><code>--public-vlan <em>&lt;public_vlan_id&gt;</em></code></td>
        <td><ul>
          <li>**Clústeres gratuitos**: No tiene que definir una VLAN pública. El clúster gratuito se conecta automáticamente a una VLAN pública propiedad de IBM.</li>
          <li>**Clústeres estándares**: Si ya tiene una VLAN pública configurada en su cuenta de infraestructura de IBM Cloud (SoftLayer) para esta zona, escriba el ID de la VLAN pública. Si desea conectar los nodos trabajadores solo a una VLAN privada, no especifique esta opción. **Nota**: Si los nodos trabajadores únicamente se configuran con una VLAN privada, debe configurar una solución alternativa para la conectividad de red. Para obtener más información, consulte [Planificación de redes de clúster solo privado](cs_network_cluster.html#private_vlan).<br/><br/>
          <strong>Nota</strong>: Los direccionadores de VLAN privadas siempre empiezan por <code>bcr</code> (back-end router, direccionador de fondo) y los direccionadores de VLAN públicas siempre empiezan por <code>fcr</code> (direccionador frontal). Al crear un clúster especificando las VLAN privadas y públicas, deben coincidir el número y la combinación de letras después de dichos prefijos.</li>
        </ul></td>
        </tr>
        <tr>
        <td><code>--private-vlan <em>&lt;private_vlan_id&gt;</em></code></td>
        <td><ul><li>**Clústeres gratuitos**: No tiene que definir una VLAN privada. El clúster gratuito se conecta automáticamente a una VLAN privada propiedad de IBM.</li><li>**Clústeres estándares**: Si ya tiene una VLAN privada configurada en su cuenta de infraestructura de IBM Cloud (SoftLayer) para esta zona, escriba el ID de la VLAN privada. Si no tiene una VLAN privada en la ubicación, no especifique esta opción. {{site.data.keyword.containerlong_notm}} crea automáticamente una VLAN privada.<br/><br/><strong>Nota</strong>: Los direccionadores de VLAN privadas siempre empiezan por <code>bcr</code> (back-end router, direccionador de fondo) y los direccionadores de VLAN públicas siempre empiezan por <code>fcr</code> (direccionador frontal). Al crear un clúster especificando las VLAN privadas y públicas, deben coincidir el número y la combinación de letras después de dichos prefijos.</li></ul></td>
        </tr>
        <tr>
        <td><code>--name <em>&lt;name&gt;</em></code></td>
        <td>**Clústeres estándares y gratuitos**: Sustituya <em>&lt;name&gt;</em> por el nombre del clúster. El nombre debe empezar por una letra, puede contener letras, números, guiones (-) y debe tener 35 caracteres como máximo. El nombre del clúster y la región en la que el clúster se despliega forman el nombre de dominio completo para el subdominio de Ingress. Para garantizar que el subdominio de Ingress es exclusivo dentro de una región, el nombre del clúster podría ser truncado y añadírsele un valor aleatorio dentro del nombre de dominio de Ingress.
</td>
        </tr>
        <tr>
        <td><code>--workers <em>&lt;number&gt;</em></code></td>
        <td>**Clústeres estándares**: El número de nodos trabajadores que desea incluir en el clúster. Si no se especifica la opción <code>--workers</code>, se crea 1 nodo trabajador.</td>
        </tr>
        <tr>
          <td><code>--kube-version <em>&lt;major.minor.patch&gt;</em></code></td>
          <td>**Clústeres estándares**: La versión de Kubernetes del nodo maestro del clúster. Este valor es opcional. Cuando no se especifica la versión, el clúster se crea con el valor predeterminado de las versiones de Kubernetes soportadas. Para ver todas las versiones disponibles, ejecute <code>ibmcloud ks kube-versions</code>.
</td>
        </tr>
        <tr>
        <td><code>--disable-disk-encrypt</code></td>
        <td>**Clústeres estándares y gratuitos**: Los nodos trabajadores tienen cifrado de disco de forma predeterminada; [más información](cs_secure.html#encrypted_disk). Si desea inhabilitar el cifrado, incluya esta opción.</td>
        </tr>
        <tr>
        <td><code>--trusted</code></td>
        <td>**Clústeres nativos estándares**: Habilite [Trusted Compute](cs_secure.html#trusted_compute) para verificar que los nodos trabajadores nativos no se manipulan de forma indebida. Trusted Compute está disponible para determinados tipos de máquinas nativas. Por ejemplo, los distintos tipos de GPU `mgXc` no dan soporte a Trusted Compute. Si no habilita la confianza durante la creación del clúster pero desea hacerlo posteriormente, puede utilizar el [mandato](cs_cli_reference.html#cs_cluster_feature_enable) `ibmcloud ks feature-enable`. Una vez que habilita la confianza, no puede inhabilitarla posteriormente.</td>
        </tr>
        </tbody></table>

4.  Verifique que ha solicitado la creación del clúster.

    ```
    ibmcloud ks clusters
    ```
    {: pre}

    **Nota:** Para máquinas virtuales, se puede tardar varios minutos en pedir las máquinas de nodo trabajador y en configurar y suministrar el clúster en la cuenta. El suministro de las máquinas físicas nativas se realiza mediante interacción manual con la infraestructura de IBM Cloud (SoftLayer), por lo que puede tardar más de un día laborable en realizarse.

    Una vez completado el suministro del clúster, el estado del clúster pasa a ser **deployed**.

    ```
    Name         ID                                   State      Created          Workers   Zone   Version
    my_cluster   paf97e8843e29941b49c598f516de72101   deployed   20170201162433   1         mil01      1.10.7
    ```
    {: screen}

5.  Compruebe el estado de los nodos trabajadores.

    ```
    ibmcloud ks workers <cluster_name_or_ID>
    ```
    {: pre}

    Cuando los nodos trabajadores están listos, el estado pasa a **normal** y el estado es **Ready**. Cuando el estado del nodo sea **Preparado**, podrá acceder al clúster.

    **Nota:** A cada nodo trabajador se la asigna un ID exclusivo y un nombre de dominio que no se debe cambiar de forma manual después de haber creado el clúster. Si se cambia el nombre de dominio o el ID se impide que el maestro de Kubernetes gestione el clúster.

    ```
    ID                                                 Public IP       Private IP      Machine Type   State    Status   Zone   Version
    kube-mil01-paf97e8843e29941b49c598f516de72101-w1   169.xx.xxx.xxx  10.xxx.xx.xxx   free           normal   Ready    mil01      1.10.7
    ```
    {: screen}

6.  Defina el clúster que ha creado como contexto para esta sesión. Siga estos pasos de configuración cada vez que de trabaje con el clúster.
    1.  Obtenga el mandato para establecer la variable de entorno y descargar los archivos de configuración de Kubernetes.

        ```
        ibmcloud ks cluster-config <cluster_name_or_ID>
        ```
        {: pre}

        Cuando termine la descarga de los archivos de configuración, se muestra un mandato que puede utilizar para establecer la vía de acceso al archivo de configuración de
Kubernetes como variable de entorno.

        Ejemplo para OS X:

        ```
        export KUBECONFIG=/Users/<user_name>/.bluemix/plugins/container-service/clusters/mycluster/kube-config-prod-dal10-mycluster.yml
        ```
        {: screen}

    2.  Copie y pegue el mandato que se muestra en el terminal para definir la variable de entorno `KUBECONFIG`.
    3.  Compruebe que la variable de entorno `KUBECONFIG` se haya establecido correctamente.

        Ejemplo para OS X:

        ```
        echo $KUBECONFIG
        ```
        {: pre}

        Salida:

        ```
        /Users/<user_name>/.bluemix/plugins/container-service/clusters/mycluster/kube-config-prod-dal10-mycluster.yml

        ```
        {: screen}

7.  Inicie el panel de control de Kubernetes con el puerto predeterminado `8001`.
    1.  Establezca el proxy con el número de puerto predeterminado.

        ```
        kubectl proxy
        ```
        {: pre}

        ```
        Starting to serve on 127.0.0.1:8001
        ```
        {: screen}

    2.  Abra el siguiente URL en un navegador web para ver el panel de control de Kubernetes.

        ```
        http://localhost:8001/ui
        ```
        {: codeblock}


**¿Qué es lo siguiente?**

-   Distribuya los nodos trabajadores entre varias zonas [añadiendo una zona al clúster](#add_zone).
-   [Desplegar una app en el clúster.](cs_app.html#app_cli)
-   [Gestionar el clúster con la línea de mandatos de `kubectl`. ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://kubernetes.io/docs/reference/kubectl/overview/)
-   [Configure su propio registro privado en {{site.data.keyword.Bluemix_notm}} para almacenar y compartir imágenes de Docker con otros usuarios. ](/docs/services/Registry/index.html)
- Si tiene un cortafuegos, es posible que tenga que [abrir los puertos necesarios](cs_firewall.html#firewall) para utilizar los mandatos `ibmcloud`, `kubectl` o `calicotl`, para permitir el tráfico de salida desde el clúster o para permitir el tráfico de entrada para los servicios de red.
-  Clústeres con Kubernetes versión 1.10 o posterior: controle quién puede crear pods en el clúster con [políticas de seguridad de pod](cs_psp.html).

<br />



## Adición de nodos trabajadores y de zonas a clústeres
{: #add_workers}

Para aumentar la disponibilidad de las apps, puede añadir nodos trabajadores a una zona existente o a varias zonas existentes en el clúster. Para ayudar a proteger las apps frente a anomalías de zona, puede añadir zonas al clúster.
{:shortdesc}

Cuando se crea un clúster, los nodos trabajadores se suministran en una agrupación de nodos trabajadores. Después de la creación del clúster, puede añadir más nodos trabajadores a una agrupación cambiando el tamaño de la agrupación o añadiendo más agrupaciones de nodos trabajadores. De forma predeterminada, la agrupación de nodos trabajadores existe en una zona. Los clústeres que tienen una agrupación de nodos trabajadores en una sola zona se denominan clústeres de una sola zona. Cuando se añaden más zonas al clúster, la agrupación de nodos trabajadores existe entre las zonas. Los clústeres que tienen una agrupación de trabajadores distribuida entre más de una zona se denominan clústeres multizona.

Si tiene un clúster multizona, mantenga equilibrados los recursos de los nodos trabajadores. Asegúrese de que todas las agrupaciones de nodos trabajadores estén distribuidas entre las mismas zonas y añada o elimine nodos trabajadores cambiando el tamaño de las agrupaciones en lugar de añadir nodos individuales.
{: tip}

En las secciones siguientes se muestra cómo:
  * [Añadir nodos trabajadores cambiando el tamaño de una agrupación de nodos trabajadores existente en el clúster](#resize_pool)
  * [Añadir nodos trabajadores añadiendo al clúster una agrupación de nodos trabajadores](#add_pool)
  * [Añadir una zona al clúster y replicar los nodos trabajadores de las agrupaciones de nodos trabajadores entre varias zonas](#add_zone)
  * [En desuso: añada al clúster un nodo trabajador autónomo](#standalone)


### Adición de nodos trabajadores mediante el redimensionando de una agrupación de nodos trabajadores existente
{: #resize_pool}

Puede añadir o reducir el número de nodos trabajadores del clúster redimensionando una agrupación de nodos trabajadores existente, independientemente de si la agrupación de nodos trabajadores está en una zona o está distribuida entre varias zonas.
{: shortdesc}

Por ejemplo, supongamos que tiene un clúster con una agrupación de nodos trabajadores que tiene tres nodos trabajadores por zona.
* Si el clúster está en una sola zona y existe en `dal10`, la agrupación de nodos trabajadores tiene tres nodos trabajadores en `dal10`. El clúster tiene un total de tres nodos trabajadores.
* Si el clúster es multizona y existe en `dal10` y en `dal12`, la agrupación de nodos trabajadores tiene tres nodos trabajadores en `dal10` y tres nodos trabajadores en `dal12`. El clúster tiene un total de seis nodos trabajadores.

Para las agrupaciones de nodos trabajadores nativas, tenga en cuenta que la facturación es mensual. El redimensionamiento de la agrupación afectará a sus costes mensuales.
{: tip}

Para redimensionar la agrupación de nodos trabajadores, cambie el número de nodos trabajadores que la agrupación de nodos trabajadores despliega en cada zona:

1. Obtenga el nombre de la agrupación de nodos trabajadores que desea redimensionar.
    ```
    ibmcloud ks worker-pools --cluster <cluster_name_or_ID>
    ```
    {: pre}

2. Redimensione la agrupación de nodos trabajadores designando el número de nodos trabajadores que desea desplegar en cada zona. El valor mínimo es 1.
    ```
    ibmcloud ks worker-pool-resize --cluster <cluster_name_or_ID> --worker-pool <pool_name>  --size-per-zone <number_of_workers_per_zone>
    ```
    {: pre}

3. Verifique que se ha cambiado el tamaño de la agrupación de nodos trabajadores.
    ```
    ibmcloud ks workers <cluster_name_or_ID> --worker-pool <pool_name>
    ```
    {: pre}

    Salida de ejemplo correspondiente a una agrupación de nodos trabajadores que está en dos zonas, `dal10` y `dal12`, y que se redimensiona a dos nodos trabajadores por zona:
    ```
    ID                                                 Public IP        Private IP      Machine Type      State    Status  Zone    Version
    kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w7   169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal10   1.8.6_1504
    kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w8   169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal10   1.8.6_1504
    kube-dal12-crb20b637238ea471f8d4a8b881aae4962-w9   169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal12   1.8.6_1504
    kube-dal12-crb20b637238ea471f8d4a8b881aae4962-w10  169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal12   1.8.6_1504
    ```
    {: screen}

### Adición de nodos trabajadores mediante la creación de una nueva agrupación de nodos trabajadores
{: #add_pool}

Puede añadir nodos trabajadores a un clúster creando una nueva agrupación de nodos trabajadores.
{:shortdesc}

1. Obtenga una lista de las zonas disponibles y elija la zona en la que desea desplegar los nodos trabajadores en la agrupación de nodos trabajadores. Si tiene intención de distribuir los nodos trabajadores entre varias zonas, elija una [zona con soporte multizona](cs_regions.html#zones).
   ```
   ibmcloud ks zones
   ```
   {: pre}

2. Para cada zona, obtenga una lista de las VLAN privadas y públicas disponibles. Anote las VLAN privadas y públicas que desea utilizar. Si no tiene una VLAN privada o pública, la VLAN se crea automáticamente cuando añade una zona a la agrupación de nodos trabajadores.
   ```
   ibmcloud ks vlans <zone>
   ```
   {: pre}

3. Cree una agrupación de nodos trabajadores. Para ver las opciones de tipo de máquina, consulte la documentación del [mandato `machine-types`](cs_cli_reference.html#cs_machine_types).
   ```
   ibmcloud ks worker-pool-create --name <pool_name> --cluster <cluster_name_or_ID> --machine-type <machine_type> --size-per-zone <number_of_workers_per_zone>
   ```
   {: pre}

4. Verifique que la agrupación de nodos trabajadores se ha creado.
   ```
   ibmcloud ks worker-pools --cluster <cluster_name_or_ID>
   ```
   {: pre}

5. De forma predeterminada, la adición de una agrupación de nodos trabajadores crea una agrupación sin zonas. Para desplegar nodos trabajadores en una zona, debe añadir zonas a la agrupación de nodos trabajadores. Si desea distribuir los nodos trabajadores en varias zonas, repita este mandato con otra zona con soporte multizona.  
   ```
   ibmcloud ks zone-add --zone <zone> --cluster <cluster_name_or_ID> --worker-pools <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
   ```
   {: pre}

6. Verifique que los nodos trabajadores se suministran en la zona que ha añadido.
   ```
   ibmcloud ks workers <cluster_name_or_ID> --worker-pool <pool_name>
   ```
   {: pre}

   Salida de ejemplo:
   ```
   ID                                                 Public IP        Private IP      Machine Type      State    Status  Zone    Version
   kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w7   169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal10   1.8.6_1504
   kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w8   169.xx.xxx.xxx   10.xxx.xx.xxx   b2c.4x16          normal   Ready   dal10   1.8.6_1504
   ```
   {: screen}

### Adición de nodos trabajadores mediante la adición de una zona a una agrupación de nodos trabajadores
{: #add_zone}

Puede distribuir el clúster entre varias zonas dentro de una región mediante la adición de una zona a su agrupación de nodos trabajadores existente.
{:shortdesc}

Cuando se añade una zona a una agrupación de nodos trabajadores, los nodos trabajadores definidos en la agrupación de nodos trabajadores se suministran en la nueva zona y se tienen en cuenta para la futura planificación de la carga de trabajo. {{site.data.keyword.containerlong_notm}} añade automáticamente a cada nodo trabajador la etiqueta `failure-domain.beta.kubernetes.io/region` correspondiente a la región y la etiqueta `failure-domain.beta.kubernetes.io/zone` correspondiente a la zona. El planificador de Kubernetes utiliza estas etiquetas para distribuir los pods entre zonas de la misma región.

**Nota**: si tiene varias agrupaciones de nodos trabajadores en el clúster, añada la zona a todas ellas para que los nodos trabajadores se distribuya uniformemente en el clúster.

Antes de empezar:
*  Para añadir una zona a la agrupación de nodos trabajadores, la agrupación de nodos trabajadores debe estar en una [zona con soporte multizona](cs_regions.html#zones). Si la agrupación de nodos trabajadores no está en una zona con soporte multizona, tenga en cuenta la posibilidad de [crear una nueva agrupación de nodos trabajadores](#add_pool).
*  Si tiene varias VLAN para un clúster, varias subredes en la misma VLAN o un clúster multizona, debe habilitar la [expansión de VLAN](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning) para la cuenta de infraestructura de IBM Cloud (SoftLayer) para que los nodos trabajadores puedan comunicarse entre sí en la red privada. Para llevar a cabo esta acción, necesita el [permiso de la infraestructura](cs_users.html#infra_access) **Red > Gestionar expansión de VLAN de la red**, o bien puede solicitar al propietario de la cuenta que lo habilite. Para comprobar si la expansión de VLAN ya está habilitada, utilice el [mandato](/docs/containers/cs_cli_reference.html#cs_vlan_spanning_get) `ibmcloud ks vlan-spanning-get`. Si utiliza {{site.data.keyword.BluDirectLink}}, en su lugar debe utilizar una [función de direccionador virtual (VRF)](/docs/infrastructure/direct-link/subnet-configuration.html#more-about-using-vrf). Para habilitar la VRF, póngase en contacto con el representante de cuentas de la infraestructura de IBM Cloud (SoftLayer).

Para añadir una zona con nodos trabajadores a la agrupación de nodos trabajadores:

1. Obtenga una lista de las zonas disponibles y elija la zona que desea añadir a la agrupación de nodos trabajadores. La zona que elija debe ser una zona con soporte multizona.
   ```
   ibmcloud ks zones
   ```
   {: pre}

2. Obtenga una lista de las VLAN de dicha zona. Si no tiene una VLAN privada o pública, la VLAN se crea automáticamente cuando añade una zona a la agrupación de nodos trabajadores.
   ```
   ibmcloud ks vlans <zone>
   ```
   {: pre}

3. Obtenga una lista de las agrupaciones de nodos trabajadores de su clúster y anote sus nombres.
   ```
   ibmcloud ks worker-pools --cluster <cluster_name_or_ID>
   ```
   {: pre}

4. Añada la zona a la agrupación de nodos trabajadores. Si tiene varias agrupaciones de nodos trabajadores, añada la zona a todas las agrupaciones de nodos trabajadores de forma que el clúster quede equilibrado en todas las zonas. Sustituya `<pool1_id_or_name,pool2_id_or_name,...>` por los nombres de todas sus agrupaciones de nodos trabajadores en una lista separada por comas. </br>**Nota:** debe existir una VLAN privada y pública para que pueda añadir una zona a varias agrupaciones de nodos trabajadores. Si no tiene una VLAN privada y pública en dicha zona, añada primero la zona a una agrupación de nodos trabajadores para que se cree una VLAN privada y pública. A continuación, puede añadir la zona a otras agrupaciones de nodos trabajadores especificando la VLAN privada y pública que se ha creado automáticamente.

   Si desea utilizar distintas VLAN para distintas agrupaciones de nodos trabajadores, repita este mandato para cada VLAN y sus agrupaciones de nodos trabajadores correspondientes. Los nuevos nodos trabajadores se añaden a las VLAN que especifique, pero las VLAN de los nodos trabajadores existentes no se modifican.
   {: tip}
   ```
   ibmcloud ks zone-add --zone <zone> --cluster <cluster_name_or_ID> --worker-pools <pool1_name,pool2_name,...> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
   ```
   {: pre}

5. Verifique que la zona se ha añadido al clúster. Busque la zona añadidos en el campo **Worker zones** de la información de salida. Observe que el número total de nodos trabajadores del campo **Workers** ha aumentado ya que se han suministrado nuevos nodos trabajadores en la zona que se ha añadido.
    ```
    ibmcloud ks cluster-get <cluster_name_or_ID>
    ```
    {: pre}

    Salida de ejemplo:
    ```
    Name:               mycluster
    ID:                 a20a637238aa471f8d4a8b881aaa4988
    State:              normal
    Created:            2018-04-19T01:49:22+0000
    Master zone:    us-south
    Worker zones:       dal10,dal12
    Master URL:         https://169.xx.xxx.xxx:21111
    Ingress subdomain:  ...
    Ingress secret:     ...
    Workers:            6
    Version:            1.8.6_1504
    ```
    {: screen}  

### En desuso: Adición de nodos trabajadores autónomos
{: #standalone}

Si tiene un clúster que se ha creado antes de la introducción de las agrupaciones de nodos trabajadores, puede utilizar los mandatos en desuso para añadir nodos trabajadores autónomos.
{: shortdesc}

**Nota:** si tiene un clúster que se ha creado después de que se hayan incorporado las agrupaciones de nodos trabajadores, no puede añadir nodos trabajadores autónomos. En su lugar, puede [crear una agrupación de nodos trabajadores](#add_pool), [redimensionar la agrupación de nodos trabajadores existente](#resize_pool) o [añadir una zona a una agrupación de nodos trabajadores](#add_zone) para añadir nodos trabajadores al clúster.

1. Obtenga una lista de las zonas disponibles y elija la zona en la desea añadir nodos trabajadores.
   ```
   ibmcloud ks zones
   ```
   {: pre}

2. Obtenga una lista de las VLAN disponibles en dicha zona y anote sus ID.
   ```
   ibmcloud ks vlans <zone>
   ```
   {: pre}

3. Obtenga una lista de los tipos de máquina de dicha zona.
   ```
   ibmcloud ks machine-types <zone>
   ```
   {: pre}

4. Añada nodos trabajadores autónomos al clúster.
   ```
   ibmcloud ks worker-add --cluster <cluster_name_or_ID> --number <number_of_worker_nodes> --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID> --machine-type <machine_type> --hardware <shared_or_dedicated>
   ```
   {: pre}

5. Verifique que los nodos trabajadores se han creado.
   ```
   ibmcloud ks workers <cluster_name_or_ID>
   ```
   {: pre}



## Visualización de estados de clúster
{: #states}

Revise el estado de un clúster de Kubernetes para obtener información sobre la disponibilidad y la capacidad del clúster, y posibles problemas que puedan haberse producido.
{:shortdesc}

Para ver información sobre un clúster específico, como sus zonas, el URL maestro, el subdominio de Ingress, la versión, el propietario y el panel de control de supervisión, utilice el [mandato](cs_cli_reference.html#cs_cluster_get) `ibmcloud ks cluster-get <cluster_name_or_ID>`. Incluya el distintivo `--showResources` para ver más recursos de clúster, como complementos para pods de almacenamiento o VLAN de subred para IP públicas y privadas.

Puede ver el estado actual del clúster ejecutando el mandato `ibmcloud ks clusters` y localizando el campo **State**. Para resolver el clúster y los nodos de trabajador, consulte [Resolución de problemas de clústeres](cs_troubleshoot.html#debug_clusters).

<table summary="Cada fila de la tabla se debe leer de izquierda a derecha, con el estado del clúster en la columna uno y una descripción en la columna dos.">
<caption>Estados de clúster</caption>
   <thead>
   <th>Estado del clúster</th>
   <th>Descripción</th>
   </thead>
   <tbody>
<tr>
   <td>Terminado anormalmente</td>
   <td>El usuario ha solicitado la supresión del clúster antes de desplegar el maestro de Kubernetes. Una vez realizada la supresión del clúster, el clúster se elimina del panel de control. Si el clúster está bloqueado en este estado durante mucho tiempo, abra una [incidencia de soporte de {{site.data.keyword.Bluemix_notm}}](cs_troubleshoot.html#ts_getting_help).</td>
   </tr>
 <tr>
     <td>Crítico</td>
     <td>No se puede acceder al maestro de Kubernetes o todos los nodos trabajadores del clúster están inactivos. </td>
    </tr>
   <tr>
     <td>Error al suprimir</td>
     <td>No se puede suprimir el maestro de Kubernetes o al menos un nodo trabajador.  </td>
   </tr>
   <tr>
     <td>Suprimido</td>
     <td>El clúster se ha suprimido pero todavía no se ha eliminado del panel de control. Si el clúster está bloqueado en este estado durante mucho tiempo, abra una [incidencia de soporte de {{site.data.keyword.Bluemix_notm}}](cs_troubleshoot.html#ts_getting_help). </td>
   </tr>
   <tr>
   <td>Suprimiendo</td>
   <td>El clúster se está suprimiendo y la infraestructura del clúster se está desmontando. No puede acceder al clúster.  </td>
   </tr>
   <tr>
     <td>Error al desplegar</td>
     <td>El despliegue del maestro de Kubernetes no se ha podido realizar. No puede resolver este estado. Póngase en contacto con el soporte de IBM Cloud abriendo una [incidencia de soporte de {{site.data.keyword.Bluemix_notm}}](cs_troubleshoot.html#ts_getting_help).</td>
   </tr>
     <tr>
       <td>Despliegue</td>
       <td>El maestro de Kubernetes no está completamente desplegado. No puede acceder a su clúster. Espere hasta que el clúster se haya desplegado por completo para revisar el estado del clúster.</td>
      </tr>
      <tr>
       <td>Normal</td>
       <td>Todos los nodos trabajadores de un clúster están activos y en ejecución. Puede acceder al clúster y desplegar apps en el clúster. Este estado se considera correcto y no requiere ninguna acción por su parte. **Nota**: Aunque los nodos trabajadores podrían poseer un estado normal, otros recursos de infraestructura como, por ejemplo la [red](cs_troubleshoot_network.html) y el [almacenamiento](cs_troubleshoot_storage.html), podrían requerir su atención.</td>
    </tr>
      <tr>
       <td>Pendiente</td>
       <td>El maestro de Kubernetes está desplegado. Los nodos trabajadores se están suministrando y aún no están disponibles en el clúster. Puede acceder al clúster, pero no puede desplegar apps en el clúster.  </td>
     </tr>
   <tr>
     <td>Solicitado</td>
     <td>Se ha enviado una solicitud para crear el clúster y pedir la infraestructura para el maestro de Kubernetes y los nodos trabajadores. Cuando se inicia el despliegue del clúster, el estado del clúster cambia a <code>Desplegando</code>. Si el clúster está bloqueado en el estado <code>Solicitado</code> durante mucho tiempo, abra una [incidencia de soporte de {{site.data.keyword.Bluemix_notm}}](cs_troubleshoot.html#ts_getting_help). </td>
   </tr>
   <tr>
     <td>Actualizando</td>
     <td>El servidor de API de Kubernetes que se ejecuta en el maestro de Kubernetes está siendo actualizado a una versión nueva de API de Kubernetes. Durante la actualización, no puede acceder ni cambiar el clúster. Los nodos trabajadores, apps y recursos que el usuario despliega no se modifican y continúan en ejecución. Espere a que la actualización se complete para revisar el estado del clúster. </td>
   </tr>
    <tr>
       <td>Aviso</td>
       <td>Al menos un nodo trabajador del clúster no está disponible, pero los otros nodos trabajadores están disponibles y pueden asumir la carga de trabajo. </td>
    </tr>
   </tbody>
 </table>


<br />


## Eliminación de clústeres
{: #remove}

Los clústeres gratuitos y estándares que se crean con una cuenta Pago según uso se deben eliminar de forma manual cuando dejan de necesitarse para que dejen de consumir recursos.
{:shortdesc}

**Aviso:**
  - No se crean copias de seguridad del clúster ni de los datos del almacén persistente. La supresión de un clúster o del almacenamiento persistente es irreversible y no se puede deshacer.
  - Cuando se elimina un clúster, también se eliminan todas las subredes suministradas de forma automática al crear el clúster con el mandato `ibmcloud ks cluster-subnet-create`. Sin embargo, si ha añadido de forma manual subredes existentes al clúster con el mandato `ibmcloud ks cluster-subnet-add`, estas subredes no se eliminan de su cuenta (SoftLayer) de la infraestructura IBM Cloud y podrá reutilizarlas en otros clústeres.

Antes de empezar:
* Anote el ID de clúster. Podría necesitar el ID de clúster para investigar y eliminar recursos (Softlayer) de la infraestructura IBM Cloud que no se suprimen de forma automática al suprimir su clúster.
* Si desea suprimir los datos en el almacenamiento persistente, [debe comprender las opciones de supresión](cs_storage_remove.html#cleanup).

Para eliminar un clúster:

-   Desde la interfaz gráfica de usuario de {{site.data.keyword.Bluemix_notm}}
    1.  Seleccione el clúster y pulse **Suprimir** en el menú **Más acciones...**.

-   Desde la CLI de {{site.data.keyword.Bluemix_notm}}
    1.  Liste los clústeres disponibles.

        ```
        ibmcloud ks clusters
        ```
        {: pre}

    2.  Suprima el clúster.

        ```
        ibmcloud ks cluster-rm <cluster_name_or_ID>
        ```
        {: pre}

    3.  Siga las indicaciones y decidir si desea suprimir los recursos del clúster (contenedores, pods, servicios enlazados, almacenamiento persistente y secretos).
      - **Almacenamiento persistente**: El almacenamiento persistente proporciona alta disponibilidad a los datos. Si ha creado una reclamación de volumen persistente utilizando una [compartición de archivo existente](cs_storage_file.html#existing_file), no puede suprimir la compartición de archivo al suprimir el clúster. Suprima manualmente la compartición de archivo desde el portafolio de infraestructura de IBM Cloud (SoftLayer).

          **Nota**: Debido al ciclo de facturación mensual, una reclamación de volumen persistente no se puede suprimir el último día del mes. Si suprime la reclamación de volumen persistente el último día del mes, la supresión permanece pendiente hasta el principio del siguiente mes.

Pasos siguientes:
- Después de que deje de aparecer en la lista de clústeres disponibles, al ejecutar el mandato `ibmcloud ks clusters`, podrá reutilizar el nombre de un clúster eliminado.
- Si ha conservado las subredes, puede [reutilizarlas en un nuevo clúster](cs_subnets.html#custom) o suprimirlas de forma manual más tarde de su portafolio (SoftLayer) de la infraestructura IBM Cloud.
- Si ha conservado el almacenamiento persistente, puede [suprimir el almacenamiento](cs_storage_remove.html#cleanup) más tarde a través el panel de control (SoftLayer) de la infraestructura IBM Cloud en la interfaz gráfica de usuario de {{site.data.keyword.Bluemix_notm}}.
