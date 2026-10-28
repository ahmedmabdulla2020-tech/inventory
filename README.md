# 📚 Inventory – Azure App Service + Azure SQL Demo

A small **ASP.NET Core Razor Pages** web app for managing a bookstore's inventory, built and deployed to **Microsoft Azure** as part of the course
**[Microsoft Azure From Zero to Hero – The Complete Guide](https://www.udemy.com/course/microsoft-azure-from-zero-to-hero-the-complete-guide/)**.

The app lists books stored in an **Azure SQL Database** and lets you update the quantity in stock for each book. Its main purpose is to practice hosting a web app on **Azure App Service** and connecting it securely to a managed database.

---

## 📑 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Data Model](#-data-model)
- [Running Locally](#-running-locally)
- [Deploying to Azure](#-deploying-to-azure)
- [Azure Concepts Practiced](#-azure-concepts-practiced)
- [Lessons Learned](#-lessons-learned)
- [Course Reference](#-course-reference)

---

## ✨ Features

- Displays all books (name, author, pages, price) from the database.
- Update the **In Stock** quantity for any book with a single click.
- Connection string read from configuration, so it can be supplied by **Azure App Service settings** instead of being hard-coded.
- Friendly error message on the page if the database can't be reached.

## 🛠 Tech Stack

| Layer          | Technology                                   |
| -------------- | -------------------------------------------- |
| Framework      | ASP.NET Core 3.1 (Razor Pages)               |
| Language       | C#                                           |
| ORM            | Entity Framework Core 3.1 (SQL Server)       |
| Database       | Azure SQL Database                           |
| Hosting        | Azure App Service (Web App)                  |
| Front end      | Bootstrap, jQuery                            |

## 🏗 Architecture

```text
             ┌──────────────┐
  Browser ──▶│ Azure App    │   EF Core / TCP 1433   ┌────────────────────┐
             │ Service      │ ─────────────────────▶ │ Azure SQL Database │
             │ (inventory)  │                        │  (Books table)     │
             └──────────────┘                        └────────────────────┘
                    ▲
                    │  Connection string "BooksDB"
                    │  (App Service → Configuration → Connection strings)
```

1. The user opens the web app hosted on **Azure App Service**.
2. `IndexModel` uses `BookContext` (EF Core) to query the `Books` table.
3. When the user clicks **Save**, the new quantity is written back to **Azure SQL Database**.

## 📂 Project Structure

```text
inventory/
├── Data/
│   └── BookContext.cs        # EF Core DbContext (Books DbSet)
├── Models/
│   └── Book.cs               # Book entity
├── Pages/
│   ├── Index.cshtml          # Inventory table UI
│   ├── Index.cshtml.cs       # Page logic: load books / save stock
│   └── Shared/_Layout.cshtml # Site layout
├── wwwroot/                  # Static files (CSS, JS, images, libs)
├── Program.cs                # App entry point
├── Startup.cs                # Services (Razor Pages, DbContext) & pipeline
├── appsettings.json          # Config – contains the "BooksDB" placeholder
└── inventory.csproj
```

## 🗃 Data Model

`Models/Book.cs` maps to the `Books` table:

| Column    | Type            | Description            |
| --------- | --------------- | ---------------------- |
| ID        | int (PK)        | Book identifier        |
| Name      | nvarchar        | Book title             |
| Pages     | int             | Number of pages        |
| Author    | nvarchar        | Author name            |
| ImageUrl  | nvarchar        | Cover image URL        |
| Price     | float           | Price                  |
| InStock   | int             | Quantity in stock      |

Example SQL to create and seed the table:

```sql
CREATE TABLE Books (
    ID       INT IDENTITY(1,1) PRIMARY KEY,
    Name     NVARCHAR(200),
    Pages    INT,
    Author   NVARCHAR(200),
    ImageUrl NVARCHAR(500),
    Price    FLOAT,
    InStock  INT
);

INSERT INTO Books (Name, Pages, Author, ImageUrl, Price, InStock)
VALUES ('The Pragmatic Programmer', 352, 'Andrew Hunt', '', 39.99, 10),
       ('Clean Code', 464, 'Robert C. Martin', '', 34.50, 5);
```

## 💻 Running Locally

**Prerequisites:** [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet/3.1) (or newer SDK able to build `netcoreapp3.1`) and access to a SQL Server / Azure SQL database.

1. Clone the repository:

   ```bash
   git clone <your-repo-url>
   cd inventory
   ```

2. Set the connection string. **Don't commit real credentials** – use user secrets instead:

   ```bash
   dotnet user-secrets init
   dotnet user-secrets set "ConnectionStrings:BooksDB" "Server=tcp:<server>.database.windows.net,1433;Database=<db>;User ID=<user>;Password=<password>;Encrypt=True;"
   ```

3. In `Pages/Index.cshtml.cs`, uncomment the block marked `UNCOMMENT AFTER SETTING THE CONNECTION STRING` so the page loads books from the database.

4. Run the app:

   ```bash
   dotnet run
   ```

   Then open `http://localhost:5002`.

## ☁️ Deploying to Azure

### 1. Create the Azure SQL Database

1. In the Azure Portal, create a **SQL Database** (and a new logical **SQL Server**).
2. Under the server's **Networking**, allow **Azure services and resources to access this server**, and add your client IP if you want to connect from your machine.
3. Run the SQL script above (using the Portal **Query editor**, Azure Data Studio, or SSMS).
4. Copy the **ADO.NET connection string** from the database's **Connection strings** blade.

### 2. Create the App Service

1. Create a **Web App** with runtime stack **.NET Core 3.1** (or a newer .NET version if you upgrade the project).
2. Choose an **App Service Plan** (Free F1 / Basic B1 are enough for learning).

### 3. Configure the connection string

In the Web App → **Configuration** (or **Environment variables**) → **Connection strings**, add:

| Name      | Value                          | Type      |
| --------- | ------------------------------ | --------- |
| `BooksDB` | *your ADO.NET connection string* | SQLAzure  |

App Service injects it at runtime, overriding the placeholder in `appsettings.json`.

### 4. Publish the app

Using the .NET CLI:

```bash
dotnet publish -c Release -o ./publish
```

Then deploy the `publish` folder using one of:

- **Visual Studio** → right-click the project → **Publish** → Azure App Service.
- **VS Code** with the **Azure App Service** extension → *Deploy to Web App*.
- **Azure CLI**:

  ```bash
  az webapp deploy --resource-group <rg> --name <app-name> --src-path publish.zip --type zip
  ```

Browse to `https://<app-name>.azurewebsites.net` to see the inventory page.

## 🎓 Azure Concepts Practiced

- **Resource Groups** – organizing related resources and deleting them together.
- **Azure App Service** – PaaS hosting for web apps, App Service Plans, scaling up/out.
- **Azure SQL Database** – managed relational database, server firewall rules, DTU/vCore tiers.
- **Configuration management** – App Service application settings & connection strings instead of secrets in code.
- **Deployment** – publishing from Visual Studio / VS Code / CLI.
- **Cost awareness** – choosing low tiers for learning and cleaning up resources afterwards.

## 💡 Lessons Learned

- Keep secrets out of source control; App Service **Connection strings** override `appsettings.json` values with the same name.
- The SQL Server firewall blocks everything by default – remember to allow Azure services and your client IP.
- Always **delete the resource group** when you finish experimenting to avoid unexpected charges.

## 📘 Course Reference

- **Course:** Microsoft Azure From Zero to Hero – The Complete Guide (Udemy)
- **Instructor:** Memi Lavi

This repository contains my own practice work while following the course.

---

⭐ If you find this helpful, feel free to star the repo!
