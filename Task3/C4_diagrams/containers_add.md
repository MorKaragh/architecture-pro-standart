@startuml standart bank deposit process
!include assets/C4_templates/C4_Container.puml
!include assets/icons/building_columns.puml

title Целевая архитектура банка Стандарт (депозитный процесс)

Person_Ext(user, "Клиент", "")

Container(site, "Сайт", "PHP", "")
Container(ebank, "Интернет банк", "C#", "")
Container(call_center, "Система колл-центра", "", "")

Boundary(backend, "бэкенд бизнес-сценариев") {
    Container(crm, "Система CRM", "на платформе call-центра", "Хранение, агрегация клиентский данных")
    Container(zayav_service, "Сервис заявок на депозит", "Java", "Агрегация заявок из разных каналов вокруг клиента. Управление жизненным циклом заявки до загрузки в учёт")
    Container(notification_service, "Сервис нотификаций и SMS-подтверждений", "Java", "Отправляет SMS-нотификации, позволяет делать проверки кодом и реализовать ПЭП")
    Container(personalization_service, "Сервис персональных предложений", "Java", "Считает и подготваливает персональные предложения")
    Container(gateway, "API Gateway", "", "")
}

Boundary(deep, "бэкенд строгого учёта") {
    Container(abs, "АБС", "Delphi", "Постепенно отсюда будут убираться функции, не связанные с основными процессами и учётом", $sprite="building_columns")
    Container(abs_adapter, "Адаптер к АБС", "Java, Spring", "Кеш, реализация асинхронности, resilience-паттерны")
}

Boundary(partners, "Компоненты подрядчиков") {
    Container_Ext(call_center_partner, "Система партнёрского колл-центра", "", "")
    Container_Ext(sms_gateway, "SMS-шлюз", "", "")
}

ContainerQueue(eventq, "Брокер событий", "Kafka")
ContainerDb(abs_db, "БД АБС", "")


' === СВЯЗИ ===

' Rel(abs_adapter, abs, "HTTPS", "интеграция данных в АБС")
Rel(site, gateway, "HTTPS", "сохранение заявок канала Сайт")
Rel(ebank, gateway, "HTTPS", "сохранение заявок канала Интернет банк")
Rel(call_center, gateway, "HTTPS", "сохранение заявок канала Колл-центр")
Rel(gateway, zayav_service, "HTTPS", "")
Rel(gateway, personalization_service, "HTTPS", "")
Rel(zayav_service, notification_service, "HTTPS", "")

Rel(personalization_service, crm, "HTTPS", "")
Rel(zayav_service, crm, "HTTPS", "")

BiRel(zayav_service, abs_adapter, "HTTPS", "")

Rel(abs_adapter, abs, "HTTPS", "")

Rel(user, call_center, "телефон", "")
Rel(user, ebank, "HTTPS", "")
Rel(user, site, "HTTPS", "")
' BiRel(eventq, event_srv, "Subscribe/Publish", "Обработка событий")

@enduml
