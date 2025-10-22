🧾 DICIONÁRIO DE DADOS — Plataforma Lacrei Saúde 

Banco: PostgreSQL 16+
Arquivo de referência: scripts/01_create_tables.sql

Domínios (ENUM e tabelas de apoio)
1.1 Tipo: genero_enum

Define o gênero da pessoa usuária.

Valor	Descrição
cisgênero	Pessoa cuja identidade de gênero corresponde ao sexo atribuído no nascimento
transgênero	Pessoa cuja identidade de gênero difere do sexo atribuído no nascimento
não-binário	Pessoa que não se identifica exclusivamente como homem ou mulher
intersexo	Pessoa com variações biológicas de sexo
agênero	Pessoa que não se identifica com nenhum gênero
outro	Outro gênero não especificado
prefiro_não_dizer	Usuário optou por não informar
Tabela: tipo_usuario

Tabela de domínio para definir se o usuário é profissional, paciente, administrador, etc.

Coluna	Tipo	Restrições	Descrição
id	SERIAL	PRIMARY KEY (pk_tipo_usuario_id)	Identificador do tipo
nome	VARCHAR(50)	UNIQUE (uk_tipo_usuario_nome), NOT NULL	Nome do tipo de usuário
ativo	BOOLEAN	DEFAULT TRUE	Indica se o tipo está ativo

Exemplo de valores:
('profissional'), ('paciente'), ('administrador')

Tabela: planos_saude

Tabela de domínio para os planos de saúde, conforme decisão ENUM vs. Tabela de domínio → Tabela de domínio escolhida ✅

Coluna	Tipo	Restrições	Descrição
id	SERIAL	PRIMARY KEY (pk_planos_id)	Identificador do plano
nome	VARCHAR(100)	NOT NULL, UNIQUE (uk_planos_nome)	Nome do plano de saúde
ativo	BOOLEAN	DEFAULT TRUE	Indica se o plano está ativo
criado_em	TIMESTAMP	DEFAULT now()	Data de criação
atualizado_em	TIMESTAMP	DEFAULT now()	Última atualização
 usuarios

Tabela principal com todos os tipos de usuários (pacientes, profissionais, admin).

Coluna	Tipo	Restrições	Descrição
id	UUID	PRIMARY KEY (pk_usuarios_id), DEFAULT gen_random_uuid()	Identificador único
nome	VARCHAR(100)	NOT NULL	Nome completo
email	VARCHAR(100)	NOT NULL, UNIQUE (uk_usuarios_email)	E-mail do usuário
tipo_usuario_id	INT	FK (fk_usuarios_tipo) → tipo_usuario(id)	Tipo de usuário
genero	genero_enum	NULLABLE	Identidade de gênero
data_nascimento	DATE	NULLABLE	Data de nascimento
criado_em	TIMESTAMP	DEFAULT now()	Data de criação
atualizado_em	TIMESTAMP	DEFAULT now()	Data de atualização
criado_por	UUID	NULLABLE	Auditoria: usuário que criou o registro

Índices:

idx_usuarios_email

idx_usuarios_nome

 Tabela: pacientes
Coluna	Tipo	Restrições	Descrição
usuario_id	UUID	PRIMARY KEY, FK (fk_pacientes_usuario) → usuarios(id)	Identificador do paciente
plano_saude_id	INT	FK (fk_pacientes_plano) → planos_saude(id)	Plano de saúde vinculado
prontuario	JSONB	NULLABLE	Informações médicas dinâmicas
ativo	BOOLEAN	DEFAULT TRUE	Paciente ativo/inativo

Índices:

idx_pacientes_prontuario_gin → USING GIN(prontuario)

idx_pacientes_plano_saude_id

Regras:

Um paciente só pode ter um plano ativo por vez (UNIQUE por paciente).

Tabela: profissionais
Coluna	Tipo	Restrições	Descrição
usuario_id	UUID	PRIMARY KEY, FK (fk_profissionais_usuario) → usuarios(id)	Identificador do profissional
registro_profissional	VARCHAR(30)	NOT NULL, UNIQUE (uk_profissionais_registro)	Registro no conselho (CRM, CRP, etc.)
ativo	BOOLEAN	DEFAULT TRUE	Indica se o profissional está ativo

Índices:

idx_profissionais_registro

 Tabela: especialidades
Coluna	Tipo	Restrições	Descrição
id	SERIAL	PRIMARY KEY (pk_especialidades_id)	Identificador da especialidade
nome	VARCHAR(100)	NOT NULL, UNIQUE (uk_especialidades_nome)	Nome da especialidade
ativo	BOOLEAN	DEFAULT TRUE	Indica se está ativa
Tabela: profissionais_especialidades (N:N)
Coluna	Tipo	Restrições	Descrição
usuario_id	UUID	FK (fk_prof_esp_usuario) → profissionais(usuario_id)	Profissional vinculado
especialidade_id	INT	FK (fk_prof_esp_especialidade) → especialidades(id)	Especialidade
ativo	BOOLEAN	DEFAULT TRUE	Se o vínculo está ativo
criado_em	TIMESTAMP	DEFAULT now()	Data de criação

Chaves e Índices:

PRIMARY KEY (usuario_id, especialidade_id)

idx_prof_esp_especialidade_id

 Tabela: profissionais_planos (N:N)

Relaciona profissionais e planos de saúde aceitos.

Coluna	Tipo	Restrições	Descrição
usuario_id	UUID	FK (fk_prof_plan_usuario) → profissionais(usuario_id)	Profissional
plano_saude_id	INT	FK (fk_prof_plan_plano) → planos_saude(id)	Plano aceito
ativo	BOOLEAN	DEFAULT TRUE	Se o plano está ativo para o profissional
criado_em	TIMESTAMP	DEFAULT now()	Data de vínculo

Chaves e Índices:

PRIMARY KEY (usuario_id, plano_saude_id)

idx_prof_plan_plano_id

 Tabela: atendimentos
Coluna	Tipo	Restrições	Descrição
id	UUID	PRIMARY KEY (pk_atendimentos_id), DEFAULT gen_random_uuid()	Identificador
paciente_id	UUID	FK (fk_atendimento_paciente) → pacientes(usuario_id)	Paciente
profissional_id	UUID	FK (fk_atendimento_profissional) → profissionais(usuario_id)	Profissional
data_hora	TIMESTAMP	NOT NULL, CHECK (data_hora >= now())	Data e hora da consulta
descricao	TEXT	NULLABLE	Observações
criado_em	TIMESTAMP	DEFAULT now()	Data de criação
atualizado_em	TIMESTAMP	DEFAULT now()	Atualização
ativo	BOOLEAN	DEFAULT TRUE	Estado ativo/inativo

Regras de Negócio (Constraints):

UNIQUE (profissional_id, data_hora) → Evita duplo agendamento

CHECK (profissional_id <> paciente_id) → Paciente ≠ profissional

CHECK (data_hora >= now()) → Data futura válida

Índices:

idx_atendimentos_data_hora

idx_atendimentos_profissional_id

idx_atendimentos_paciente_id

 LGPD e Auditoria

Minimização de dados: só informações essenciais são guardadas.

Dados sensíveis (saúde): armazenados em prontuario JSONB, separado da tabela usuarios.

Auditoria básica: todas as tabelas principais possuem criado_em, atualizado_em e ativo.

Acesso controlado: informações médicas nunca devem ser retornadas em consultas públicas.
 Consultas de Exemplo (para testes)
Buscar profissionais por especialidade
SELECT u.nome, e.nome AS especialidade
FROM profissionais_especialidades pe
JOIN profissionais p ON p.usuario_id = pe.usuario_id
JOIN usuarios u ON u.id = p.usuario_id
JOIN especialidades e ON e.id = pe.especialidade_id
WHERE e.nome ILIKE '%Clínico Geral%';

Buscar profissionais por plano de saúde
SELECT u.nome, ps.nome AS plano_saude
FROM profissionais_planos pp
JOIN profissionais p ON p.usuario_id = pp.usuario_id
JOIN usuarios u ON u.id = p.usuario_id
JOIN planos_saude ps ON ps.id = pp.plano_saude_id
WHERE ps.nome ILIKE '%Unimed%';
