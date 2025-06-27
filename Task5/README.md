## Задание 5. Проектирование GraphQL API

### Cхема GraphQL: 

- см. файл [schema.graphql](./schema.graphql)

### Описание типов данных

```
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

# Определение Query (запросов)

type Query {
  client(id: ID!): Client
  documents(clientId: ID!): [Document]
  relatives(clientId: ID!): [Relative]
}

# Определение Mutation (операции изменения данных)

type Mutation {
  createClient(input: CreateClientInput!): Client
  updateClient(id: ID!, input: UpdateClientInput!): Client
  deleteClient(id: ID!): Boolean

  createDocument(clientId: ID!, input: CreateDocumentInput!): Document
  updateDocument(id: ID!, input: UpdateDocumentInput!): Document
  deleteDocument(id: ID!): Boolean

  createRelative(clientId: ID!, input: CreateRelativeInput!): Relative
  updateRelative(id: ID!, input: UpdateRelativeInput!): Relative
  deleteRelative(id: ID!): Boolean
}

# Определение Input Types для Mutation

input CreateClientInput {
  name: String
  age: Int
}

input UpdateClientInput {
  name: String
  age: Int
}

input CreateDocumentInput {
  type: String
  number: String
  issueDate: String
  expiryDate: String
}

input UpdateDocumentInput {
  type: String
  number: String
  issueDate: String
  expiryDate: String
}

input CreateRelativeInput {
  relationType: String
  name: String
  age: Int
}

input UpdateRelativeInput {
  relationType: String
  name: String
  age: Int
}

schema {
  query: Query
  mutation: Mutation
}
```

### Разъяснения к схеме  

1. **Типы данных (type)**:  

- **Client**: Описывает структуру данных клиента, включая поля `id`, `name`, `age`, а также списки документов (`documents`) и родственников (`relatives`). Ключевое отличие от REST - документы и родственники не возвращаются по отдельным endpoint-ам, а могут быть запрошены вместе с основной информацией о клиенте.  

- **Document**: Описывает структуру документа с полями `id`, `type`, `number`, `issueDate` и `expiryDate`.  

  - **Relative**: Описывает структуру родственника с полями `id`, `relationType`, `name` и `age`.  

- `ID!` и `String`, `Int` - это стандартные скалярные типы GraphQL, указывающие на тип данных каждого поля. `!` означает, что поле обязательно должно присутствовать в ответе.  

2. **Запросы (Query)**:  

- **client(id: ID!)**: `Client`: Позволяет получить информацию о клиенте по его ID. Клиент может указать, какие поля клиента ему нужны (включая документы и родственников, или только основные данные).  

- **documents(clientId: ID!)**: `[Document]`: Позволяет получить список документов клиента по его ID. Полезно, если нужна только информация о документах.  

- **relatives(clientId: ID!)**: `[Relative]`: Позволяет получить список родственников клиента по его ID. Полезно, если нужна только информация о родственниках.  

- Вся информация получается одним запросом, без необходимости делать множество запросов, как в REST.  

3. **Мутации (Mutation)**:  

- **createClient(input: CreateClientInput!)**: `Client`: Создает нового клиента.  
- **updateClient(id: ID!, input: UpdateClientInput!)**: `Client`: Обновляет существующего клиента по ID.  
- **deleteClient(id: ID!)**: `Boolean`: Удаляет клиента по ID. Возвращает `true` в случае успеха.  

- **createDocument(clientId: ID!, input: CreateDocumentInput!)**: `Document`: Создает новый документ для клиента. Требует указания ID клиента.  
- **updateDocument(id: ID!, input: UpdateDocumentInput!)**: `Document`: Обновляет существующий документ по ID.  
- **deleteDocument(id: ID!)**: `Boolean`: Удаляет документ по ID.  

- **createRelative(clientId: ID!, input: CreateRelativeInput!)**: `Relative`: Создает нового родственника для клиента. Требует указания ID клиента.  
- **updateRelative(id: ID!, input: UpdateRelativeInput!)**: `Relative`: Обновляет существующего родственника по ID.  
- **deleteRelative(id: ID!)**: `Boolean`: Удаляет родственника по ID.  

- Мутации позволяют `core-app` вносить изменения в клиентские данные, если пользователя нет в базе данных или его данные изменились (как описано в задании). Также, мутации используются для редактирования клиентских данных в личном кабинете.  

4. **Input Types**:  

- **CreateClientInput**, **UpdateClientInput**, **CreateDocumentInput**, **UpdateDocumentInput**, **CreateRelativeInput**, **UpdateRelativeInput**: Определяют структуру данных, необходимых для создания и обновления соответствующих сущностей. Это позволяет четко определить, какие поля являются обязательными для создания, и какие могут быть обновлены. Использование input types упрощает и типизирует передачу данных в мутации.  

### Примеры использования 

•  Получение основной информации о клиенте и списка его документов:  
    ```graphql
    query {
      client(id: <CLIENT_ID>) {
        id
        name
        age
        documents {
          id
          type
          number
        }
      }
    }
    ```  

•  Создание нового клиента:  
    ```graphql
    mutation {
      createClient(input: {
        name: <NAME>
        age: <AGE>
      }) {
        id
        name
        age
      }
    }
    ```  

•  Обновление информации о клиенте:  
    ```graphql
    mutation {
      updateClient(id: <CLIENT_ID>, input: {
        age: <AGE>
      }) {
        id
        name
        age
      }
    }
    ```  

•  Добавление нового документа клиенту:  
    ```graphql
    mutation {
      createDocument(clientId: <CLIENT_ID>, input: {
        type: <TYPE>
        number: <NUMBER>
        issueDate: <ISSUE_DATE>
        expiryDate: <EXPIRY_DATE>
      }) {
        id
        type
        number
      }
    }
    ```  

### Как GraphQL оптимизирует взаимодействие:  

•  **Избежание избыточной передачи данных**: Клиенты (`core-app`, веб-приложения) могут запрашивать только те данные, которые им действительно нужны. Вместо получения всей информации о клиенте, можно запросить только имя и список документов.  
•  **Уменьшение количества запросов**: Вместо нескольких REST-запросов для получения клиента, его документов и родственников, можно выполнить один GraphQL-запрос, который вернет все необходимые данные.  
•  **Четкая типизация**: GraphQL обеспечивает четкую структуру данных, что облегчает разработку и отладку как на стороне клиента, так и на стороне сервера.  
•  **Самодокументируемость**: GraphQL-схема является самодокументируемой, что упрощает понимание API и его использование. Инструменты, такие как GraphiQL, позволяют исследовать схему и создавать запросы интерактивно.  

**Замечания**:  

•  Эта схема является базовой и может быть расширена в зависимости от дальнейших требований к сервису `client-info`. Например, можно добавить поддержку фильтрации, сортировки и пагинации.  
•  Дополнительно можно предусмотреть механизмы аутентификации и авторизации для защиты данных.  
•  На первом этапе можно реализовать GraphQL API как прослойку над существующим REST API, постепенно переходя к прямой работе с базой данных. Это позволит плавно перейти на новую технологию без необходимости переписывать весь сервис сразу.  
