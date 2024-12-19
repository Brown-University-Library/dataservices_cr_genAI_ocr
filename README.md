# Historical trade data digitization with genAI/Claude API: A workflow + code + resources

## Introduction

This repository documents the workflow for data cleaning, genAI prompting, and data transformation to utilize a genAI API (specifically, the Claude API) to perform optical character recognition (OCR) digitization of historical trade data tables. 

For the primary research team, this will replicate work already performed over November and December 2024, and provide the template for future development and expansion of this project.

For future researchers, students, and colleagues, we hope this will provide a template for further work.

## Background: the challenge of digitizing historical trade data; 1960-1982 Costa Rica trade documents

For this project, our challenge is to digitize 20+ years of import and export data. Each year is collected into a single PDF file, with a range of about 300 pages of import data and 100 pages of export data. 

There are several data quality challenges for this task. Each page was generated via typewriting and features a broadly consistent format of one row per category per country, or one row per category when a country is not specified. There is a broad variance in the quality of the scan, legibility of the typeface, consistent formatting aids of the tables themselves (e.g. how many guiding lines are used), as well as few guiding lines per record. 

The researchers had previously attempted to use tools like ABBYY FineReader, built-in OCR tools in Adobe Acrobat, etc., but each of these performed poorly at recreating the tabular data structure of the underlying data. Thus, we turned to a mix of reproducible code (Python) and genAI (via the Claude API) in order to break this task into a sequence of tasks that could perform at a higher accuracy rate.

## The workflow: Overview

Here are the steps in summary:
1. **Gather tools and resources**: Generate API key; pay for tokens; install and gather tools
2. **Data collection and documentation**: Gather input PDFs from research/partner datasets and generate manifest file
3. **Image/PDF preprocessing**: Create regularized images for all import and export data.
4. **Prompt engineering for genAI**: Generate an effective prompt/messaging approach.
5. **Run bulk API queries**: Run all images through API; rerun in case of failures
6. **Export results**: Gather results into a single tabular output

Now we'll explore each at greater depth.

## Step one: Gather tools and resources

### Tools
* Python / Jupyter Notebooks / Anaconda
* Google Sheets
* Text editor (Visual Studio Code)

### Resources:
* Claude API key
* API tokens - note free + paid cost, tiers, etc. Note 

### Skills:
* Programming with Python
* Pandas
* Familiarity with querying APIs
* The Jupyter Notebook approach

## Step two: Data collection and documentation

Gather source documents - PDFs per year

Generate manifest file to identify target pages: [Manifest link](https://docs.google.com/spreadsheets/d/1CN9zL2GGu0YHP1xMiRY2r63xYN-PtCw-nkni8st9UWw/edit?usp=sharing)

## Step three: Image/PDF preprocessing

### Python process:
* Convert each PDF page to an image (grayscale image to [Claude maximum image dimensions](https://docs.anthropic.com/en/docs/build-with-claude/vision) + name with year + page_num
* Read in OCR manifest file
* Move import and export images to separate folders

## Step four: Prompt engineering for genAI

Below is the function used to generate a prompt for Claude API using [Vision prompting best practices](https://docs.anthropic.com/en/docs/build-with-claude/vision).

### Key components:

#### User message: 
` "text": "For the image included, please read this historical table and return a JSON list that contains one dictionary for every row of the table The output should include the following three keys: DESCRIPCION, PAIS, and VALORES and the values should come from every row in the original table. Every row in the original data should appear in the output JSON list. Here are the rows for each key. For the Descripcion value, please only include the string of numbers and spaces that appear in the DESCRIPCION table column - this value looks something like 051 07 02 00. Treat this data like a string and include leading zeros and spaces. Please note that every JSON dictionary object must include a DESCRIPCION column value. When reading the original historical table image, you must fill in the blank DESCRIPCION values for each row that contains only a PAIS value - these PAIS values are all sub-items of a main row and so you can fill in the DESCRIPCION value with the first value you see above those rows with only PAIS values. The PAIS value is either missing (which is ok) or the value in that column, like NICARAGUA, HONDURAS, etc. For VALORES, use the value in the " + value_type + " column."`

#### System message:
`"text": "For the image included, please read this historical table and return a JSON list that contains one dictionary for every row of the table The output should include the following three keys: DESCRIPCION, PAIS, and VALORES and the values should come from every row in the original table. Every row in the original data should appear in the output JSON list. Here are the rows for each key. For the Descripcion value, please only include the string of numbers and spaces that appear in the DESCRIPCION table column - this value looks something like 051 07 02 00. Treat this data like a string and include leading zeros and spaces. Please note that every JSON dictionary object must include a DESCRIPCION column value. When reading the original historical table image, you must fill in the blank DESCRIPCION values for each row that contains only a PAIS value - these PAIS values are all sub-items of a main row and so you can fill in the DESCRIPCION value with the first value you see above those rows with only PAIS values. The PAIS value is either missing (which is ok) or the value in that column, like NICARAGUA, HONDURAS, etc. For VALORES, use the value in the " + value_type + " column."`

##### Assistant message:
`text": "For the image included, please read this historical table and return a JSON list that contains one dictionary for every row of the table The output should include the following three keys: DESCRIPCION, PAIS, and VALORES and the values should come from every row in the original table. Every row in the original data should appear in the output JSON list. Here are the rows for each key. For the Descripcion value, please only include the string of numbers and spaces that appear in the DESCRIPCION table column - this value looks something like 051 07 02 00. Treat this data like a string and include leading zeros and spaces. Please note that every JSON dictionary object must include a DESCRIPCION column value. When reading the original historical table image, you must fill in the blank DESCRIPCION values for each row that contains only a PAIS value - these PAIS values are all sub-items of a main row and so you can fill in the DESCRIPCION value with the first value you see above those rows with only PAIS values. The PAIS value is either missing (which is ok) or the value in that column, like NICARAGUA, HONDURAS, etc. For VALORES, use the value in the " + value_type + " column."`

```python
# take in an image (e.g. from a PDF page); query claude API; return table
def create_dataframe_from_image(image_path, year = 1960):
    # read image
    with open(image_path, "rb") as f:
        image_data = f.read()
        base64_image = base64.b64encode(image_data).decode()

    if year < 1964:
        value_type = "PESOS"
    else:
        value_type = "DOLARES"
    messages = [
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "For the image included, please read this historical table and return a JSON list that contains one dictionary for every row of the table The output should include the following three keys: DESCRIPCION, PAIS, and VALORES and the values should come from every row in the original table. Every row in the original data should appear in the output JSON list. Here are the rows for each key. For the Descripcion value, please only include the string of numbers and spaces that appear in the DESCRIPCION table column - this value looks something like 051 07 02 00. Treat this data like a string and include leading zeros and spaces. Please note that every JSON dictionary object must include a DESCRIPCION column value. When reading the original historical table image, you must fill in the blank DESCRIPCION values for each row that contains only a PAIS value - these PAIS values are all sub-items of a main row and so you can fill in the DESCRIPCION value with the first value you see above those rows with only PAIS values. The PAIS value is either missing (which is ok) or the value in that column, like NICARAGUA, HONDURAS, etc. For VALORES, use the value in the " + value_type + " column."
                },
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/jpeg",
                        "data": base64_image
                    }
                }
            ]
        },
        {
            "role": "assistant",
            "content":'''[
    {
        "DESCRIPCION":''' }
    ]

    # send message

    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        messages=messages,
        system="You are a historian of economics data, looking to create detailed, error-free JSON list data based on scanned typerwriter tables. Please double-check that all values appear in the correct columns and rows, and that every individual digit has been digitized correctly. Include a complete list of everything you detected in the historical table in your response. Only include the JSON list in your result. Return the complete results, and make sure to properly close the JSON list in your response.",
        max_tokens=8192
    )

    # return message as dataframe
    # clean up this response to represent as a DataFrame
    
    # Extract the text content from the response
    text_content = response.content[0].text

    #append our JSON list assistant prompt
    
    text_content_final = '''[
    {
        "DESCRIPCION":''' + text_content

    return text_content_final
```

## Step five: Run bulk API queries

The bulk of the notebook code implements a workflow for this. More to add.

## Step six: Export results

Read and collate all output CSVs and output as single CSV(s)
