# Microsoft SQL Server 2022 – Procedures, Parâmetros e Cursores

## Funções Definidas pelo Usuário (UDF) – SQL Server

As **UDFs (User Defined Functions)** são funções criadas para encapsular uma lógica que retorna **um único valor escalar** ou uma **tabela**, permitindo reutilização e clareza em queries complexas.

### Diferenças entre Funções e Stored Procedures

| Característica         | Função (UDF)                         | Stored Procedure                   |
|------------------------|--------------------------------------|------------------------------------|
| Retorno obrigatório    | Sim (valor escalar ou tabela)        | Não (opcional, pode só executar)   |
| Permite SELECT         | Sim                                  | Sim                                |
| Permite INSERT/UPDATE  | Não diretamente                      | Sim                                |
| Permite uso em SELECT  | Sim                                  | Não                                |
| Suporta OUTPUT         | Não                                  | Sim                                |
| Pode ser usada como expressão | Sim                        | Não                                |

---

### Exemplo 1: Função que calcula idade com base na data de nascimento

```sql
CREATE FUNCTION dbo.CalculaIdade (@DataNascimento DATE)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @DataNascimento, GETDATE())
END

Uso:
SELECT NOME, dbo.CalculaIdade([DATA DE NASCIMENTO]) AS IdadeAtual
FROM [TABELA DE CLIENTES]


Exemplo 2: Função que retorna nota média de um cliente

CREATE FUNCTION dbo.MediaNotasCliente (@CPF VARCHAR(12))
RETURNS FLOAT
AS
BEGIN
    DECLARE @MEDIA FLOAT

    SELECT @MEDIA = AVG(NOTA)
    FROM [NOTAS]
    WHERE CPF = @CPF

    RETURN @MEDIA
END

Uso:
SELECT CPF, dbo.MediaNotasCliente(CPF) AS Media
FROM [CLIENTES]

Exemplo 3: Função que retorna uma tabela com clientes VIP
CREATE FUNCTION dbo.ClientesVIP ()
RETURNS TABLE
AS
RETURN
(
    SELECT CPF, NOME, [LIMITE DE CRÉDITO]
    FROM [TABELA DE CLIENTES]
    WHERE [LIMITE DE CRÉDITO] >= 10000
)

Uso:
SELECT * FROM dbo.ClientesVIP()


Alterar ou excluir funções:
-- Alterar função
ALTER FUNCTION dbo.CalculaIdade ...

-- Excluir função
DROP FUNCTION dbo.CalculaIdade




## Stored Procedures

Stored Procedures são rotinas armazenadas no servidor que encapsulam blocos de comandos T-SQL. Elas são úteis para automatizar tarefas repetitivas, atualizar dados em massa e aplicar lógica de negócios centralizada no banco.

### Estrutura básica de uma Stored Procedure:

```sql
CREATE PROCEDURE NomeDaProcedure
@Parametro1 Tipo,
@Parametro2 Tipo OUTPUT
AS
BEGIN
    -- Corpo da procedure
END

Exemplo 1: Atualizando a idade de todos os clientes
CREATE PROCEDURE CalculaIdade
AS
BEGIN
    UPDATE [TABELA DE CLIENTES]
    SET IDADE = DATEDIFF(YEAR, [DATA DE NASCIMENTO], GETDATE())
END

Uso:
EXEC CalculaIdade

Stored Procedure com Parâmetros OUTPUT
Permite retornar valores modificados para fora da procedure.

Exemplo 2: Calculando número de notas e faturamento de um cliente por ano
ALTER PROCEDURE retornaValoresFaturamentoQuantidade
@CPF VARCHAR(12),
@ANO INT,
@NUM_NOTAS INT OUTPUT,
@FATURAMENTO FLOAT OUTPUT
AS
BEGIN
    SELECT @NUM_NOTAS = COUNT(*) 
    FROM [NOTAS FISCAIS]
    WHERE CPF = @CPF AND YEAR([DATA]) = @ANO

    SELECT @FATURAMENTO = SUM(INF.QUANTIDADE * INF.[PREÇO])
    FROM [ITENS NOTAS FISCAIS] INF
    INNER JOIN [NOTAS FISCAIS] NF ON INF.NUMERO = NF.NUMERO
    WHERE NF.CPF = @CPF AND YEAR(NF.[DATA]) = @ANO
END

Chamada:
DECLARE @NUM_NOTAS INT = 0
DECLARE @FATURAMENTO FLOAT = 0

EXEC retornaValoresFaturamentoQuantidade 
    @CPF = '19290992743',
    @ANO = 2017,
    @NUM_NOTAS = @NUM_NOTAS OUTPUT,
    @FATURAMENTO = @FATURAMENTO OUTPUT

SELECT @NUM_NOTAS, @FATURAMENTO


CURSOR no SQL Server
CURSOR é utilizado para iterar sobre registros de forma controlada, linha a linha.

Exemplo 3: Listar os nomes dos 4 primeiros clientes
DECLARE @NOME VARCHAR(200)

DECLARE CURSOR1 CURSOR FOR
SELECT TOP 4 NOME FROM [TABELA DE CLIENTES]

OPEN CURSOR1
FETCH NEXT FROM CURSOR1 INTO @NOME

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT @NOME
    FETCH NEXT FROM CURSOR1 INTO @NOME
END

CLOSE CURSOR1
DEALLOCATE CURSOR1

CURSOR com múltiplas colunas
Exemplo 4: Exibir nome e endereço completo
DECLARE @NOME VARCHAR(200)
DECLARE @ENDERECO VARCHAR(MAX)

DECLARE CURSOR1 CURSOR FOR
SELECT NOME, ([ENDERECO 1] + ' ' + BAIRRO + ' ' + CIDADE + ' ' + ESTADO + ', CEP: ' + CEP) AS ENDCOMPLETO
FROM [TABELA DE CLIENTES]

OPEN CURSOR1
FETCH NEXT FROM CURSOR1 INTO @NOME, @ENDERECO

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT @NOME + ', Endereço: ' + @ENDERECO
    FETCH NEXT FROM CURSOR1 INTO @NOME, @ENDERECO
END

CLOSE CURSOR1
DEALLOCATE CURSOR1



Selecionar Cliente Aleatório
Exemplo 5: Selecionar CPF de cliente aleatório com uso de função externa
DECLARE @CLIENTE_ALEATORIO VARCHAR(12)
DECLARE @VAL_INICIAL INT = 1
DECLARE @VAL_FINAL INT
DECLARE @ALEATORIO INT
DECLARE @CONTADOR INT = 1

-- Obter número total de clientes
SELECT @VAL_FINAL = COUNT(*) FROM [TABELA DE CLIENTES]

-- Gerar número aleatório entre 1 e total de registros
SET @ALEATORIO = dbo.NumeroAleatorio(@VAL_INICIAL, @VAL_FINAL)

-- Cursor para percorrer CPFs
DECLARE CURSOR1 CURSOR FOR
SELECT CPF FROM [TABELA DE CLIENTES]

OPEN CURSOR1
FETCH NEXT FROM CURSOR1 INTO @CLIENTE_ALEATORIO

WHILE @CONTADOR < @ALEATORIO
BEGIN
    FETCH NEXT FROM CURSOR1 INTO @CLIENTE_ALEATORIO
    SET @CONTADOR = @CONTADOR + 1
END

CLOSE CURSOR1
DEALLOCATE CURSOR1

-- Exibir resultado
SELECT @CLIENTE_ALEATORIO AS CPF_Aleatorio, @ALEATORIO AS Posição


Gerenciamento de Procedures:
  Editar Procedure
ALTER PROCEDURE NomeDaProcedure
-- modificações

Excluir Procedure
DROP PROCEDURE [dbo].[NomeDaProcedure]


Observações finais:

Procedures aumentam a reutilização e performance do banco.
Parâmetros OUTPUT são ideais para retornos múltiplos.
CURSOR deve ser usado com cautela por questões de performance.
Recursos como PRINT, @@FETCH_STATUS, OUTPUT, FETCH, OPEN, CLOSE, DEALLOCATE e JOIN são fundamentais para lógica avançada em T-SQL.





