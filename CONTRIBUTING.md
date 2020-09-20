# Guide to Making Developer Contributions

## Overview:

A CSV with a list of package names and versions is uploaded by the user, then
their individual risks are assessed using `riskmetric`.

The `riskmetric` package is loaded in the `setup.R` file. Then, in the `./Modules/dbupload.R` file, the function `metric_mm_tm_Info_upload_to_DB` calls
`riskmetric` and saves the different metric results on the database (SQLite by
default).

The database is updated to include the user specified packages and their
associated metrics.

These metrics are used to populate the UI of the application. 
Each package only needs to be entered into the database once for the user
to see its associated risk.

## Package files and folder 

[TODO order in which to introduce these files, 
maybe also include a schematic of how the files are related]

### Root
- `setup.R` Load all libraries, install libraries that aren't included. Called in `app.R.`
- `app.R` This file contains the server and UI code for displaying the app, 
     calling the files within the Modules, Server, UI, and Utils folders.

### UI/Server

**Most files are in both the UI and Server folders with the same name. If files are only in one folder, the folder name is included in the extension below:**

- `UI/dashboard_screen.R` contains the UI layout of the landing page of the application

- `UI/login_screen.R` Input user name and role if not found in the database.

- `Server/sidebar.R` Based on selected package, allow the user to submit an overall comment,
   or overall risk decision. Comments and decisions are added to the table `Packageinfo`
     
- `assessment_critera.R` On the top left of the application the user can open a 
   help modal which describes the criteria used to assess the risk of the application.
   The text to populate this modal is located in the `Data` directory

- `uploadpackage.R` Import the packages from the user supplied CSV. 
   TODO I'm not exactly sure how this file "knows" about prior uploaded packages - ie
   How is `Total`, `New`, `Undescovered`, and `Duplicates` populated?
   I also see code where this function uses `db_fun` to grab the `Packageinfo` table
   but I don't really understand what is happening in this function...]
   The UI and server code for viewing and downloading sample data.

- `communityusage_metrics.R` Using the fields within the table `CommunityUsageMetrics`,
   create a `Highchart` graph of package downloads and versions over the lifetime of the           package. [TODO do we need to repeat the code to plot every month? Can this be cleaned up?].
   There is also an area to leave comments, which will be added to the`CommunityUsageMetrics` table. 
   
- `cum_report.R` [TODO Repeats the code of `communityusage_metrics.R`]

- `maintenance_metrics.R` Uses the table `MaintenanceMetrics` to create the 9 UI badges associated with the risk metric package, and creates an area for leaving comments,
which will be added to the `MaintenanceMetrics` table. 

- `mm_report.R` [TODO Repeats the code of `maintenance_metrics.R`]

- `testing_metrics.R` Uses the fields within the table `TestMetrics` to create a `amAngularGauge`. Also includes a comment section which will be added to the `TestMetrics` table.

- `tm_report.R` [TODO Repeats the code of `testing_metrics.R`]

- `reportpreview.R` Uses `cum_report.R`, `mm_report.R`, and `testing_metrics.R`
  to repeat the code seen in the other tabs and create a singular, downloadable report.
  [TODO honestly do we need this tab at all? Can we just have one big download button
  either in the sidebar or at the top that just prints the contents of the other tabs?]

### Modules

- `DB.R` open connections to each of the tables used within the application. All table names can be found in `Utils/SQLite.sql`
- `dbupload.R` [TODO Upload the existing packages from within the database and run riskmetric on any new packages added via CSV?]
- `file_upload_error_handling.R` How to handle corrupt CSV uploads

### Utils
  

- `utils.R` 
  - `db_fun` connects to database. This function is called in `app.R` 
     to populate the drop-down of packages either already included in the database
     or within the uploaded CSV of packages.
  - `TimeStamp` helper function to create a time-stamp. 
     Any time the user can write data to the database,
     this function is called to append the time to the end of the string.
  - `GetUserName` return computer username. Used on the initial screen 
     to get the users name and credentials from a database rather than manually entering

### Data

Text files containing the explanations of all metrics. Files are used to populate text within `assessment_critera.R`.

### www 
    
Non-R assets for the application: images and a JavaScript helper file to app reactivity.

### Files created by the app

- `loggit.json`: Log file created each time the application is ran. It contains information about the uploaded packages and application errors. For example, when the user uploads the example csv file `Upload_file_structure.csv` for the first time, a line similar to this one is added to the `loggit.json` file:

  ```
  {"timestamp": "2020-08-28T18:21:29-0400", "log_lvl": "INFO", "log_msg": "Summary of the uploaded file: Upload_file_structure.csv Total Packages: 3 New Packages: 0 Undiscovered Packages: 0 Duplicate Packages: 0"}
  ```

  The log functionality is handled by the `loggit` package. The following files write to `loggit.json`:

  - `/Modules/dbupload.R`: Logs whenever there is an error extracting or uploading info of a package.
  - `./Server/sidebar.R`: Logs whenever there the user makes a final decision on a package, i.e., when the user clicks 'Submit Decision'
  - `./Server/uploadpackage.R`: Logs whenever a csv file is uploaded. In particular, it logs a summary of the uploaded file.

- `database.sqlite`: SQLite database containing the risk, metrics, and comments of each package uploaded. It is created the first time the application is ran. Subsequent runs of the application will update the existing db.

   The db is created in the `Utils/utils.R` file. It contains the following tables:
   - `Packageinfo`
   - `MaintenanceMetrics`
   - `CommunityUsageMetrics`
   - `TestMetrics`
   - `Comments`

<center>**Last Updated on 09/2020**</center>