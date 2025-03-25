# Projeto_E-COMMERCE

# Passo a Passo para Implementação do Sistema de Oficina Mecânica
1. Requisitos do Sistema
1.1. Entidades Principais
Clientes

Veículos

Equipes de mecânicos

Ordens de Serviço (OS)

Serviços

Peças

1.2. Funcionalidades Principais
Cadastro de clientes e veículos

Criação e gerenciamento de OS

Controle de serviços e peças

Cálculo automático de valores

Autorização de serviços pelo cliente

2. Modelagem do Banco de Dados
2.1. Diagrama Entidade-Relacionamento
```
erDiagram
    CLIENTE ||--o{ VEICULO : "POSSUI"
    CLIENTE {
        string CPF PK
        string Nome
        string Telefone
        string Endereço
        string Email
    }
    
    VEICULO {
        string Placa PK
        string Marca
        string Modelo
        int Ano
        string Cor
        string CPF_Cliente FK
    }
    
    EQUIPE ||--o{ MECANICO : "COMPÕE"
    EQUIPE {
        int Codigo_Equipe PK
        string Nome_Equipe
        string Especialidade
    }
    
    MECANICO {
        string CPF PK
        string Nome
        string Endereço
        string Telefone
        string Especialidade
        int Codigo_Equipe FK
    }
    
    EQUIPE ||--o{ ORDEM_SERVICO : "ATENDE"
    VEICULO ||--o{ ORDEM_SERVICO : "AVALIADO_POR"
    
    ORDEM_SERVICO {
        int Numero_OS PK
        date Data_Emissao
        date Data_Entrega_Prevista
        date Data_Entrega_Real
        string Status
        decimal Valor_Total
        string CPF_Cliente FK
        string Placa_Veiculo FK
        int Codigo_Equipe FK
    }
    
    ORDEM_SERVICO }|--|{ SERVICO : "CONTÉM"
    SERVICO {
        int Codigo_Servico PK
        string Descricao
        time Tempo_Estimado
        decimal Valor_Mao_de_Obra
    }
    
    ORDEM_SERVICO }|--|{ PECA : "UTILIZA"
    PECA {
        int Codigo_Peca PK
        string Descricao
        decimal Valor_Unitario
        int Quantidade_Estoque
    }
   
```
2.2. Script SQL para Criação do Banco
```
CREATE TABLE CLIENTE (
    CPF VARCHAR(11) PRIMARY KEY,
    Nome VARCHAR(100) NOT NULL,
    Telefone VARCHAR(15),
    Endereço VARCHAR(200),
    Email VARCHAR(100)
);

CREATE TABLE VEICULO (
    Placa VARCHAR(7) PRIMARY KEY,
    Marca VARCHAR(50) NOT NULL,
    Modelo VARCHAR(50) NOT NULL,
    Ano INT,
    Cor VARCHAR(30),
    CPF_Cliente VARCHAR(11) REFERENCES CLIENTE(CPF)
);

CREATE TABLE EQUIPE (
    Codigo_Equipe INT PRIMARY KEY,
    Nome_Equipe VARCHAR(50) NOT NULL,
    Especialidade VARCHAR(100)
);

CREATE TABLE MECANICO (
    CPF VARCHAR(11) PRIMARY KEY,
    Nome VARCHAR(100) NOT NULL,
    Endereço VARCHAR(200),
    Telefone VARCHAR(15),
    Especialidade VARCHAR(100),
    Codigo_Equipe INT REFERENCES EQUIPE(Codigo_Equipe)
);

CREATE TABLE SERVICO (
    Codigo_Servico INT PRIMARY KEY,
    Descricao VARCHAR(200) NOT NULL,
    Tempo_Estimado TIME,
    Valor_Mao_de_Obra DECIMAL(10,2) NOT NULL
);

CREATE TABLE PECA (
    Codigo_Peca INT PRIMARY KEY,
    Descricao VARCHAR(200) NOT NULL,
    Valor_Unitario DECIMAL(10,2) NOT NULL,
    Quantidade_Estoque INT DEFAULT 0
);

CREATE TABLE ORDEM_SERVICO (
    Numero_OS INT PRIMARY KEY,
    Data_Emissao DATE NOT NULL,
    Data_Entrega_Prevista DATE,
    Data_Entrega_Real DATE,
    Status VARCHAR(20) DEFAULT 'Aberta',
    Valor_Total DECIMAL(10,2) DEFAULT 0,
    CPF_Cliente VARCHAR(11) REFERENCES CLIENTE(CPF),
    Placa_Veiculo VARCHAR(7) REFERENCES VEICULO(Placa),
    Codigo_Equipe INT REFERENCES EQUIPE(Codigo_Equipe)
);

-- Tabelas de relacionamento N:M
CREATE TABLE OS_SERVICO (
    Numero_OS INT REFERENCES ORDEM_SERVICO(Numero_OS),
    Codigo_Servico INT REFERENCES SERVICO(Codigo_Servico),
    Quantidade INT DEFAULT 1,
    Valor_Unitario DECIMAL(10,2),
    Valor_Total DECIMAL(10,2),
    PRIMARY KEY (Numero_OS, Codigo_Servico)
);

CREATE TABLE OS_PECA (
    Numero_OS INT REFERENCES ORDEM_SERVICO(Numero_OS),
    Codigo_Peca INT REFERENCES PECA(Codigo_Peca),
    Quantidade INT DEFAULT 1,
    Valor_Unitario DECIMAL(10,2),
    Valor_Total DECIMAL(10,2),
    PRIMARY KEY (Numero_OS, Codigo_Peca)
);
```
# 3. Implementação da Aplicação

```
# Criar projeto (exemplo para Node.js)
mkdir oficina-mecanica
cd oficina-mecanica
npm init -y
npm install express sequelize pg pg-hstore cors
npm install --save-dev nodemon
```
# 3.2. Estrutura de Pastas
```
/oficina-mecanica
│
├── /src
│   ├── /controllers
│   ├── /models
│   ├── /routes
│   ├── /services
│   └── app.js
│
├── .gitignore
├── package.json
└── README.md
```
3.3. Exemplo de Modelo (Sequelize)
```
// src/models/Cliente.js
const { Model, DataTypes } = require('sequelize');

class Cliente extends Model {
    static init(sequelize) {
        super.init({
            CPF: {
                type: DataTypes.STRING(11),
                primaryKey: true
            },
            Nome: DataTypes.STRING,
            Telefone: DataTypes.STRING,
            Endereco: DataTypes.STRING,
            Email: DataTypes.STRING
        }, { sequelize });
    }
    
    static associate(models) {
        this.hasMany(models.Veiculo, { foreignKey: 'CPF_Cliente', as: 'veiculos' });
    }
}

module.exports = Cliente;
```
3.4. Exemplo de Controller
```
// src/controllers/OSController.js
const OS = require('../models/OrdemServico');

class OSController {
    async criarOS(req, res) {
        try {
            const { clienteCPF, veiculoPlaca, equipeId, servicos, pecas } = req.body;
            
            // Cálculo do valor total
            let valorTotal = 0;
            
            // Adicionar lógica para calcular serviços e peças
            
            const novaOS = await OS.create({
                Data_Emissao: new Date(),
                Status: 'Aberta',
                Valor_Total: valorTotal,
                CPF_Cliente: clienteCPF,
                Placa_Veiculo: veiculoPlaca,
                Codigo_Equipe: equipeId
            });
            
            // Adicionar relacionamentos com serviços e peças
            
            return res.status(201).json(novaOS);
        } catch (error) {
            return res.status(400).json({ error: error.message });
        }
    }
    
    // Outros métodos (listar, atualizar, etc.)
}

module.exports = new OSController();
```
# 4. Testes e Validações
## 4.1. Testes Unitários
```
// tests/OS.test.js
const request = require('supertest');
const app = require('../src/app');
const OS = require('../src/models/OrdemServico');

describe('Testes de Ordem de Serviço', () => {
    beforeEach(async () => {
        await OS.destroy({ where: {} });
    });
    
    it('deve criar uma nova OS', async () => {
        const response = await request(app)
            .post('/os')
            .send({
                clienteCPF: '12345678901',
                veiculoPlaca: 'ABC1234',
                equipeId: 1
            });
            
        expect(response.status).toBe(201);
        expect(response.body).toHaveProperty('Numero_OS');
    });
});
```
# 4.2. Testes de Integração
```
// tests/integration/clienteOS.test.js
describe('Integração Cliente-OS', () => {
    it('deve listar todas as OS de um cliente', async () => {
        // Criar cliente e OS de teste primeiro
        const response = await request(app)
            .get('/clientes/12345678901/os');
            
        expect(response.status).toBe(200);
        expect(response.body).toBeInstanceOf(Array);
    });
});
```
5. Implantação
5.1. Dockerfile
```
FROM node:14

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```
5.2. docker-compose.yml
```
version: '3'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - DB_NAME=oficina
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=oficina
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```
# 6. Documentação da API
## 6.1. Endpoints Principais
Método	Endpoint	Descrição
POST	/clientes	Cadastrar novo cliente
GET	/clientes/{cpf}/os	Listar OS de um cliente
POST	/os	Criar nova ordem de serviço
PUT	/os/{id}/autorizar	Autorizar execução de serviços
GET	/os/{id}/relatorio	Gerar relatório completo da OS

6.2. Exemplo de Request/Response
```
// POST /os
{
  "clienteCPF": "12345678901",
  "veiculoPlaca": "ABC1234",
  "equipeId": 1,
  "servicos": [
    {
      "codigo": 101,
      "quantidade": 1
    }
  ],
  "pecas": [
    {
      "codigo": 205,
      "quantidade": 2
    }
  ]
}

// Response 201
{
  "Numero_OS": 1001,
  "Valor_Total": 450.00,
  "Status": "Aguardando autorização"
}
```




