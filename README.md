# Spanish-Standardization.-Preprocessing-stage-in-text-mining

General process text mining has different steps: data acquisition, preprocessing, transformation, analysis and evaluation.
Regarding preprocessing step, it is need a first activity before spelling correction and stemming. This first activity is called standardization and cleaning. The main steps are:
  - Treatment of regular expression to unify the text. 
  - Elimination of verb endings in infinitives.
  - Elimination of puntuation marks, spaces, etc. 

This code to standardizate a text is implemented in Python.
The packages needed are:
````
#import packages 
import re
import spacy
nlp = spacy.load("es_core_news_sm") #use spanish language
````
PreprocessText is the main function. A specific Text is introduced to be preprocessed
````
#main function. Standardization of a Text

def PreprocessText(Text,UseHagstag,UseMention,UseWebSite):
    #delete abreviations and frecuent expression
    Text, hagstag, mention, webs= GetRegularExpression(Text)
    #delete endings verbs
    Text=DeleteEndings(Text)
    #delete wordstop 
    Text=DeleteStopWord(str(Text))
    #For a full preprocess text mining, you should consider:
    #Spelling correction
	#Use a specific function to correct
    #Stemming
	#Use a specific function to stem
    
    #Include hagstag/mention/Text if UseHagstag,UseMention,UseWebSite areTrue
    if UseHagstag==True:
        Text=Text+" "+hagstag
    if UseMention==True:
        Text=Text+" "+mention
    if UseWebSite==True:
        Text=Text+" "+webs

    return Text
````
functions needed in the main function
````
def GetRegularExpression(Text):
    """
    Save webs hagstags mentions
    delete web
    delete mentions (@)
    delete hagstag (#)
    delete numbers and simbols
    delete laughs jaja,jeje,etc
    delete abbreviations
    """
    FinalText=""
    CorrectText=[]
    CorrectText2=[]
    GetWebs=""
    GetHagstag=""
    GetMention=""
    TextAnalysis=Text.lower().split() #Text formato lista
    

    #look for webSite, hagstag and mentions
    for w in TextAnalysis:
        if re.search('www.+|http:+|https:+',w):
            GetWebs=w+" "
        elif re.search('#+',w):
             GetHagstag=w+" "
        elif re.search('@+',w):
            GetMention=w+" "
        else:
            CorrectText.append(w)
    
    #dete simbols/number/emoticons/etc
    for w in CorrectText:
        wc=(re.sub('[^a-zA-Záéíóúüñ]+',' ',w))
        CorrectText2.append(wc)
    #delete laughs
    for w in CorrectText2:
        if not re.search('jaj+|jj+jaj+|jj+|jej+|jij+|joj+|juj+|jsj+|lol|xd|quot',w):
            FinalText=FinalText+w+" "
    #delete abbreviations
    FinalText=GetTextNoAbbreviation(FinalText.lower().split())

    #return FinalText: text standarizated. Also Hagstag, Mention, Webs
    return FinalText, GetHagstag, GetMention, GetWebs

````
````
#Transform an abbreviated word into its corresponding correct form
def GetCorrectWordAbbreviation(word):
    original=word
    switcher={
            "q":'que',
            "qe":'que',
            "k":'que',
            "ke":'que',
            "xk":'porque',
            "xke":'porque',
            "pq":'porque',
            "x":'por',
            'muxo':'mucho',
            'tk':'te quiero',
            'tq':'te quiero',
            'tkm':'te quiero',
            'tqm':'te quiero mucho',
            'ads':'adiós',
            'tb':'también',
            'vdd':'verdad',
            'xa':'para',
            'xo':'pero',
            'dnd':'donde' ,  
            'mñn':'mañana',
            'tp':'tampoco',
            'kieres':'quieres',
            'kerer':'querer',
            'kiero':'quiero',
            'kieren':'quieren',
            'pa':'para',
            'd':'de',
            'm':'me',
            't':'te',
            'aki':'aquí',
            'min':'minuto',
            'sto':'esto',
            'sta':'esta',
            'quero':'quiero',
            'kero':'quiero',
            'salu2':'saludos'
            }
    return switcher.get(word,original)
````
````
#Transform an abbreviated words in a text into its corresponding correct form
def GetTextNoAbbreviation(Text):
    TextSinAbreviacion=""
    for i in range(len(Text)):
        TextSinAbreviacion=TextSinAbreviacion+GetCorrectWordAbbreviation(Text[i])+" "
    return TextSinAbreviacion
````
````
#delete endings in infinitive verbs in a Text
def DeleteEndings(Text):
    FinalText=[]
    doc = nlp (Text.lower())
    for token in doc:
        if (token.pos_=='VERB'):
            FinalText.append(DeleteCDVerbo(token.text))
        else:
            FinalText.append(token.text)
    return FinalText
````
````
#delete endings in an infinitive verb word
def DeleteCDVerbo(word):
    FinalText=""
    if len(word)>3:
	#infinitive + nos las los les
        if re.search('[aei]rl[aeo]s$|[aei]rnos$', word):
            FinalText=word[0:len(word)-3]+" " #Get just infinitive form
        #infinitive + me te le lo la os se
        elif re.search('[aei]rl[aeo]$|[aei]ros$|[aei]r[mts]e$', word):
            FinalText=word[0:len(word)-2]+" " #Get just infinitive form
        #infinitive + -selo -sela -sele
        elif  re.search('[aei]rsel[aeo]$', word):
            FinalText=FinalText+word[0:len(word)-4]+" " #Get just infinitive form
        #it is normal
        else:
            FinalText=word #save original word
    return FinalText
````
````
#Delete StopWords in a Text
def DeleteStopWord(Text):
    FinalText=[]
    doc = nlp (Text)
    for token in doc: 
        if token.pos_ != 'PUNCT' and token.pos_ != 'SPACE' and token.text !='\n' and token.text !='\n\t' and token.text !='\t' and token.is_stop==False:
            if not len(token.text)==1: # if it a single letter, don't save it
                FinalText.append(token.text)
    return FinalText
````
