````puml

@startuml

package "InsureTech Prod" {
    rectangle "core-app" as core_app
    rectangle "ins-product-aggregator" as aggregator
    rectangle "ins-comp-settlement" as settlement
    database "PostgreSQL" as core_db
    database "ins-comp-settlement-db" as settlement_db
    rectangle "client-info" as client_info
    database "client-info-db" as client_db
    rectangle "InsureTech Web" as web
    queue "Kafka (Event Bus)" as kafka
}

actor "Клиент" as client
rectangle "Системы страховых компаний" as insurance_systems
rectangle "Системы партнёров" as partner_systems
rectangle "Платёжный сервис" as payment_service

client -> web : Использует
web -> core_app : REST\nПолучение информации по продуктам и тарифам

core_app -> kafka : Публикация событий\n"Оформлена страховка"
kafka -> settlement : Подписка на события\n"Обновление данных о страховках"
settlement -> settlement_db : TCP\nЗапись данных о страховках

core_app -> kafka : Публикация событий\n"Запрос информации о продуктах"
kafka -> aggregator : Подписка на события\n"Получение информации от страховых компаний"
aggregator -> insurance_systems : REST / SOAP / GraphQL\nЗапрос данных у страховых компаний
aggregator -> kafka : Публикация событий\n"Актуальные продукты и тарифы"

kafka -> core_app : Подписка на события\n"Актуальные продукты и тарифы"
core_app -> core_db : TCP\nОбновление локального кеша продуктов

kafka -> settlement : Подписка на события\n"Актуальные продукты и тарифы"
settlement -> settlement_db : TCP\nОбновление локального кеша продуктов

core_app -> client_info : REST\nПолучение/сохранение информации о клиентах
client_info -> client_db : TCP\nРабота с данными

core_app -> payment_service : HTTP\nПроведение оплаты за страховки
settlement -> insurance_systems : REST / SOAP / GraphQL\nПередача данных для взаиморасчётов

partner_systems -> core_app : REST\nОформление страховок

@enduml



```