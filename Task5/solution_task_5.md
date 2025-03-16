# Проектирование GraphQL API

## Постановка проблемы и требования

При развитии сервиса управления клиентскими данными (client-info) команда столкнулась с проблемой. Потребители данных (веб-приложения и сервиса core-app) в разных сценариях продажи и обслуживания страховок могут требовать абсолютно разные данные. При этом карточка клиента у сервиса достаточно объёмная: общий атрибутивный состав достигает 500 штук. Из-за высокой вариативности набора запрашиваемых данных команда приняла решение в своём REST API предоставить множество отдельных ресурсов, с помощью которых можно запрашивать отдельные объекты данных клиента: контакты, документы, родственники и так далее.
Однако такая реализация кратно увеличивает нагрузку и RPS сервиса client-info, поскольку в рамках одного сценария могут потребоваться сразу несколько объектов данных — их придётся запрашивать по отдельности. Предоставить один ресурс для получения абсолютно всех данных клиента не представляется возможным: объём передаваемых данных будет настолько большим, что замедлит скорость взаимодействия с сервисами (особенно с веб-приложением).
Вы обсудили проблему с командой и приняли решение перевести REST API сервиса client-info на GraphQL.

## Технологические решение

#### Контракт Swagger
```yaml
swagger: '2.0'
info:
 description: API сервиса управления клиентскими данными
 version: 1.0.0
 title: Клиентский Сервис
host: api.client-service.com
basePath: /v1
schemes:
 - https
paths:
 /clients/{id}:
   get:
     tags:
       - Клиент
     summary: Получить информацию о клиенте по ID
     description: Возвращает информацию о клиенте.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           $ref: '#/definitions/Client'
 /clients/{id}/documents:
   get:
     tags:
       - Документы
     summary: Список документов клиента
     description: Возвращает список документов клиента по ID.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента для поиска его документов
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           type: array
           items:
             $ref: '#/definitions/Document'
 /clients/{id}/relatives:
   get:
     tags:
       - Родственники
     summary: Информация о родственниках клиента
     description: Возвращает информацию о родственниках клиента по ID.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           type: array
           items:
             $ref: '#/definitions/Relative'
definitions:
 Client:
   type: object
   properties:
     id:
       type: string
     name:
       type: string
     age:
       type: integer
 Document:
   type: object
   properties:
     id:
       type: string
     type:
       type: string
     number:
       type: string
     issueDate:
       type: string
     expiryDate:
       type: string
 Relative:
   type: object
   properties:
     id:
       type: string
     relationType:
       type: string
     name:
       type: string
     age:
       type: integer
```

#### Схема GraphQL
```
schema {
  query: Query
}

type Query {
  client(id: ID!): Client
}

type Client {
  id: ID!
  name: String
  age: Int
  documents: [Document]
  relatives: [Relative]
}

type Document {
  id: ID!
  type: String
  number: String
  issueDate: String
  expiryDate: String
}

type Relative {
  id: ID!
  relationType: String
  name: String
  age: Int
}
```
[schema.graphql](schema.graphql)

#### Описание схемы
1) Корневой Тип Query
   - client(id: ID!): Client 
     - Позволяет получить информацию о клиенте по его идентификатору.
     - В ответе можно выбрать необходимые поля клиента, включая связанные документы и родственников.
2) Тип Client
   - id: ID!
     - Уникальный идентификатор клиента.
   - name: String
     - Имя клиента.
   - age: Int
     - Возраст клиента.
   - documents: [Document]
     - Список документов клиента. Позволяет получать связанные документы в рамках одного запроса.
   - relatives: [Relative]
     - Список родственников клиента. Позволяет получать связанные данные о родственниках в рамках одного запроса.
3) Тип Document
   - id: ID!
     - Уникальный идентификатор документа.
   - type: String
     - Тип документа (например, паспорт, водительское удостоверение).
   - number: String
     - Номер документа.
   - issueDate: String
     - Дата выдачи документа.
   - expiryDate: String
     - Дата истечения срока действия документа.
4) Тип Relative
   - id: ID!
     - Уникальный идентификатор родственника.
   - relationType: String
     - Тип родственной связи (например, отец, мать, брат).
   - name: String
     - Имя родственника.
   - age: Int
     - Возраст родственника.

#### Пример запроса, который позволяет получить информацию о клиенте вместе с его документами и родственниками:
```jsx
query GetClientDetails($clientId: ID!) {
  client(id: $clientId) {
    id
    name
    age
    documents {
      id
      type
      number
      issueDate
      expiryDate
    }
    relatives {
      id
      relationType
      name
      age
    }
  }
}
```
