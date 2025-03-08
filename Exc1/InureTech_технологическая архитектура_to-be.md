````puml

@startuml

frame "Регион Москва (Зона доступности 1)" {
    rectangle "Kubernetes Cluster (Москва)" as k8s_msk
    rectangle "PostgreSQL (Master)" as db_msk
    rectangle "Redis (Cluster)" as redis_msk
    rectangle "InsureTech namespace" as ns_msk
    rectangle "API Gateway (Москва)" as gw_msk

    gw_msk -> k8s_msk : HTTP(s) traffic
    k8s_msk -> ns_msk : API calls
    ns_msk -> db_msk : Write requests
    ns_msk -> redis_msk : Read cache
}

frame "Регион Дальний Восток (Зона доступности 2)" {
    rectangle "Kubernetes Cluster (ДВ)" as k8s_dv
    rectangle "PostgreSQL (Replica)" as db_dv
    rectangle "Redis (Cluster)" as redis_dv
    rectangle "InsureTech namespace" as ns_dv
    rectangle "API Gateway (ДВ)" as gw_dv

    gw_dv -> k8s_dv : HTTP(s) traffic
    k8s_dv -> ns_dv : API calls
    ns_dv -> db_dv : Read requests
    ns_dv -> redis_dv : Read cache
}

actor "Браузер пользователя" as user
rectangle "Global Load Balancer (GSLB)" as glb
rectangle "GeoDNS" as geodns

user -> geodns : DNS запрос
geodns -> glb : Load balancing
glb -> gw_msk : Redirect traffic
glb -> gw_dv : Redirect traffic

db_msk -- db_dv : Multi-master PostgreSQL
redis_msk -- redis_dv : Redis replication

@enduml

```