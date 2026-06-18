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




<Window x:Class="VelosipedDrive.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Вход в систему"
        Height="360"
        Width="420"
        WindowStartupLocation="CenterScreen"
        ResizeMode="NoResize">

    <Grid Margin="25">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Text="ООО Велосипед Драйв"
                   FontSize="24"
                   FontWeight="Bold"
                   HorizontalAlignment="Center"
                   Margin="0,0,0,25"/>

        <StackPanel Grid.Row="1" Margin="0,0,0,12">
            <TextBlock Text="Логин"/>
            <TextBox x:Name="LoginTextBox"
                     Height="32"
                     FontSize="16"/>
        </StackPanel>

        <StackPanel Grid.Row="2" Margin="0,0,0,20">
            <TextBlock Text="Пароль"/>
            <PasswordBox x:Name="PasswordBox"
                         Height="32"
                         FontSize="16"/>
        </StackPanel>

        <StackPanel Grid.Row="3">
            <Button Content="Войти"
                    Height="36"
                    Margin="0,0,0,10"
                    Click="LoginButton_Click"/>

            <Button Content="Просмотр товаров без авторизации"
                    Height="36"
                    Click="GuestButton_Click"/>
        </StackPanel>
    </Grid>
</Window>





using System;
using System.Data.SqlClient;
using System.Windows;

namespace VelosipedDrive
{
    public partial class MainWindow : Window
    {
        private string connectionString =
            @"Data Source=.\SQLEXPRESS;Initial Catalog=velosiped;Integrated Security=True;TrustServerCertificate=True;";

        public MainWindow()
        {
            InitializeComponent();
        }

        private void LoginButton_Click(object sender, RoutedEventArgs e)
        {
            string login = LoginTextBox.Text.Trim();
            string password = PasswordBox.Password.Trim();

            if (string.IsNullOrWhiteSpace(login) || string.IsNullOrWhiteSpace(password))
            {
                MessageBox.Show("Введите логин и пароль");
                return;
            }

            try
            {
                using (SqlConnection connection = new SqlConnection(connectionString))
                {
                    connection.Open();

                    string query = @"
                        SELECT TOP 1 *
                        FROM dbo.user_import
                        WHERE Логин = @login AND Пароль = @password";

                    SqlCommand command = new SqlCommand(query, connection);
                    command.Parameters.AddWithValue("@login", login);
                    command.Parameters.AddWithValue("@password", password);

                    SqlDataReader reader = command.ExecuteReader();

                    if (reader.Read())
                    {
                        string role = "";

                        if (reader["Роль"] != DBNull.Value)
                            role = reader["Роль"].ToString();

                        ProductsWindow productsWindow = new ProductsWindow(role);
                        productsWindow.Show();

                        this.Close();
                    }
                    else
                    {
                        MessageBox.Show("Неверный логин или пароль");
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка подключения или авторизации:\n" + ex.Message);
            }
        }

        private void GuestButton_Click(object sender, RoutedEventArgs e)
        {
            ProductsWindow productsWindow = new ProductsWindow("Гость");
            productsWindow.Show();

            this.Close();
        }
    }
}
