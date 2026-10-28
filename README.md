# 📚 Readit Inventory – Azure App Service, Azure SQL & Microsoft Entra ID

An **ASP.NET Core Razor Pages** web app that manages the book inventory of **Readit**, the fictional online bookstore used as the case study in the course
**[Microsoft Azure From Zero to Hero – The Complete Guide](https://www.udemy.com/course/microsoft-azure-from-zero-to-hero-the-complete-guide/)**.

The app lists the books stored in an **Azure SQL Database** and lets employees update the quantity in stock. It runs on **Azure App Service**, signs users in with **Microsoft Entra ID (Azure AD)**, and connects to the database using a **Managed Identity**, so there are no database passwords in the code or configuration.

> 📝 For my notes on the whole course, see **[docs/AZURE-COURSE-NOTES.md](docs/AZURE-COURSE-NOTES.md)**.

---

## 📑 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Data Model](#-data-model)
- [Configuration](#-configuration)
- [Building & Running Locally](#-building--running-locally)
- [Deploying to Azure](#-deploying-to-azure)
- [Course Reference](#-course-reference)

---

## ✨ Features

- Lists all books (name, author, pages, price) from Azure SQL.
- Update the **In Stock** quantity of any book.
- Sign-in with **Microsoft Entra ID** (`Microsoft.Identity.Web`); the header shows the signed-in user's name or *Anonymous User*.
- **Passwordless database access** with `Authentication=Active Directory Managed Identity`.
- Shows the database error on the page if the connection fails.

## 🛠 Tech Stack

| Layer          | Technology                                         |
| -------------- | -------------------------------------------------- |
| Framework      | ASP.NET Core on **.NET 10** (Razor Pages)          |
| Language       | C#                                                 |
| ORM            | Entity Framework Core 10 (SQL Server provider)     |
| Database       | Azure SQL Database                                 |
| Hosting        | Azure App Service (Web App)                        |
| Authentication | Microsoft Entra ID via `Microsoft.Identity.Web`    |
| DB auth        | System-assigned Managed Identity (`Azure.Identity`)|
| Front end      | Bootstrap, jQuery                                  |

## 🏗 Architecture

```text
                     ① Sign in (OpenID Connect)
            ┌──────────────────────────────────────┐
            ▼                                      │
   ┌─────────────────┐                    ┌────────┴────────┐
   │ Microsoft Entra │                    │    Browser      │
   │ ID (Azure AD)   │                    └────────┬────────┘
   └─────────────────┘                             │ ② HTTPS
                                                   ▼
                                         ┌───────────────────┐
                                         │ Azure App Service │
                                         │  (inventory app)  │
                                         │  Managed Identity │
                                         └─────────┬─────────┘
                                                   │ ③ EF Core, token-based auth
                                                   ▼
                                         ┌───────────────────┐
                                         │ Azure SQL Database│
                                         │ readit-db / Books │
                                         └───────────────────┘
```

1. The user signs in through **Microsoft Entra ID**.
2. The request reaches the app on **Azure App Service**.
3. `IndexModel` queries the `Books` table through `BookContext` (EF Core). The app authenticates to Azure SQL with its **Managed Identity**, so no password is stored anywhere.

## 📂 Project Structure

```text
inventory/
├── Data/BookContext.cs          # EF Core DbContext (Books DbSet)
├── Models/Book.cs               # Book entity
├── Pages/
│   ├── Index.cshtml(.cs)        # Inventory table + save stock handler
│   └── Shared/_Layout.cshtml    # Layout, shows signed-in user
├── wwwroot/                     # Static files (CSS, JS, images, libs)
├── docs/AZURE-COURSE-NOTES.md   # Course notes
├── Program.cs                   # Entry point
├── Startup.cs                   # Razor Pages, Entra ID auth, DbContext
├── appsettings.json             # Connection string + AzureAd section
└── inventory.csproj             # net10.0
```

## 🗃 Data Model

`Models/Book.cs` maps to the `Books` table:

| Column   | Type     | Description       |
| -------- | -------- | ----------------- |
| ID       | int (PK) | Book identifier   |
| Name     | nvarchar | Book title        |
| Pages    | int      | Number of pages   |
| Author   | nvarchar | Author name       |
| ImageUrl | nvarchar | Cover image URL   |
| Price    | float    | Price             |
| InStock  | int      | Quantity in stock |

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

## ⚙️ Configuration

`appsettings.json`:

```json
"ConnectionStrings": {
  "BooksDB": "Server=tcp:<server>.database.windows.net,1433;Initial Catalog=<db>;Authentication=Active Directory Managed Identity;"
},
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "your-domain.onmicrosoft.com",
  "TenantId": "your-tenant-id",
  "ClientId": "your-client-id",
  "CallbackPath": "/signin-oidc"
}
```

| Setting               | Where it comes from                                                  |
| --------------------- | -------------------------------------------------------------------- |
| `BooksDB`             | Azure SQL server & database names. No password is needed.            |
| `AzureAd:TenantId`    | Entra ID → **Overview** → Directory (tenant) ID                      |
| `AzureAd:ClientId`    | Entra ID → **App registrations** → your app → Application (client) ID |
| `AzureAd:Domain`      | Entra ID → **Overview** → Primary domain                             |

> In App Service you can override any of these with **Environment variables**, for example `AzureAd__ClientId`.

## 💻 Building & Running Locally

**Prerequisites:** [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0).

```bash
git clone https://github.com/ahmedmabdulla2020-tech/inventory.git
cd inventory
dotnet build inventory.csproj
dotnet run --project inventory.csproj
```

The app runs at `http://localhost:5002`.

> **Note:** `Active Directory Managed Identity` only works inside Azure. Locally, use `Authentication=Active Directory Default` in the connection string (it uses your `az login` / Visual Studio account). Also add your client IP to the SQL Server firewall.

## ☁️ Deploying to Azure

1. **Azure SQL Database**: create the server and database, run the SQL script above, and set a **Microsoft Entra admin** on the server.
2. **App Service**: create a Web App with runtime **.NET 10**.
3. **Managed Identity**: Web App → **Identity** → System assigned → **On**.
4. **Grant the identity access to the database** (run as the Entra admin):

   ```sql
   CREATE USER [<web-app-name>] FROM EXTERNAL PROVIDER;
   ALTER ROLE db_datareader ADD MEMBER [<web-app-name>];
   ALTER ROLE db_datawriter ADD MEMBER [<web-app-name>];
   ```

5. **Networking**: on the SQL server, allow **Azure services and resources to access this server**.
6. **App registration**: in Entra ID, register the app and add the redirect URI `https://<web-app-name>.azurewebsites.net/signin-oidc`. Put the tenant ID and client ID in the configuration.
7. **Publish** the app:

   ```bash
   dotnet publish inventory.csproj -c Release -o ./publish
   ```

   Deploy the `publish` folder from Visual Studio, from the VS Code Azure extension, or with `az webapp deploy`.

## 📘 Course Reference

- **Course:** Microsoft Azure From Zero to Hero – The Complete Guide (Udemy)
- **Instructor:** Memi Lavi
- **Notes:** [docs/AZURE-COURSE-NOTES.md](docs/AZURE-COURSE-NOTES.md)

This repository contains my own practice work while following the course.
