# Data preperation

## Objective
For data preparation I was trying to understand how could I get past exam papers into individual questions in a specific form which will make not only identifying them easy but also make it so that they can easily be added to a pdf allowing for easy paper generation.
Additionally I also designed and built an automated pipeline to allow for much quicker adding of processed questions to the question bank

## Problem
The main reason for this is that if the questions were not extracted into a suitable, uniform format it would make the paper generation far more complex as there would be alot of different cases to consider, and it would also make adding questions to the question bank more difficult hence making it less maintainable and expandable

## Method

### Identifying question pages
The first step was finding a reliable way of finding the page numbers where the questions are written.
This was done by manually analysing A Level Edexcel Mathematics papers, and after a while a clear pattern was spotted:
- Every page 2 is a question page
- The page after the end of every question apart form the last one is a question page

This provided a useful starting point for automatically identifying likely question pages without having to inspect every page individually.

### Determining page boundaries
The next step was determining how these patterns could be detected automatically.

Rather than relying soley on fixed page numbers, I investigated the text contained within each page, and indentified phrases and formatting patterns that consistently appeared at certain points in the paper
For example every page where a question finished there was always text at the bottom saying either "Total for Question 1 is ..." or "Total ... marks", initially I thought they could be identified by just finiding total however given it is a math paper it would lead to alot of false positives. The route I eventually decided on was having 2 different phrases one being "Total for question" and the other being a regex for "Total ... marks" as these specific phrases are unique to that final page

This approach allowed question ends to be identified and hence subsiquently the start of all questions as the start of every question apart from the first is after a question end page.

### Extracting questions
<img width="306" height="427" alt="myexambuilderdiagram" src="https://github.com/user-attachments/assets/757199f8-57b1-4cdc-bb1c-ae9ffa2f103e" />

The image above shows what a question page looks like in a real paper. My goal was to only extract the green area (the area the question actually occupied on the page) so that I could put any question on my answer page template.

I approached this by splitting this area into 4 distinct bounds
- Top bound
- Bottom bound
- Left bound
- Right bound

And then for each I found ways to find them for all cases.
For the left and right bounds, they were the majoirty of the time a similar distance form the ends of the page, hence the approach I decided on was to get the page width and multiple that by a certain factor. Using the result as the left bound and taking it away from the width value for the right bound.

For the top bound, across questions, it occured in a similar position form the top of the page each time, hence I used a similar approach. I got the page height, multiplied it by a certain factor and used that as the top bound.

The bottom bound was the most difficult one, as it was variable for every question. After analysing the structure of the questions, I decided that a possible approach would be identifying where the first answering line is then a fixed % above that is the bottom bound. Since in every question they always provide answering space under the question. 
To find the specific ways to find it, I extracted the text of a question page to analyse the structure. (A snippet of that extraction is below).
It was clear that in the papers the character ```_``` is used to represent the lines on the page. Hence I thought to find the first instance of ```_``` on a page and find its Y value, and then reduce the Y value by a certain % of the height to get the bottom bound.

The reason I remove from the Y value rather than add to it despite needing the lower bound to be higher than the first line is that in PyMuPDF, the coordinate system starts from the top left rather than bottom left.

```
Given y = 3x
, express each of the following in terms of y. Write each expression in its
simplest form.
(a) 3
3x
(1)
(b) 1
3
x−2
(2)
(c) 81
9
23− x
(2)
___________________________________________________________________________
___________________________________________________________________________
___________________________________________________________________________
___________________________________________________________________________
___________________________________________________________________________
___________________________________________________________________________
```

### Creating unique identifiers for each.
I created a format of MARKS_YEAR_QUESTION NUMBER as the unique identifier for each extracted question since it means that once the mark schemes are also extracted using information soley in the mark scheme the same unique identifier can be derieved for the associated question, allowing question and mark scheme to be linked independantly of one another.

Methods used to find each variables:
- **Marks:** On all question the marks are always shown like this (5) (2) and so on for different parts of the question, hence I created a script which would scan through the extracted page text and look for something in the format of "(" + an string between 1 and 20 + ")", and if it found something matching this it would convert that string into an int and add it to a variable called total_marks until it scanned the entire page.
- **Year:** The most efficient method was that since the papers would have to be manually downloaded anyway, as they would need to be verified to be text based pdf rather than image based, and so their names are set to the actual year they are associated with and the python script uses os.listdir(path) to output all the paper names as a list and then list[i].split(".")[0] to get the name without the .pdf in it.
- **Question Number:** There is a variable called question number and when the script is looping through to find question pages when it fnds a question page it increments the question number variable by one and assigns it to that page.

## Implementation
I used the PyMuPDF library to process the pdfs as it had all the necessary functions prebuilt such as .search_for() to find specific strings and show_pdf_page() allowing for easy overlaying of pages and extracting pages and .insert_pdf() allowing for easy paper generation, and since it is based on a C engine is very fast aswell.

### Identifying question pages
```
def page_locater(pdf, look_up_phrase, is_regex):


    page_number = 1
    pages_with_phrase = []


    if is_regex:
        print("REGEX")
        for page in pdf:
            output = page.get_text()
            print(output)

            search_result = re.search(look_up_phrase, output)

            if search_result:
                pages_with_phrase.append(page_number)

            page_number += 1

        return pages_with_phrase


    else:

        for page in pdf:
            output = page.search_for(look_up_phrase)

            if len(output) != 0:
                pages_with_phrase.append(page_number)#

            page_number += 1
        return pages_with_phrase
```

### Determining page boundaries

Finding bottom bound

<img width="403" height="68" alt="image" src="https://github.com/user-attachments/assets/47bd19dc-a143-48d4-b6b7-71cd22414b69" />


Finding other bounds

<img width="470" height="64" alt="image" src="https://github.com/user-attachments/assets/79afc91b-4441-4be2-92f2-c4799193d7e0" />


