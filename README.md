# Form-16-PDF-DataExtract

This repository contains a Python script to extract tax-related data from Form-16 PDF files using pdfplumber and regular expressions.


Features-
-Extracts textual data and tables from Form-16 PDFs.
-Parses specific tax-related details like gross salary, house rent allowance, and deductions under various sections.
-Outputs the extracted data in JSON format.


Requirements-
Python 3.x
pdfplumber
re (Regular Expressions)
json


Installation
Clone the repository:
git clone https://github.com/Anshuman7t/form16-data-extraction.git
Navigate to the repository directory:
cd form16-data-extraction


Install the required packages:
pip install pdfplumber

Place your Form-16 PDF file in a suitable directory and update the file path in the script:
user_file = r"C:\path\to\your\sampleform16.pdf"

Run the script:
python extract_form16_data.py
The script will output the extracted tax details in JSON format.



**Code Explanation**

Import Libraries
python
import pdfplumber
import re
import json

Define the File Path
Update this variable with the path to your Form-16 PDF file.

user_file = r"C:\m\m\sampleform16.pdf"
Function to Extract Data from Form-16
The extract_form16_data function opens the PDF file, extracts text and tables from each page, and returns them.

def extract_form16_data(user_file):
    text = ""
    tables = []
    with pdfplumber.open(user_file) as pdf:
        for page in pdf.pages:  
            text += page.extract_text()
            tables.extend(page.extract_tables())
    return text, tables

Function to Extract Tax Details
The extract_tax_details function uses regular expressions to find specific patterns in the extracted text and tables to obtain tax details.

def extract_tax_details(pdf_text, pdf_tables):
    def find_value(pattern, default="0"):
        match = re.search(pattern, pdf_text)
        return match.group(1).replace(',', '') if match else default

    tax_details = {
        "assessment_year": find_value(r'Assessment Year\s+(\d{4}-\d{2})', "N/A"),
        "gross_salary": "0",
        "hra": "0",
        "travel_allowance": "0",
        "gratuity": "0",
        "leave_encashment": "0",
        "standard_deduction": 50000.0 if re.search(r'Standard Deduction\s+Yes', pdf_text) else 0.0,
        "professional_tax": "0",
        "other_income": "0",
        "section_80C": "0",
        "section_80CCC": "0",
        "section_80D": "0",
        "section_80E": "0",
        "section_80G": "0",
        "section_80TTA": "0",
        "rebate_87A": "0",
        "additional_cess_info": "0"
    }

    table_readable_values = {
        "gross_salary_readable": "Gross Salary",
        "hra_readable": "House rent allowance under section 10(13A)",
        "travel_allowance_readable": "Travel concession or assistance under section 10(5)",
        "gratuity_readable": "Death-cum-retirement gratuity under section 10(10)",
        "leave_encashment_readable": "Cash equivalent of leave salary encashment under section 10\n(10AA)",
        "professional_tax_readable": "Tax on employment under section 16(iii)",
        "other_income_readable": "Total amount of other income reported by the employee\n[7(a)+7(b)]",
        "section_80C_readable": "Deduction in respect of life insurance premia, contributions to\nprovident fund etc. under section 80C",
        "section_80CCC_readable": "Deduction in respect of contribution to certain pension funds\nunder section 80CCC",
        "section_80D_readable": "Deduction in respect of health insurance premia under section\n80D",
        "section_80E_readable": "Deduction in respect of interest on loan taken for higher\neducation under section 80E",
        "section_80G_readable": "Total Deduction in respect of donations to certain funds,\ncharitable institutions, etc. under section 80G",
        "section_80TTA_readable": "Deduction in respect of interest on deposits in savings account\nunder section 80TTA",
        "rebate_87A_readable": "Rebate under section 87A, if applicable",
        "additional_cess_info_readable": "Health and education cess"
    }

    gross_salary_section = False

    for table in pdf_tables:
        for row in table:
            if table_readable_values["gross_salary_readable"] in row:
                gross_salary_section = True
            if gross_salary_section and "Total" in row:
                tax_details["gross_salary"] = row[row.index('Total') + 2]
                gross_salary_section = False
            if table_readable_values["hra_readable"] in row:
                tax_details["hra"] = row[row.index(table_readable_values["hra_readable"]) + 1].replace(',', '')
            if table_readable_values["travel_allowance_readable"] in row:
                tax_details["travel_allowance"] = row[row.index(table_readable_values["travel_allowance_readable"]) + 1].replace(',', '')
            if table_readable_values["gratuity_readable"] in row:
                tax_details["gratuity"] = row[row.index(table_readable_values["gratuity_readable"]) + 1].replace(',', '')
            if table_readable_values["leave_encashment_readable"] in row:
                tax_details["leave_encashment"] = row[row.index(table_readable_values["leave_encashment_readable"]) + 1].replace(',', '')
            if table_readable_values["professional_tax_readable"] in row:
                tax_details["professional_tax"] = row[row.index(table_readable_values["professional_tax_readable"]) + 1].replace(',', '')
            if table_readable_values["other_income_readable"] in row:
                tax_details["other_income"] = row[row.index(table_readable_values["other_income_readable"]) + 2].replace(',', '')
            if table_readable_values["section_80C_readable"] in row:
                tax_details["section_80C"] = row[row.index(table_readable_values["section_80C_readable"]) + 2].replace(',', '')
            if table_readable_values["section_80CCC_readable"] in row:
                tax_details["section_80CCC"] = row[row.index(table_readable_values["section_80CCC_readable"]) + 2].replace(',', '')
            if table_readable_values["section_80D_readable"] in row:
                tax_details["section_80D"] = row[row.index(table_readable_values["section_80D_readable"]) + 8].replace(',', '')
            if table_readable_values["section_80E_readable"] in row:
                tax_details["section_80E"] = row[row.index(table_readable_values["section_80E_readable"]) + 8].replace(',', '')
            if table_readable_values["section_80G_readable"] in row:
                tax_details["section_80G"] = row[row.index(table_readable_values["section_80G_readable"]) + 9].replace(',', '')
            if table_readable_values["section_80TTA_readable"] in row:
                tax_details["section_80TTA"] = row[row.index(table_readable_values["section_80TTA_readable"]) + 9].replace(',', '')
            if table_readable_values["rebate_87A_readable"] in row:
                tax_details["rebate_87A"] = row[row.index(table_readable_values["rebate_87A_readable"]) + 5].replace(',', '')
            if table_readable_values["additional_cess_info_readable"] in row:
                tax_details["additional_cess_info"] = row[row.index(table_readable_values["additional_cess_info_readable"]) + 5].replace(',', '')

    return tax_details
Extract and Print Tax Details
Finally, the script extracts the text and tables from the PDF file and prints the tax details in JSON format.

pdf_text, pdf_tables = extract_form16_data(user_file)
tax_details = extract_tax_details(pdf_text, pdf_tables)
print(json.dumps(tax_details, indent=4))



**Contributing**

- **Improving Regular Expressions**: Enhance the accuracy and coverage of the regular expressions used for extracting tax details.
- **Handling Edge Cases**: Identify and handle various edge cases where the PDF structure may vary.
- **Extending Tax Details Extraction**: Add support for extracting additional tax details or other relevant information from Form 16.
- **Performance Optimization**: Improve the performance of the script, especially for large or complex PDF files.
- **Documentation**: Improve or expand the documentation, including usage examples, edge cases, and detailed explanations.
- **Unit Tests**: Add unit tests to ensure the script works correctly under various conditions.
- **GUI Development**: Develop a simple graphical user interface for users who are not comfortable with running scripts from the command line.

Feel free to fork this repository, make your changes, and submit a pull request. We welcome contributions to improve this script.

License
This project is licensed under the MIT License - see the LICENSE file for details.
