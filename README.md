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
    
    
    
### Important Note: Stratified Output

The "stratified output" section of the script relies on specific data files that are too large to host directly in this repository.

To successfully run this part of the script, you **must** download the following file:

* **[PICRUSt2 Output Data](https://drive.google.com/file/d/1IfQi9TMPXJ9QyKjmX4EOgzn8JLyFKWNz/view?usp=drive_link)**

After downloading, extract its contents and use them to **replace** the `picrust2_out_steady` and `picrust2_out_transplant` folders located within the `data/` directory of this repository.