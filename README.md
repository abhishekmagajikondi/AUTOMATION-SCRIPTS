import os
import pandas as pd

# === Step 1: Load the Excel file ===
input_file = "your_excel_file.xlsx"  # replace with your file name
df = pd.read_excel(input_file)

# === Step 2: Select relevant columns (indexing from 0) ===
case_id_col = df.columns[4]
section_col = df.columns[6]
df_filtered = df[[case_id_col, section_col]].dropna()

# === Step 3: Create output directory ===
output_folder = "Fetched Cases"
os.makedirs(output_folder, exist_ok=True)

# === Step 4: Prepare data for the pivot Excel and TXT files ===
summary_data = []

for section, group in df_filtered.groupby(section_col):
    case_ids = group[case_id_col].astype(str).unique()
    case_string = " or ".join(case_ids)

    txt_filename = f"{section}.txt"
    txt_path = os.path.join(output_folder, txt_filename)

    # Write TXT file
    with open(txt_path, "w") as f:
        f.write(case_string)

    # Append to summary
    summary_data.append([section, len(case_ids), txt_filename])

# === Step 5: Create Pivot Excel File ===
pivot_df = pd.DataFrame(summary_data, columns=["Section", "Case Count", "TXT File Name"])
pivot_excel_path = os.path.join(output_folder, "pivot_chart.xlsx")
pivot_df.to_excel(pivot_excel_path, index=False)

print("✅ Files created successfully in 'Fetched Cases' folder.")
