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

ENUM

Descrição:
O tipo ENUM define um conjunto fixo de valores diretamente no esquema do banco (ex.: tipo_usuario ENUM('profissional', 'paciente')).
O valor só pode ser um dos definidos no tipo, e o controle é feito pelo próprio banco de dados.

 Vantagens

Simplicidade: rápido de implementar, sem necessidade de criar tabela extra.

Integridade automática: o banco garante que apenas valores válidos sejam inseridos.

Performance: o armazenamento interno é otimizado, e comparações são mais rápidas.

Baixa complexidade: ideal para listas pequenas e estáveis (ex.: “ativo/inativo”, “profissional/paciente”).

Desvantagens

Baixa flexibilidade: adicionar, remover ou renomear valores exige alterar o schema (migrar tipo ENUM).

Dependência do SGBD: diferentes bancos (PostgreSQL, MySQL, etc.) têm sintaxes e comportamentos distintos.

Dificulta internacionalização (i18n): se os valores precisarem ser exibidos em diferentes idiomas, o ENUM não oferece suporte direto.

Difícil versionamento: cada alteração precisa de uma migração manual e pode impactar sistemas em produção.


FK tipo_usuario_id → tipo_usuario.id
CREATE TABLE tipo_usuario (
  id SERIAL PRIMARY KEY,
  nome VARCHAR(50) UNIQUE NOT NULL
);

Vantagens

Flexibilidade e manutenção simples: novos valores podem ser adicionados via INSERT, sem alterar o schema.

Centralização de regras: o domínio pode ter colunas extras (ex.: descricao, ativo, ordem_exibicao).

Internacionalização fácil: permite traduzir nomes e descrições em tabelas auxiliares.

Integração com sistemas externos: útil quando há APIs, cadastros ou dashboards que leem domínios dinamicamente.

 Desvantagens

Consulta mais complexa: exige JOIN para exibir o valor textual.

Leve perda de performance: o acesso indireto (JOIN) pode ser marginalmente mais lento em grandes volumes.

Sobrecarga inicial: requer criação e manutenção de tabela, chaves e índices adicionais.


Quando usar cada um

Domínio pequeno, fixo e raramente alterado (ex.: “profissional/paciente”, “sexo”, “status ativo/inativo”)	ENUM	Melhor performance, simplicidade e integridade automática.
Domínio sujeito a mudança ou expansão (ex.: “tipo de plano de saúde”, “especialidade médica”, “categoria de atendimento”)	Tabela de Domínio	Facilita manutenção e evita migrações de schema.

Aplicações multilíngues, dinâmicas ou com painel administrativo	Tabela de Domínio	Permite adicionar traduções e gerenciar opções via CRUD.
Protótipo, MVP ou ambiente de testes	ENUM	Implementação mais rápida, menos tabelas.
Produção de longo prazo, múltiplos sistemas integrados	Tabela de Domínio	Escalabilidade e governança de dados.
