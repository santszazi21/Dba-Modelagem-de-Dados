usuarios (id PK) 
│
├──< atendimentos >──┤
                     │
                profissionais_especialidades
                     │
               especialidades (id PK)
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

