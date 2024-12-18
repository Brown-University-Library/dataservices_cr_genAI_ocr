# Historical trade data digitization with genAI/Claude API: A workflow + code + resources

## Introduction

This repository documents the workflow for data cleaning, genAI prompting, and data transformation to utilize a genAI API (specifically, the Claude API) to perform optical character recognition (OCR) digitization of historical trade data tables. 

For the primary research team, this will replicate work already performed over November and December 2024, and provide the template for future development and expansion of this project.

For future researchers, students, and colleagues, we hope this will provide a template for further work.

## The challenge of digitizing historical trade data: 1960-1982 Costa Rica trade documents

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

## Step two: Data collection and documentation

## Step three: Image/PDF preprocessing

## Step four: Prompt engineering for genAI

## Step five: Run bulk API queries

## Step six: Export results
