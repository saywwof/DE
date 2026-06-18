USE velosiped;
GO

DROP TABLE IF EXISTS dbo.Products;
GO

CREATE TABLE dbo.Products (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Article NVARCHAR(50) NOT NULL,
    Name NVARCHAR(300) NOT NULL,
    Unit NVARCHAR(50),
    Price DECIMAL(10,2),
    Supplier NVARCHAR(150),
    Manufacturer NVARCHAR(150),
    Category NVARCHAR(150),
    Photo NVARCHAR(255)
);
GO

INSERT INTO dbo.Products (
    Article,
    Name,
    Unit,
    Price,
    Supplier,
    Manufacturer,
    Category
)
SELECT
    [Артикул],
    [Наименование_товара],
    [Единица_измерения],
    TRY_CAST([Цена] AS DECIMAL(10,2)),
    [Поставщик],
    [Производитель],
    [Категория_товара]
FROM dbo.Tovar_import
WHERE [Артикул] IS NOT NULL
  AND [Наименование_товара] IS NOT NULL;
GO

SELECT * FROM dbo.Products;
