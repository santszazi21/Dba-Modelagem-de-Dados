
# 🧾 DICIONÁRIO DE DADOS — Lacrei Saúde

> Arquivo SQL de referência: `scripts/01_create_tables.sql`  
> Banco: **PostgreSQL 16+**

---

## **Tabela: usuarios**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| id | `UUID` | `PRIMARY KEY (pk_usuarios_id)`, `DEFAULT gen_random_uuid()` | Identificador único do usuário |
| nome | `VARCHAR(100)` | `NOT NULL` | Nome completo do usuário |
| email | `VARCHAR(100)` | `NOT NULL`, `UNIQUE (uk_usuarios_email)` | E-mail único |
| genero | `genero_enum` | `NULLABLE` | Identidade de gênero |
| data_nascimento | `DATE` | `NULLABLE` | Data de nascimento |
| criado_em | `TIMESTAMP` | `DEFAULT now()` | Data e hora de criação do registro |

**Índices:**
- `idx_usuarios_nome` → (`nome`)
- `idx_usuarios_email` → (`email`)

---

## **Tabela: profissionais**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| usuario_id | `UUID` | `PRIMARY KEY`, `FOREIGN KEY (fk_profissionais_usuario)` → `usuarios(id)` | Identificador do usuário vinculado |
| registro_profissional | `VARCHAR(30)` | `NOT NULL` | Registro do conselho profissional (ex.: CRM, CRP) |

**Índices:**
- `idx_profissionais_registro` → (`registro_profissional`)

---

## **Tabela: pacientes**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| usuario_id | `UUID` | `PRIMARY KEY`, `FOREIGN KEY (fk_pacientes_usuario)` → `usuarios(id)` | Identificador do paciente vinculado |
| prontuario | `JSONB` | `NULLABLE` | Informações médicas dinâmicas (ex.: alergias, histórico, etc.) |

**Índices:**
- `idx_pacientes_prontuario_gin` → `USING GIN(prontuario)`

---

## **Tabela: especialidades**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| id | `SERIAL` | `PRIMARY KEY (pk_especialidades_id)` | Identificador da especialidade |
| nome | `VARCHAR(100)` | `NOT NULL` | Nome da especialidade médica |

---

## **Tabela: profissionais_especialidades**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| usuario_id | `UUID` | `FOREIGN KEY (fk_prof_esp_usuario)` → `profissionais(usuario_id)` | Profissional vinculado |
| especialidade_id | `INT` | `FOREIGN KEY (fk_prof_esp_especialidade)` → `especialidades(id)` | Especialidade vinculada |

**Chaves e Índices:**
- `PRIMARY KEY (usuario_id, especialidade_id)` (`pk_prof_esp_composta`)
- `idx_prof_esp_especialidade_id` → (`especialidade_id`)

---

## **Tabela: atendimentos**
| Coluna | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| id | `UUID` | `PRIMARY KEY (pk_atendimentos_id)`, `DEFAULT gen_random_uuid()` | Identificador do atendimento |
| paciente_id | `UUID` | `FOREIGN KEY (fk_atendimento_paciente)` → `pacientes(usuario_id)` | Paciente vinculado |
| profissional_id | `UUID` | `FOREIGN KEY (fk_atendimento_profissional)` → `profissionais(usuario_id)` | Profissional responsável |
| data_hora | `TIMESTAMP` | `NOT NULL`, `CHECK (data_hora >= now())` | Data e hora da consulta |
| descricao | `TEXT` | `NULLABLE` | Observações sobre o atendimento |

**Regras de negócio (constraints adicionais):**
- `UNIQUE (profissional_id, data_hora)` (`uk_atendimento_profissional_data`) → evita double booking  
- `CHECK (profissional_id <> paciente_id)` (`ck_atendimento_profissional_diferente_paciente`)  
- `CHECK (data_hora >= now())` (`ck_atendimento_data_valida`)

**Índices:**
- `idx_atendimentos_data_hora` → (`data_hora`)

---

## **Domínio: genero_enum**
```sql
CREATE TYPE genero_enum AS ENUM (
  'cisgênero', 'transgênero', 'não-binário', 'intersexo',
  'agênero', 'outro', 'prefiro_não_dizer'
);
```

---

## **Exemplo de Inserts (dados iniciais)**
```sql
INSERT INTO usuarios (nome, email, genero) VALUES
('Ana Souza', 'ana@exemplo.com', 'cisgênero'),
('Carlos Lima', 'carlos@exemplo.com', 'transgênero');

INSERT INTO profissionais (usuario_id, registro_profissional)
SELECT id, 'CRM-12345' FROM usuarios WHERE email = 'ana@exemplo.com';

INSERT INTO pacientes (usuario_id, prontuario)
SELECT id, '{"alergias": ["penicilina"], "historico": ["asma leve"]}'::jsonb
FROM usuarios WHERE email = 'carlos@exemplo.com';

INSERT INTO especialidades (nome) VALUES ('Clínica Geral'), ('Pediatria');

INSERT INTO profissionais_especialidades (usuario_id, especialidade_id)
SELECT p.usuario_id, e.id FROM profissionais p, especialidades e WHERE e.nome = 'Clínica Geral';

INSERT INTO atendimentos (paciente_id, profissional_id, data_hora, descricao)
SELECT 
  (SELECT usuario_id FROM pacientes),
  (SELECT usuario_id FROM profissionais),
  NOW() + INTERVAL '1 day',
  'Consulta de rotina';
```

---

## **Resumo Técnico**
| Categoria | Implementação |
|------------|----------------|
| Banco | PostgreSQL |
| PKs e FKs | Nomeadas e indexadas |
| Índices adicionais | Criados para busca e performance |
| Campos dinâmicos | `JSONB` com índice GIN |
| Regras de integridade | `CHECK`, `UNIQUE`, `NOT NULL`, `FK`, `<>` |
