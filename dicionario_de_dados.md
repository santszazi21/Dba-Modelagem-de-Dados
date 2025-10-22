# 🗂️ Dicionário de Dados — Sistema Lacrei Saúde

> **Banco:** PostgreSQL 16+  
> **Autor:** Pedro Trindade  
> **Data da última atualização:** 21/10/2025  
> **Descrição:** Estrutura oficial e atualizada do banco de dados Lacrei Saúde.

---

## 📑 Sumário

1. [Tabela: tipo_usuario](#1-tabela--tipo_usuario)  
2. [Tabela: planos_saude](#2-tabela--planos_saude)  
3. [Tabela: usuarios](#3-tabela--usuarios)  
4. [Tabela: pacientes](#4-tabela--pacientes)  
5. [Tabela: profissionais](#5-tabela--profissionais)  
6. [Tabela: especialidades](#6-tabela--especialidades)  
7. [Tabela: profissionais_especialidades](#7-tabela--profissionais_especialidades)  
8. [Tabela: profissionais_planos](#8-tabela--profissionais_planos)  
9. [Tabela: atendimentos](#9-tabela--atendimentos)  
10. [Tipo Enumerado: genero_enum](#10-tipo-enumerado--genero_enum)  
11. [Relacionamentos](#11-relacionamentos-resumo)

---

## 1. Tabela — `tipo_usuario`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `id` | SERIAL | PK | Identificador único do tipo de usuário |
| `nome` | VARCHAR(50) | NOT NULL, UNIQUE | Nome do tipo (paciente, profissional, administrador) |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o tipo está ativo |

---

## 2. Tabela — `planos_saude`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `id` | SERIAL | PK | Identificador único do plano |
| `nome` | VARCHAR(100) | NOT NULL, UNIQUE | Nome do plano de saúde |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o plano está ativo |
| `criado_em` | TIMESTAMP | DEFAULT now() | Data de criação |
| `atualizado_em` | TIMESTAMP | DEFAULT now() | Última atualização |

---

## 3. Tabela — `usuarios`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `id` | UUID | PK, DEFAULT gen_random_uuid() | Identificador único do usuário |
| `nome` | VARCHAR(100) | NOT NULL | Nome completo do usuário |
| `email` | VARCHAR(100) | NOT NULL, UNIQUE | Endereço de e-mail |
| `tipo_usuario_id` | INT | FK → tipo_usuario(id) | Tipo de usuário |
| `genero` | genero_enum | — | Identidade de gênero do usuário |
| `data_nascimento` | DATE | — | Data de nascimento |
| `criado_em` | TIMESTAMP | DEFAULT now() | Data de criação |
| `atualizado_em` | TIMESTAMP | DEFAULT now() | Última atualização |
| `criado_por` | UUID | FK opcional | Usuário responsável pela criação (auditoria) |

---

## 4. Tabela — `pacientes`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `usuario_id` | UUID | PK, FK → usuarios(id) | Identificador do paciente |
| `plano_saude_id` | INT | FK → planos_saude(id) | Plano de saúde associado |
| `prontuario` | JSONB | — | Dados clínicos estruturados |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o paciente está ativo |

---

## 5. Tabela — `profissionais`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `usuario_id` | UUID | PK, FK → usuarios(id) | Identificador do profissional |
| `registro_profissional` | VARCHAR(30) | NOT NULL, UNIQUE | Registro no conselho de classe |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o profissional está ativo |

---

## 6. Tabela — `especialidades`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `id` | SERIAL | PK | Identificador da especialidade |
| `nome` | VARCHAR(100) | NOT NULL, UNIQUE | Nome da especialidade médica |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se está ativa no sistema |

---

## 7. Tabela — `profissionais_especialidades`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `usuario_id` | UUID | PK (composite), FK → profissionais(usuario_id) | Profissional vinculado |
| `especialidade_id` | INT | PK (composite), FK → especialidades(id) | Especialidade associada |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o vínculo está ativo |
| `criado_em` | TIMESTAMP | DEFAULT now() | Data de criação da relação |

---

## 8. Tabela — `profissionais_planos`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `usuario_id` | UUID | PK (composite), FK → profissionais(usuario_id) | Profissional vinculado |
| `plano_saude_id` | INT | PK (composite), FK → planos_saude(id) | Plano aceito pelo profissional |
| `ativo` | BOOLEAN | DEFAULT TRUE | Indica se o vínculo está ativo |
| `criado_em` | TIMESTAMP | DEFAULT now() | Data de criação da relação |

---

## 9. Tabela — `atendimentos`

| Campo | Tipo | Restrições | Descrição |
|--------|------|-------------|------------|
| `id` | UUID | PK, DEFAULT gen_random_uuid() | Identificador único do atendimento |
| `paciente_id` | UUID | FK → pacientes(usuario_id), NOT NULL | Paciente atendido |
| `profissional_id` | UUID | FK → profissionais(usuario_id), NOT NULL | Profissional responsável |
| `data_hora` | TIMESTAMP | NOT NULL, CHECK (data_hora >= now()) | Data e hora do atendimento |
| `descricao` | TEXT | — | Detalhes do atendimento |
| `criado_em` | TIMESTAMP | DEFAULT now() | Data de criação |
| `atualizado_em` | TIMESTAMP | DEFAULT now() | Última atualização |
| `ativo` | BOOLEAN | DEFAULT TRUE | Status ativo/inativo |
| 🔹 **Restrições extras** | | |
| `UNIQUE (profissional_id, data_hora)` | — | Evita agendamento duplicado |
| `CHECK (profissional_id <> paciente_id)` | — | Garante que um profissional não atenda a si mesmo |

---

## 10. Tipo Enumerado — `genero_enum`

| Valor | Descrição |
|--------|------------|
| `cisgênero` | Pessoa cujo gênero corresponde ao sexo de nascimento |
| `transgênero` | Pessoa cuja identidade de gênero difere do sexo de nascimento |
| `não-binário` | Pessoa que não se identifica exclusivamente como homem ou mulher |
| `intersexo` | Pessoa com características sexuais mistas |
| `agênero` | Pessoa que não possui identidade de gênero |
| `outro` | Outra identidade de gênero |
| `prefiro_não_dizer` | Opção de não declarar gênero |

---

## 11. Relacionamentos (Resumo)

| Origem | Destino | Tipo | Descrição |
|--------|----------|------|------------|
| `usuarios.tipo_usuario_id` | `tipo_usuario.id` | 1:N | Define o tipo do usuário |
| `pacientes.usuario_id` | `usuarios.id` | 1:1 | Cada paciente é um usuário |
| `profissionais.usuario_id` | `usuarios.id` | 1:1 | Cada profissional é um usuário |
| `pacientes.plano_saude_id` | `planos_saude.id` | N:1 | Cada paciente possui um plano |
| `profissionais_especialidades` | `profissionais` / `especialidades` | N:N | Profissionais com múltiplas especialidades |
| `profissionais_planos` | `profissionais` / `planos_saude` | N:N | Planos aceitos por profissionais |
| `atendimentos.paciente_id` | `pacientes.usuario_id` | N:1 | Paciente do atendimento |
| `atendimentos.profissional_id` | `profissionais.usuario_id` | N:1 | Profissional do atendimento |

---

📘 **Observações Gerais**

- As tabelas possuem colunas de auditoria (`criado_em`, `atualizado_em`).
- Enum `genero_enum` garante padronização de identidade de gênero.


---

🧠 **Autor:** *Pedro Trindade*  

