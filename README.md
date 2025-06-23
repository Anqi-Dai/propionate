# How to Use This Repository

---

To get started with this project, follow these simple steps:

## 1. Download the Repository

* **Download ZIP:** On the main page of this repository, click the green **"Code"** button and select **"Download ZIP"**. This will give you a compressed file of the current project.


## 2. Recreate the R Environment

This project uses **renv** to manage its R package dependencies, ensuring a consistent environment for everyone.

* **Open the R Project:** Open the `.Rproj` file located in the downloaded repository. If you're using RStudio, it should prompt you to restore the `renv` environment automatically.
* **Restore Manually:** If you're not prompted or using a different R setup, open an R console within the project directory and run:
    ```R
    renv::restore()
    ```
    This command will read the `renv.lock` file and install all the necessary R packages required for the project.