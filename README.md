1 Exemplo
usuarios (id PK) 
│
├──< atendimentos >──┤
                     │
                profissionais_especialidades
                     │
               especialidades (id PK)
| Campo           | Tipo         | Restrições                         |
| --------------- | ------------ | ---------------------------------- |
| id              | UUID         | PK                                 |
| nome            | VARCHAR(100) | NOT NULL                           |
| email           | VARCHAR(100) | NOT NULL, UNIQUE                   |
| tipo_usuario    | ENUM         | CHECK ('profissional', 'paciente') |
| genero          | VARCHAR(50)  | NULLABLE                           |
| data_nascimento | DATE         | NULLABLE                           |
| criado_em       | TIMESTAMP    | DEFAULT now()                      |

| Campo | Tipo         | Restrições |
| ----- | ------------ | ---------- |
| id    | SERIAL       | PK         |
| nome  | VARCHAR(100) | NOT NULL   |

| Campo            | Tipo | Restrições                  |
| ---------------- | ---- | --------------------------- |
| usuario_id       | UUID | PK, FK → usuarios(id)       |
| especialidade_id | INT  | PK, FK → especialidades(id) |

| Campo           | Tipo      | Restrições        |
| --------------- | --------- | ----------------- |
| id              | UUID      | PK                |
| paciente_id     | UUID      | FK → usuarios(id) |
| profissional_id | UUID      | FK → usuarios(id) |
| data_hora       | TIMESTAMP | NOT NULL          |
| descricao       | TEXT      | NULLABLE          |



               
2 Exemplo
usuarios (id PK)
├── profissionais (usuario_id PK, FK)
│     └── profissionais_especialidades
│             └── especialidades
├── pacientes (usuario_id PK, FK)
     └── atendimentos
           └── profissionais
           
| Campo           | Tipo         | Restrições       |
| --------------- | ------------ | ---------------- |
| id              | UUID         | PK               |
| nome            | VARCHAR(100) | NOT NULL         |
| email           | VARCHAR(100) | NOT NULL, UNIQUE |
| genero          | VARCHAR(50)  | NULLABLE         |
| data_nascimento | DATE         | NULLABLE         |
| criado_em       | TIMESTAMP    | DEFAULT now()    |

| Campo      | Tipo        | Restrições            |
| ---------- | ----------- | --------------------- |
| usuario_id | UUID        | PK, FK → usuarios(id) |
| registro   | VARCHAR(30) | NOT NULL              |

| Campo      | Tipo | Restrições            |
| ---------- | ---- | --------------------- |
| usuario_id | UUID | PK, FK → usuarios(id) |
| prontuario | TEXT | NULLABLE              |

| Campo           | Tipo      | Restrições                     |
| --------------- | --------- | ------------------------------ |
| id              | UUID      | PK                             |
| paciente_id     | UUID      | FK → pacientes(usuario_id)     |
| profissional_id | UUID      | FK → profissionais(usuario_id) |
| data_hora       | TIMESTAMP | NOT NULL                       |
| descricao       | TEXT      | NULLABLE                       |


