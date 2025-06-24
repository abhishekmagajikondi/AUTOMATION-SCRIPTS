import requests
from bs4 import BeautifulSoup
import pandas as pd

# STEP 1: Replace this with your actual HTML report URL
url = "http://your-internal-server.com/report.html"

# STEP 2: Download HTML content
response = requests.get(url)
response.raise_for_status()  # Throw error if fetch fails

soup = BeautifulSoup(response.content, "html.parser")
data = []

# STEP 3: Parse the report using your defined hierarchy
for file_block in soup.find_all("details", class_="file"):
    content_boxes = file_block.find_all("div", class_="content box")

    for box in content_boxes:
        for test in box.find_all("details", class_="test error"):
            summary = test.find("summary")
            if not summary:
                continue

            # Extract status
            status_tag = summary.find("span", class_="status")
            status = status_tag.get_text(strip=True) if status_tag else "UNKNOWN"

            # Extract script name
            funcname_tag = summary.find("span", class_="funcname")
            funcname = funcname_tag.get_text(strip=True) if funcname_tag else "UNKNOWN"

            # Extract optional case ID
            params_tag = summary.find("span", class_="params")
            case_id = params_tag.get_text(strip=True).strip("[]") if params_tag else None

            # Set Column 0
            identifier = case_id if case_id else funcname

            # Extract all <pre> messages inside <div class="repr">
            repr_block = test.find("div", class_="repr")
            logs = []
            if repr_block:
                for pre in repr_block.find_all("pre"):
                    logs.append(pre.get_text(strip=True))
            reason = "\n".join(logs) if logs else "No logs found"

            # Save the row
            data.append([identifier, status, reason])

# STEP 4: Write to Excel
df = pd.DataFrame(data, columns=["ID or Script Name", "Status", "Reason"])
df.to_excel("test_report_summary.xlsx", index=False)

print("✅ Excel 'test_report_summary.xlsx' created successfully.")
