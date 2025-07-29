# Fundamentos de Microsoft SQL Server 2022

## Ambiente de Desenvolvimento

- Instalação do SQL Server 2022 como SGBD local
- Instalação do SQL Server Management Studio (IDE)
- Criação e exclusão de bancos de dados
- Comparação entre SSMS e SQL Server

**Aplicação:** configuração de ambiente local para desenvolvimento, testes e simulação de bancos de dados relacionais completos.

---

## Criação de Tabelas e Modelagem

Criação de tabelas com definição de tipos e restrições estruturais:

```sql
CREATE DATABASE Frutally;
GO

CREATE TABLE Produtos (
    id INT PRIMARY KEY,
    nome VARCHAR(100),
    preco DECIMAL(10, 2)
);

Principais tópicos abordados:

Tipos de dados: VARCHAR, INT, DECIMAL

Definição de chave primária (PRIMARY KEY)

Alteração de tabelas com ALTER TABLE

Exclusão de tabelas e bancos com DROP


Inserção de Dados:
  Múltiplas abordagens para inserção de dados em tabelas:
  INSERT INTO Produtos (id, nome, preco) VALUES (1, 'Banana', 3.50);

Consulta de Dados (SELECT):
  Uso do comando SELECT para leitura e filtragem de dados:
  SELECT nome, preco FROM Produtos;

Seleção de colunas específicas
Apelidamento de colunas com AS
Eliminação de duplicatas com DISTINCT
Filtros com WHERE, ex:
  SELECT * FROM Produtos WHERE preco > 5.00;

Operadores Lógicos e Condições
Utilização de operadores para filtros condicionais:
SELECT * FROM Produtos
WHERE preco > 3 AND nome LIKE 'B%';

Operadores utilizados:
AND, OR, NOT
Combinações de condições
Filtros com strings, números e padrões (LIKE)

Atualização e Exclusão de Dados
Comandos de modificação com foco em segurança e precisão:
UPDATE Produtos SET preco = 4.00 WHERE nome = 'Banana';
DELETE FROM Produtos WHERE preco < 2.00;

Atualização seletiva com UPDATE
Exclusão segura com DELETE
Sempre com uso obrigatório de WHERE para evitar alterações em massa
















