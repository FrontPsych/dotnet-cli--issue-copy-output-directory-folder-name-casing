# Problem
`dotnet build` fails on Linux distributions when a project has dependencies using `CopyToOutputDirectory` with same paths but different casing. 

Say we have a project with dependencies A and B. Each of those dependencies has set their image files to be copied to the output directory using:
`
<CopyToOutputDirectory>Always</CopyToOutputDirectory>
`
- Solution.sln
  - Dependency_A.csproj
    - **I**mages/image_A.png
  - Dependency_B.csproj
    - **i**mages/image_B.png
  - Project.csproj
    - Dependencies/Projects
      - Dependency_A
      - Dependency_B

After running `dotnet build`, the build process manages to copy over only one dependency's image files but gets stuck trying to copy the other dependency's image files and eventually terminates in an error,

### Actual outcome
The build fails.

Project.csproj build output directory:
- ... 
- Images/image_A.png
- ...

Error message
```bash
/usr/share/dotnet/sdk/3.1.100/Microsoft.Common.CurrentVersion.targets(4601,5): error MSB3027: Could not copy "/home/user/repos/folder/Dependency_B/images/image_B.png" to "bin/Debug/netcoreapp3.1/images/image_B.png". Exceeded retry count of 10. Failed.  [/home/user/repos/folder/Project/Project.csproj]
/usr/share/dotnet/sdk/3.1.100/Microsoft.Common.CurrentVersion.targets(4601,5): error MSB3021: Unable to copy file "/home/user/repos/folder/Dependency_B/images/image_B.png" to "bin/Debug/netcoreapp3.1/images/image_B.png". Could not find a part of the path 'home/user/repos/folder/Project/bin/Debug/netcoreapp3.1/images/image_B.png'. [/home/user/repos/folder/Project/Project.csproj]
```
### Desired outcome
Build succeeds.

Project.csproj build output directory:
- ... 
- Images/image_A.png
- images/image_B.png
- ...

# Setup

- Linux distribution: **Ubuntu 19.10**
- .NET Core SDK: **3.1**
- ASP.NET Core runtime: **3.1**
- .NET Core runtime: **3.1**

# Reproduce steps
Prerequisites:
- Use a Linux distribution.
- Have .NET Core 3.1 installed.

Steps:
1. Clone the repository 
   ```bash 
   git clone https://github.com/FrontPsych/dotnet-cli--issue-copy-output-directory-folder-name-casing.git
   ```
2. Navigate to the root folder `IssueReport/`.
3. Build the solution.
   ```bash 
   dotnet build IssueReport.sln
   ```
4. Done, the build will finish with 10 warnings and 2 errors.

# Interesting caveats
- The build can sometimes(1/20) be successful on the first try. In order to reproduce the error clear the build output directory and retry.

