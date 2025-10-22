
-- =========================================================

-- PROJETO: Plataforma Lacrei Saúde
-- BANCO: PostgreSQL 16+
-- DESCRIÇÃO: Criação das tabelas e domínios conforme DER e dicionário de dados
-- =========================================================

-- =========================================================
-- EXTENSÕES
-- =========================================================
CREATE EXTENSION IF NOT EXISTS "pgcrypto"; -- para gen_random_uuid()

-- =========================================================
-- ENUMS E DOMÍNIOS
-- =========================================================
DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'genero_enum') THEN
        CREATE TYPE genero_enum AS ENUM (
            'cisgênero',
            'transgênero',
            'não-binário',
            'intersexo',
            'agênero',
            'outro',
            'prefiro_não_dizer'
        );
    END IF;
END
$$;

-- =========================================================
-- TABELA: tipo_usuario
-- =========================================================
CREATE TABLE IF NOT EXISTS tipo_usuario (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(50) NOT NULL UNIQUE,
    ativo BOOLEAN DEFAULT TRUE
);

INSERT INTO tipo_usuario (nome)
VALUES ('paciente'), ('profissional'), ('administrador')
ON CONFLICT (nome) DO NOTHING;

-- =========================================================
-- TABELA: planos_saude
-- =========================================================
CREATE TABLE IF NOT EXISTS planos_saude (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL UNIQUE,
    ativo BOOLEAN DEFAULT TRUE,
    criado_em TIMESTAMP DEFAULT now(),
    atualizado_em TIMESTAMP DEFAULT now()
);

-- =========================================================
-- TABELA: usuarios
-- =========================================================
CREATE TABLE IF NOT EXISTS usuarios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    tipo_usuario_id INT REFERENCES tipo_usuario(id),
    genero genero_enum,
    data_nascimento DATE,
    criado_em TIMESTAMP DEFAULT now(),
    atualizado_em TIMESTAMP DEFAULT now(),
    criado_por UUID NULL
);

CREATE INDEX IF NOT EXISTS idx_usuarios_email ON usuarios (email);
CREATE INDEX IF NOT EXISTS idx_usuarios_nome ON usuarios (nome);

-- =========================================================
-- TABELA: pacientes
-- =========================================================
CREATE TABLE IF NOT EXISTS pacientes (
    usuario_id UUID PRIMARY KEY REFERENCES usuarios(id),
    plano_saude_id INT REFERENCES planos_saude(id),
    prontuario JSONB,
    ativo BOOLEAN DEFAULT TRUE
);

CREATE INDEX IF NOT EXISTS idx_pacientes_prontuario_gin ON pacientes USING GIN (prontuario);
CREATE INDEX IF NOT EXISTS idx_pacientes_plano_saude_id ON pacientes (plano_saude_id);

-- =========================================================
-- TABELA: profissionais
-- =========================================================
CREATE TABLE IF NOT EXISTS profissionais (
    usuario_id UUID PRIMARY KEY REFERENCES usuarios(id),
    registro_profissional VARCHAR(30) NOT NULL UNIQUE,
    ativo BOOLEAN DEFAULT TRUE
);

CREATE INDEX IF NOT EXISTS idx_profissionais_registro ON profissionais (registro_profissional);

-- =========================================================
-- TABELA: especialidades
-- =========================================================
CREATE TABLE IF NOT EXISTS especialidades (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL UNIQUE,
    ativo BOOLEAN DEFAULT TRUE
);

-- =========================================================
-- TABELA: profissionais_especialidades (N:N)
-- =========================================================
CREATE TABLE IF NOT EXISTS profissionais_especialidades (
    usuario_id UUID REFERENCES profissionais(usuario_id),
    especialidade_id INT REFERENCES especialidades(id),
    ativo BOOLEAN DEFAULT TRUE,
    criado_em TIMESTAMP DEFAULT now(),
    PRIMARY KEY (usuario_id, especialidade_id)
);

CREATE INDEX IF NOT EXISTS idx_prof_esp_especialidade_id ON profissionais_especialidades (especialidade_id);

-- =========================================================
-- TABELA: profissionais_planos (N:N)
-- =========================================================
CREATE TABLE IF NOT EXISTS profissionais_planos (
    usuario_id UUID REFERENCES profissionais(usuario_id),
    plano_saude_id INT REFERENCES planos_saude(id),
    ativo BOOLEAN DEFAULT TRUE,
    criado_em TIMESTAMP DEFAULT now(),
    PRIMARY KEY (usuario_id, plano_saude_id)
);

CREATE INDEX IF NOT EXISTS idx_prof_plan_plano_id ON profissionais_planos (plano_saude_id);

-- =========================================================
-- TABELA: atendimentos
-- =========================================================
CREATE TABLE IF NOT EXISTS atendimentos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    paciente_id UUID NOT NULL REFERENCES pacientes(usuario_id),
    profissional_id UUID NOT NULL REFERENCES profissionais(usuario_id),
    data_hora TIMESTAMP NOT NULL CHECK (data_hora >= now()),
    descricao TEXT,
    criado_em TIMESTAMP DEFAULT now(),
    atualizado_em TIMESTAMP DEFAULT now(),
    ativo BOOLEAN DEFAULT TRUE,
    CONSTRAINT uk_atendimento_unico UNIQUE (profissional_id, data_hora),
    CONSTRAINT chk_prof_paciente_diferentes CHECK (profissional_id <> paciente_id)
);

CREATE INDEX IF NOT EXISTS idx_atendimentos_data_hora ON atendimentos (data_hora);
CREATE INDEX IF NOT EXISTS idx_atendimentos_profissional_id ON atendimentos (profissional_id);
CREATE INDEX IF NOT EXISTS idx_atendimentos_paciente_id ON atendimentos (paciente_id);

-- =========================================================
-- FINALIZAÇÃO
-- =========================================================
COMMENT ON DATABASE current_database() IS 'Banco de dados Lacrei Saúde — Versão de Modelagem Completa';
COMMENT ON SCHEMA public IS 'Schema principal da aplicação Lacrei Saúde';

