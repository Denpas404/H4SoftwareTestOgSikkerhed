# Step-by-Step Guide: Installing Visual Studio, WSL 1, and Running .NET 8 Projects with SQLite

This guide will walk you through the process of:
1. Installing Visual Studio on your Windows machine,
2. Installing and configuring WSL 1 with Ubuntu,
3. Manually enabling the required Windows features for WSL,
4. Fixing the Untrusted Certificate error in ASP.NET Core,
5. Replacing LocalDB with SQLite in your .NET 8 project,
6. Running your .NET 8 project using WSL in Visual Studio,
7. Applying EF Core migrations to create your MockDB.

---

## Table of Contents
1. [Install Visual Studio](#install-visual-studio)
2. [Install WSL](#install-wsl)
3. [Manually Enable WSL Features](#manually-enable-wsl-features)
4. [Change DNS to 8.8.8.8](#change-dns-to-8888)
5. [Fix Untrusted Certificate Error](#fix-untrusted-certificate-error)
6. [Use SQLite for Database](#use-sqlite-for-database)
7. [Run .NET 8 Project in Visual Studio as WSL](#run-net-8-project-in-visual-studio-as-wsl)
8. [Apply EF Core Migrations to Create MockDB](#apply-ef-core-migrations-to-create-mockdb)

---

## 1. Install Visual Studio
Follow these steps to install Visual Studio on your Windows machine:

1. **Go to the Visual Studio website:**
   - Visit the [Visual Studio download page](https://visualstudio.microsoft.com/downloads/).

2. **Download the installer:**
   - Choose the **Community edition** (free for individual use) or another edition that suits your needs.
   - Download the installer and run it.

3. **Select the necessary workload:**
   - When the installer opens, select the **Desktop development with C++** workload. This workload includes the necessary compilers and libraries for C++ or C# development.
   - You can choose additional workloads based on your needs, but **Visual Studio Code** does not need to be selected.

4. **Complete the installation:**
   - Click **Install** to start the process. Visual Studio will download and install the selected components.
   - Once installed, launch Visual Studio to ensure it�s working properly.

---

## 2. Install WSL
After installing Visual Studio, proceed with installing WSL 1 and configuring it with Ubuntu.

1. **Open PowerShell as Administrator:**
   - Right-click on the Start button, and select **Windows Terminal (Admin)** or **PowerShell (Admin)**.

2. **Install WSL:**
   - Run the following command in PowerShell to install WSL:
     ```powershell
     wsl --install
     ```
   - This command will:
     - Enable WSL on your system.
     - Install the default Linux distribution (usually Ubuntu).

3. **Install Ubuntu from the Microsoft Store:**
   - Open the Microsoft Store on your Windows machine.
   - Search for **Ubuntu** (e.g., Ubuntu 22.04) and click **Install**.
   - Wait for the installation to complete.

4. **Restart your computer:**
   - If prompted, restart your system to complete the installation.

5. **Launch Ubuntu:**
   - After the installation completes, go to the Start Menu, search for **Ubuntu**, and launch it.
   - The first time you run Ubuntu, you will be prompted to create a user account and set a password for your Ubuntu system.

---

## 3. Manually Enable WSL Features (If `wsl --install` Doesn�t Work)
If the `wsl --install` command doesn�t automatically enable the necessary Windows features for WSL to work, manually enable them before restarting your system.

1. **Open PowerShell as Administrator:**
   - Right-click on the Start button, and select **Windows Terminal (Admin)** or **PowerShell (Admin)**.

2. **Manually Enable the Required Features:**
   - Enable WSL Feature:
     ```powershell
     dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
     ```
   - Enable Virtual Machine Platform Feature:
     ```powershell
     dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
     ```

3. **Optional: Enable Hyper-V (Required for WSL 2):**
   - If you plan to use WSL 2 in the future, you can also enable Hyper-V:
     ```powershell
     dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart
     ```

4. **Restart Your Computer:**
   - After running the commands, restart your computer to apply the changes.

---

## 4. Change DNS to 8.8.8.8
After Ubuntu is installed in WSL, you may need to change the DNS settings to avoid potential DNS resolution issues.

1. **Navigate to the WSL File System:**
   - Open File Explorer and type the following in the address bar:
     ```
     \\wsl.localhost\Ubuntu-22.04\mnt\wsl
     ```

2. **Edit the `resolv.conf` File:**
   - Open the Ubuntu terminal and run:
     ```bash
     sudo nano /etc/resolv.conf
     ```

3. **Modify the DNS Settings:**
   - Change the `nameserver` line to:
     ```bash
     nameserver 8.8.8.8
     ```

4. **Save the Changes:**
   - Press **Ctrl + X** to exit the editor.
   - Press **Y** to confirm saving the file.
   - Press **Enter** to confirm the filename and save it.

---

## 5. Fix Untrusted Certificate Error
To resolve the Untrusted Certificate error (common with ASP.NET Core and HTTPS), follow these steps to create the necessary folders in `%APPDATA%`.

1. **Navigate to `%APPDATA%`:**
   - Open File Explorer and type the following in the address bar:
     ```
     %APPDATA%
     ```

2. **Create the Necessary Folders:**
   - Inside the `AppData\Roaming` folder, create a new folder called `ASP.NET`.
   - Inside the `ASP.NET` folder, create a new folder called `Https`.

   The final folder structure should look like:

---

## 6. Use SQLite for Database
Since WSL cannot run LocalDB, you'll need to configure your .NET 8 project to use SQLite instead. Follow these steps to set up the MockDB using SQLite:

1. **Install `Microsoft.EntityFrameworkCore.Sqlite`:**
- In Visual Studio, go to **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**.
- In the NuGet Package Manager window, search for `Microsoft.EntityFrameworkCore.Sqlite`.
- Click **Install** to add the SQLite provider to your project.

2. **Update `appsettings.json`:**
- Open your `appsettings.json` file in your project.
- Add a new connection string for SQLite:
  ```json
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=mockdb.db"
  }
  ```
- Here, `mockdb.db` is the SQLite database file that will be created in your project directory.

3. **Modify `Startup.cs` or `Program.cs`:**
- In `Startup.cs` or `Program.cs`, modify the database configuration to use SQLite instead of SQL Server:
  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddDbContext<ApplicationDbContext>(options =>
          options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
  }
  ```

---

## 7. Run .NET 8 Project in Visual Studio as WSL
Now that you�ve set up WSL and SQLite, you can run your .NET 8 project in Visual Studio using WSL.

1. **Set WSL as the Target Environment:**
- In Visual Studio, go to **Tools** > **Options** > **Cross Platform** > **Linux**.
- Set **Ubuntu (WSL)** as the target environment.

2. **Run the Project:**
- Build and run your project in Visual Studio. It should now run inside Ubuntu under WSL.

---

## 8. Apply EF Core Migrations to Create MockDB
To set up the MockDB for the first time, you need to apply the Entity Framework Core migrations.

1. **Open the Terminal in Visual Studio:**
- Go to **Tools** > **NuGet Package Manager** > **Package Manager Console**.

2. **Add a Migration:**
- Run the following command to add a migration:
  ```bash
  dotnet ef migrations add InitialCreate
  ```

3. **Update the Database:**
- Apply the migration by running:
  ```bash
  dotnet ef database update
  ```
- This will create your SQLite database (`MockDB`) in the project directory.

---

This guide should help you set up your development environment for running .NET 8 projects using WSL and SQLite. Enjoy coding!
