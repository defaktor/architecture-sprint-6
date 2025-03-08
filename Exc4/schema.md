````puml

@startuml

package "InsureTech Prod" {
    rectangle "core-app" as core_app
    rectangle "osago-aggregator" as osago_aggregator
    rectangle "ins-product-aggregator" as product_aggregator
    rectangle "ins-comp-settlement" as settlement
    database "PostgreSQL" as core_db
    database "osago-db" as osago_db
    queue "Kafka (Event Bus)" as kafka
    rectangle "client-info" as client_info
    database "client-info-db" as client_db
    rectangle "InsureTech Web" as web
}

actor "Клиент" as client
rectangle "Системы страховых компаний" as insurance_systems
rectangle "Системы партнёров" as partner_systems
rectangle "Платёжный сервис" as payment_service

client -> web : Использует
web -[#blue]-> core_app : WebSocket API\nОбновление предложений
web -> core_app : REST\nОформление заявки

core_app -[#blue]-> kafka : Отправка заявки на ОСАГО
kafka -[#blue]-> osago_aggregator : Подписка на заявки ОСАГО
osago_aggregator -> osago_db : TCP\nСохранение состояния заявки

osago_aggregator -[#red]--> insurance_systems : REST\nОтправка заявок (Parallel Requests)
osago_aggregator -[#red]-> insurance_systems : REST\nОпрос статуса заявки\n<< Circuit Breaker, Retry, Timeout >>

insurance_systems -[#red]-> osago_aggregator : REST\nОтвет по ОСАГО
osago_aggregator -[#blue]-> kafka : Публикация предложений ОСАГО
kafka -[#blue]-> core_app : Подписка на предложения ОСАГО
core_app -> core_db : TCP\nСохранение предложений

core_app -> client_info : REST\nПолучение/сохранение информации о клиентах
client_info -> client_db : TCP\nРабота с данными

core_app -> payment_service : HTTP\nПроведение оплаты за страховки
settlement -> insurance_systems : REST / SOAP / GraphQL\nПередача данных для взаиморасчётов

partner_systems -> core_app : REST\nОформление страховок

@enduml




```