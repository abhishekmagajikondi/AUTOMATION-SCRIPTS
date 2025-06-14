# AUTOMATION-SCRIPTS
import re

# Read the contents of the input file
with open("task.txt", "r") as file:
    content = file.readlines()

# Step 1 & 2: Remove lines starting with <Package or <Module
content = [line for line in content if not line.strip().startswith("<Package") and not line.strip().startswith("<Module")]

# Join lines for regex operations
text = ''.join(content)


# Step 3: Remove parameterized test cases using regex
text = re.sub(r'<[A-Za-z]+.*r.*\[', '', text)

# Step 4: Remove single test cases using regex
text = re.sub(r'<[A-Za-z]+.*([A-Za-z]+(_[A-Za-z]+)+).*_', '', text)

# Step 5: Remove "]>"
text = text.replace(']>', '')

# Step 6: Remove ">"
text = text.replace('>', '')

# Step 7: Remove empty lines
text = re.sub(r'^\s*$\n', '', text, flags=re.MULTILINE)

# Step 8: Align all lines to the left (strip leading/trailing spaces)
lines = [line.strip() for line in text.splitlines()]

# Final text
final_text = '\n'.join(lines)

# Write to output file
with open("cleaned_task.txt", "w") as out_file:
    out_file.write(final_text)

print("✅ File cleaned and saved as 'cleaned_task.txt'")















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
