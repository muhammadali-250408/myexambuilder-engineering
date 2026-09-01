# MyExamBuilder
> An automated system to generate specification aligned A Level Mathematics practice papers

## Overview
The system involves a structured question bank alongside exam analysis and algorithmic generation to create new papers whilst maintaining the structure and characteristics of real papers

The project involves more than simply selecting questions at random the generation process considers many factors:
- Total marks
- Difficulty level
- Topics
- Difficulty / mark progression
- Paper variation

This resporitry documents my engineering process behind MyExamBuilder, including the analysis, design decisions, algorithms and testing used during development.

The majority of the source code is intentionally kept private

--- 

## The Problem
During A Level preparation many students exhaust the limited supply of official exam papers (to date only 16 official past papers are available with many used in school mock exams) months before their exam season leading to many students feeling under prepared when it comes to the real thing or having to redo exams they have already done. 
MyExamBuilder aims to solve this problem by allowing students to create as many practice exams as they want from a large question bank.

## Features
- Paper splitting pipeline: Used to automatically analyse a directory of exam paper pdfs into individual questions, where each question is cropped to only the size it occupies in the paper and each question given a UID in the form (MARKS_YEAR_QUESTIONNUMBER) to allow for paper generation to occur later
- Question template pipeline: Automatically adds each question to a question page template of A4 size and saves it using the same UID in order to allow the question pages to be added to the paper with no more modification needed
- Paper generation: Uses an array of dictionaries to represent each page of the paper, including type of page and any other information like UID and number of marks if needed. Uses a difficulty curve equation to make questions become progressively more difficult similar to how the real exams structure them.

## Architecture
<img width="515" height="250" alt="image" src="https://github.com/user-attachments/assets/b950f26b-fbfa-46ea-acb0-771992b8b956" />
This is an image giving a rough overview of the data pipeline and overall structure of the project, each section is gone into more detail in their individual sections in the repository

## Status & Roadmap
- [X] Data preparation
- [X] Data analysis
- [ ] Paper Generation (Current phase)
- [ ] REST API Integration
- [ ] Web Integration
- [ ] Testing

Currently I am in the process of designing a more suitable paper generation algorithim which is intended to make the papers generate with a more realistic difficulty progression and to improve the visual presentation of the paper.

After this the paper generation logic will be tested and the test outputs put on this repo I will move onto designing how the REST API implementation will work in order to ensure that papers can be generated but by only authorised users.



