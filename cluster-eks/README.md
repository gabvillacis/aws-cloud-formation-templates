# Plantilla AWS CloudFormation para EKS

## Descripción

Esta plantilla de AWS CloudFormation está diseñada para desplegar un clúster de Amazon EKS (Elastic Kubernetes Service) con una configuración detallada de VPC. La plantilla incluye configuraciones de subredes públicas y privadas, un NAT Gateway, tablas de rutas, un grupo de nodos gestionado, y el complemento VPC CNI para EKS.

## Parámetros de la plantilla

La plantilla incluye los siguientes parámetros configurables, que permiten personalizar la red y las subredes del clúster:

- **VpcBlock**: Define el rango CIDR de la VPC. Este debe ser un rango CIDR privado válido según el estándar RFC 1918. Por defecto, está configurado como `192.168.0.0/16`.
- **PublicSubnet01Block**: Define el rango CIDR para la primera subred pública dentro de la VPC. Por defecto, `192.168.0.0/18`.
- **PublicSubnet02Block**: Define el rango CIDR para la segunda subred pública dentro de la VPC. Por defecto, `192.168.64.0/18`.
- **PrivateSubnet01Block**: Define el rango CIDR para la primera subred privada dentro de la VPC. Por defecto, `192.168.128.0/18`.
- **PrivateSubnet02Block**: Define el rango CIDR para la segunda subred privada dentro de la VPC. Por defecto, `192.168.192.0/18`.

Estos parámetros permiten que la plantilla sea reutilizable y adaptable a distintas configuraciones de red.

## Recursos desplegados

La plantilla crea y configura los siguientes recursos en AWS:

### **VPC y configuración de red**

1. **VPC**: Una red virtual que abarca todo el clúster.
2. **Internet Gateway**: Permite la conectividad a Internet para las subredes públicas.
3. **Subredes públicas y privadas**:
   - Dos subredes públicas para balanceadores de carga y otros servicios públicos.
   - Dos subredes privadas para los nodos del clúster.
4. **NAT Gateways**: Dos NAT Gateways, uno por zona de disponibilidad, para permitir que los nodos privados accedan a Internet.
5. **Tablas de rutas**:
   - Una tabla de rutas para las subredes públicas, con una ruta a través del Internet Gateway.
   - Tablas de rutas privadas con rutas hacia los NAT Gateways.

### **Seguridad**

1. **Grupos de seguridad**:
   - **ClusterSecurityGroup**: Controla la comunicación entre el plano de control del clúster y los nodos de trabajo.
   - **NodeSecurityGroup**: Permite la comunicación entre nodos y servicios DNS.

### **EKS Cluster**

1. **Clúster EKS**:
   - Nombre: `eks-devops`
   - Subredes asociadas: Incluye todas las subredes públicas y privadas.
   - Grupo de seguridad asociado al plano de control.
2. **Rol IAM del clúster**: Asigna el rol `AmazonEKSClusterPolicy` para gestionar los permisos del clúster.

### **Grupo de Nodos Gestionados**

1. **Node Group**:
   - Tamaño inicial: 2 nodos (mínimo) y un máximo de 4.
   - Tipo de instancia: `t3.medium`.
   - Sistema operativo: Amazon Linux 2.
   - Tamaño del disco: 20 GB.
   - Rol IAM asociado con políticas de acceso a EKS y Amazon ECR.

### **Complementos**

1. **VPC Controller Addon**: Complemento de CNI (Container Network Interface) para gestionar la red dentro del clúster.

## Salidas (Outputs)

La plantilla genera las siguientes salidas, útiles para la gestión y operación del clúster:

- **SubnetIds**: IDs de las subredes públicas y privadas.
- **SecurityGroups**: Grupo de seguridad para la comunicación del plano de control con los nodos.
- **NodeSecurityGroupId**: ID del grupo de seguridad asociado a los nodos.
- **VpcId**: ID de la VPC creada.
- **PublicSubnet01Id y PublicSubnet02Id**: IDs de las subredes públicas.
- **PrivateSubnet01Id y PrivateSubnet02Id**: IDs de las subredes privadas.
- **InternetGatewayId**: ID del Internet Gateway creado.
- **PublicRouteTableId**: ID de la tabla de rutas públicas.
- **PrivateRouteTable01Id y PrivateRouteTable02Id**: IDs de las tablas de rutas privadas.
- **NatGateway01Id y NatGateway02Id**: IDs de los NAT Gateways.
- **EKSClusterName**: Nombre del clúster EKS.
- **EKSClusterArn**: ARN del clúster EKS.
- **NodeGroupName**: Nombre del grupo de nodos gestionados.
- **NodeInstanceRoleArn**: ARN del rol IAM asociado al grupo de nodos.

## Cómo usar esta plantilla

1. Descarga la plantilla y guárdala como un archivo `cluster-eks.yml`.
2. Accede a la consola de AWS CloudFormation.
3. Sube la plantilla y personaliza los parámetros si es necesario.
4. Lanza la pila (stack) y espera a que los recursos se desplieguen correctamente.
5. Una vez finalizado, consulta las salidas (outputs) para obtener información clave como IDs de subredes, grupos de seguridad y el ARN del clúster.

## Notas

- Esta plantilla está configurada para un escenario típico de producción, con subredes privadas para los nodos y NAT Gateways para el acceso a Internet de los mismos.
- El complemento VPC CNI está configurado para garantizar la conectividad dentro del clúster.