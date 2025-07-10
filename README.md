import pandas as pd

# Load Excel file
df = pd.read_excel("example.xlsx")  # Replace with your file name

# Find all SYSTEM_OUTPUT rows (rows with unique strings in Expected Value)
system_output_rows = []
for idx, value in enumerate(df.iloc[:, 4]):  # Column [4] = Expected Value
    if isinstance(value, str):
        system_output_rows.append(idx)

print(f"Found SYSTEM_OUTPUT rows at: {system_output_rows}")

# Prepare output blocks
output_blocks = []

for i, current_row in enumerate(system_output_rows):
    system_output_key = df.iloc[current_row, 4]

    # MAGIC_HEADER from same row, column [5]
    magic_header = df.iloc[current_row, 5]

    # LOG_PAGE_OFFSET_LOWER from same row, column [2]
    log_page_offset_lower = int(df.iloc[current_row, 2])

    # RECORD_SIZE, NUMBER_OF_RECORD, OFFSET, RESERVE_BYTES
    record_size = int(df.iloc[current_row + 1, 4])
    number_of_record = int(df.iloc[current_row + 2, 4])
    offset = int(df.iloc[current_row + 3, 4])
    reserve_bytes = int(df.iloc[current_row + 4, 4])

    # Find next SYSTEM_OUTPUT or end of file
    if i + 1 < len(system_output_rows):
        next_row = system_output_rows[i + 1]
    else:
        next_row = len(df)

    # NUMBER_DWORD:
    # Include current SYSTEM_OUTPUT row, stop before next SYSTEM_OUTPUT
    total_bytes_sum = df.iloc[current_row:next_row, 3].sum()
    number_dword = int(total_bytes_sum / 4) - 1

    # Create block
    block = f'''
"{system_output_key}" => {{
    "MAGIC_HEADER" => "{magic_header}",
    "RECORD_SIZE" => {record_size},
    "NUMBER_OF_RECORD" => {number_of_record},
    "OFFSET" => {offset},
    "NUMBER_DWORD" => {number_dword},
    "LOG_PAGE_OFFSET_LOWER" => {log_page_offset_lower},
    "RESERVE_BYTES" => {reserve_bytes},
}},
'''
    output_blocks.append(block)

# Save to telemetry_pearl.pm
with open("telemetry_pearl.pm", "w") as f:
    for block in output_blocks:
        f.write(block)

print("✅ Done! File saved as telemetry_pearl.pm")
