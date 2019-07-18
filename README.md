# PDF Tokenizer
###Uses Python to read a PDF file and produce a list of the top 30 words

The Natural Language Toolkit (NLTK) is a package used for building Python programs 
that work with human language data for statistical natural language processing
    ```python
    import nltk
    ```

The PyPDF2 package is a Python package that you can use for splitting, 
merging, cropping and transforming pages in your PDFs; including parsing text
    import PyPDF2

The textract package is a text parser for different file types.
    import textract

The Regular Expression (re) package helps with easy operations, in this case stip numbers 
    import re

Pandas is a popular package used for data science used to build dataframes
    import pandas as pd

Tokenize is similar to split. It splits files into individual words.
    from nltk.tokenize import word_tokenize

The nltk has a corpus with the most commonly used words. 
    from nltk.corpus import stopwords

Replace the file name with the PDF filename.
    filename = ''

Use a try/except block to attempt to open the file. Catch any exceptions
Open will read the file and create an object for processing
    try: 
         pdfFileObj = open(filename,'rb')

    except:
         print("File " + filename + " cannot be found")

The pdfReader variable takes the object and attempts to read it
     pdfReader = PyPDF2.PdfFileReader(pdfFileObj)

discerning the number of pages will allow us to parse through all #the pages
    num_pages = pdfReader.numPages
    count = 0
    text = ""

The while loop will read each page
    while count < num_pages:
        pageObj = pdfReader.getPage(count)
        count +=1
        text += pageObj.extractText()

This if statement exists to check if the above library returned words. It's done because PyPDF2 cannot read scanned files. 
If the above returns as False, we run the OCR library textract to convert scanned/image based PDF files into text.
    if text != "":
       text = text
    else:
        text = textract.process(pdfFileObj, method='tesseract', language='eng')

###Now we have a text variable which contains all the text derived #from our PDF file. We will clean our text variable, and return it as a list of top 30 keywords.

Strip out all the numbers
    text = re.sub(r'\d+', '', text)

Convert all letters to lower case
    for f in re.findall("([A-Z]+)", text):
        text = text.replace(f, f.lower())

The word_tokenize() function will break our text phrases into #individual words
    tokens = word_tokenize(text)

Remove all punctuation
    punctuations = ['(',')',';',':','[',']',',', '.', '|', '!', '?', '’', '‘', '“', '%', '#']

Use NLTK to remove the most common stopwords
    stop_words = stopwords.words('english')

Add custom stop words
    custom_stopwords = []

Append the list of stop words with the custom list
    stop_words.extend(custom_stopwords)

Create a list of words not in stop_words and not in punctuations.
    keywords = [word for word in tokens if not word in stop_words and not word in punctuations]

###Next we will take our list of keywords to produce the top 30 words
Instantiate a dictionary variable
    dict = {}

Start a for loop to count each word
    for words in keywords:
        if words in dict:
           dict[words] += 1
        else:
           dict[words] = 1

Sum up the total word count
    num_words = sum(dict[words] for words in dict)

Create a list of tuples with the word count and the word
    lst = [(dict[words], words) for words in dict]

    lst.sort()          #Sort the list according to word count
    lst.reverse()       #Reverse the order to show the largest count first

    print('\n The 30 most frequent words are \n')
    i = 1               #Instantiate a counter variable

    top_30 = []         #Instantiate a top 30 list variable

Use a for loop to print the top 30 words, append the list with the count and the word
    for count, keywords in lst[:30]:
        print('%2s.  %4s %s' % (i, count, keywords))
        top_30.append([count, keywords])
        i += 1

Use Pandas to create a dataframe
    top_30_df = pd.DataFrame(top_30)

Rename the columns
    top_30_df.columns = ['Count', 'Word']

Print out a csv with the list of top 30 words
    top_30_df.to_csv('output_2017.csv', index=False)
