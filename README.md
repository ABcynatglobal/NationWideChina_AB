# Car Owners CSV Processing Script

![AI image](scrapescrape.jpeg)
---

## Overview

This script processes a CSV file of car owners, primarily from China. It handles tasks such as translating column headers, merging address fields, and cleaning the data by removing invalid email addresses and duplicate records. The invalid/duplicate records are saved in a separate "garbage" CSV file, while the cleaned data is saved for further use.

---

## Requirements

Before running the script, ensure the following libraries are installed:

- `pandas`: For reading, processing, and saving CSV data.
- `deep_translator`: To translate column headers from Chinese to English.
- `re`: To validate email addresses using regular expressions.
- `warnings`: To suppress specific warnings during runtime.

You can install the necessary libraries using:

<!-- python code block -->
```python
pip install pandas deep_translator
```

---

## Script Overview

This script consists of several key steps:
1. **Loading and translating the data**.
2. **Merging province, city, and address fields** into a single column called \`Full Address\`.
3. **Cleaning the data**: removing invalid or malformed email addresses, identifying duplicate rows, and managing missing data.
4. **Handling garbage data**: storing invalid and duplicate entries in a separate CSV file for further inspection.
5. **Saving the cleaned and processed data** into a new CSV file.

---

## Detailed Breakdown of the Code

### Step 1: Loading and Translating CSV Data

<!-- python code block -->
```python
df = pd.read_csv(input_file, low_memory=False)
translator = GoogleTranslator(source='auto', target='en')
translated_columns = [translator.translate(col) for col in df.columns]
df.columns = translated_columns
```

- **Loading the CSV**: The CSV file is loaded into a pandas DataFrame using \`pd.read_csv()\`. The \`low_memory=False\` argument ensures efficient loading of large files.
- **Translating the Column Headers**: Using the \`GoogleTranslator\` from \`deep_translator\`, the script automatically detects the language of the column headers and translates them to English.

---

### Step 2: Merging Address Fields

<!-- python code block -->
```python
def merge_address_columns(df):
    df['Full Address'] = df['Province'].fillna('') + ', ' + df['City'].fillna('') + ', ' + df['address'].fillna('') + ', ' + df['post code'].fillna('')
    df['Full Address'] = df['Full Address'].str.replace(r'(^, |, $)', '', regex=True)
    garbage_df = df[['Province', 'City', 'address', 'post code', 'Monthly salary', 'marriage', 'educate', 'color', 'gender', 'Birthday', 'industry', 'Unnamed: 21']].copy()
    df.drop(columns=['Province', 'City', 'address', 'post code', 'Monthly salary', 'marriage', 'educate', 'color', 'gender', 'Birthday', 'industry', 'Unnamed: 21'], inplace=True)
    return df, garbage_df
```

- **Merging Address Fields**: The script combines the \`Province\`, \`City\`, \`address\`, and \`post code\` columns into a single \`Full Address\` field, which simplifies the dataset.
- **Dropping Unneeded Columns**: After creating \`Full Address\`, the original address-related fields, along with other unnecessary columns (such as \`Monthly salary\`, \`marriage\`, etc.), are dropped from the DataFrame and stored in a garbage file. 

---

### Step 3: Cleaning the Data

#### Email Validation:

<!-- python code block -->
```python
def is_valid_email(email):
    email_regex = r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$'
    return re.match(email_regex, email) is not None and bool(re.search(r'[a-zA-Z0-9]', email.split('@')[0]))
```

- **Validation of Email Format**: This function uses a regular expression to ensure that the email addresses have a valid format. Additionally, it checks that the local part (before the \`@\`) contains alphanumeric characters. 

#### Handling Invalid and Duplicate Data:

<!-- python code block -->
```python
invalid_email_rows = df[~df['Email'].apply(is_valid_email)]
df = df.drop(invalid_email_rows.index)

duplicates = df[df.duplicated(subset=['Email', 'ID Number', 'Frame number'], keep='first')]
df = df.drop(duplicates.index)
```

- **Invalid Email Handling**: Rows with invalid email addresses are identified and moved to the "garbage" DataFrame, while being dropped from the original dataset.
- **Duplicate Record Handling**: Rows that contain duplicate values in the \`Email\`, \`ID Number\`, and \`Frame number\` columns are also moved to the garbage file and removed from the main dataset.

---

### Step 4: Handling Invalid/Removed Data (Garbage)

<!-- python code block -->
```python
garbage_df.to_csv(garbage_file, index=False)
```

- **Saving Garbage Data**: The rows flagged as either invalid or duplicates are saved to a separate file for inspection. This allows users to review and possibly rectify data issues.

---

### Step 5: Saving the Cleaned Data

<!-- python code block -->
```python
df.to_csv(cleaned_file, index=False)
```

- **Saving the Cleaned Data**: The final cleaned DataFrame is saved to a new CSV file. This file is free from invalid email addresses and duplicate rows, making it ready for further processing or analysis.

---

## 5. Frequently Asked Questions (FAQs)

#### Q1: What happens if the CSV file is empty?
If the input CSV file is empty, the script will issue a warning and halt further processing. This prevents the creation of empty or invalid output files.

#### Q2: How are invalid email addresses identified?
The script uses a regular expression to validate the format of email addresses. If an email address doesn’t conform to this format or has alphanumeric characters before the \`@\`, it is considered invalid and moved to the garbage file.

#### Q3: How are duplicate rows detected?
Duplicate rows are detected based on the \`Email\`, \`ID Number\`, and \`Frame number\` columns. If more than one row contains the same values in these fields, the subsequent duplicates are moved to the garbage file.

#### Q4: Can I modify the columns being checked for duplicates?
Yes, the \`duplicates = df[df.duplicated(subset=['Email', 'ID Number', 'Frame number'], keep='first')]\` line can be updated to check for duplicates based on different or additional columns.

#### Q5: How can I adjust the file paths?
You can change the input, cleaned, and garbage file paths ("r"path_to_your_files") in the following lines of the script:

<!-- python code block -->
```python
input_file = r"path_to_your_input_file.csv"
garbage_file = r"path_to_garbage_file.csv"
cleaned_file = r"path_to_cleaned_file.csv"
```
All you have to do is replace the "r"path_to_your_files" with where your input file is, where you want the garbage file to go and where you want the cleaned file to go.

#### Q6: What happens to columns that are not used after the address merge?
These columns (\`Province\`, \`City\`, \`address\`, etc.) are dropped from the DataFrame after the \`Full Address\` is created, and the original columns are stored in the garbage DataFrame for reference.

---

