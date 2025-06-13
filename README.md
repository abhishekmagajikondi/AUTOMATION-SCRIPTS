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
