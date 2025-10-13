mkdir modelagem

Campo 

Tipo 

Restrições 

Descrição 

ID 

INT 

PK, Auto Increment 

Identificador único da pessoa 

Nome 

VARCHAR(100) 

NOT NULL 

Nome completo da pessoa 

Email 

VARCHAR(150) 

UNIQUE, NOT NULL 

E-mail da pessoa 

TipoPessoa 

ENUM 

NOT NULL, valores: 'cliente', 'funcionario' 

Tipo de pessoa 

Salario 

DECIMAL(10,2) 

NULL 

Somente para funcionários 

DataCadastro 

DATE 

NULL 

Somente para clientes 

 

Proposta 2 

️ Modelo DER (simplificado) 

Pessoa 

------ 

ID (PK) 

Nome 

Email 

  

Cliente 

------- 

ID (PK, FK → Pessoa.ID) 

DataCadastro 

  

Funcionario 

----------- 

ID (PK, FK → Pessoa.ID) 

Salario 

DICIONÁRIO DE DADOS 

Campo 

Tipo 

Restrições 

ID 

INT 

PK, Auto Increment 

Nome 

VARCHAR(100) 

NOT NULL 

Email 

VARCHAR(150) 

UNIQUE, NOT NULL 

Campo 

Tipo 

Restrições 

ID 

INT 

PK, FK → Pessoa.ID 

DataCadastro 

DATE 

NOT NULL 

Campo 

Tipo 

Restrições 

ID 

INT 

PK, FK → Pessoa.ID 

Salario 

DECIMAL(10,2) 

NOT NULL 

 # Dba-Modelagem-de-Dados
Desafio Técnico
