# tweet-analysis

install.packages('twitteR')         # Used to connect twitter with R
install.packages('RCurl')           # Provides functions to allow one to compose general HTTP requests
                                    # and provides convenient functions to fetch URIs, get & post forms, etc. 
                                    # and process the results returned by the Web server.
install.packages("tm")              # Text Mining Package
install.packages("wordcloud")       # Visualization using word cloud
install.packages("SnowballC")       # for text stemming (stemming is the process of reducing inflected (or sometimes derived) words to their word stem)

library(twitteR)
library(RCurl)
library(tm)
library(wordcloud)
library(SnowballC)

api_key = 
api_secret = 
access_token = 
access_token_secret = 


setup_twitter_oauth(api_key,api_secret,access_token,access_token_secret)  # handshake functions 
                                                                          # from the httr package for a twitteR session


MyTweets <- searchTwitter('roadster',n=200,lang = "en")                   # Extracting tweets

Tweet_Data_Frame <- do.call("rbind", lapply(MyTweets, as.data.frame))     # Binds all tweets one below the other

Tweet_Data_Frame$text <- sapply(Tweet_Data_Frame$text,function(row) iconv(row, "latin1", "ASCII", sub=""))  # Replacing ASCII values to blank

Tweets <- Tweet_Data_Frame$text     

corpus <- Corpus(VectorSource(Tweets))     # Corpus is a list of a document. VectorSource is for only character vectors
corpus[[18]][1]                            # Vieving the corpus

#Text transformation #Transformation is performed using tm_map() function

corpus = tm_map(corpus, content_transformer(tolower))              # converting to lower
corpus = tm_map(corpus, PlainTextDocument)                         # removes file information and convert to plain text doc
corpus = tm_map(corpus, removePunctuation)                         # removes Punctuation
corpus = tm_map(corpus, removeNumbers)                             # removes Numbers
corpus = tm_map(corpus, removeWords, stopwords('english'))         # removes Stopwords
corpus = tm_map(corpus, removeWords, c('roadster'))                # remove particular word

corpus <- Corpus(VectorSource(corpus))                  # Some of the tm_map functions do not return a corpus

TDM <- TermDocumentMatrix(corpus)                       # Document matrix is a table containing the frequency of the words. 
                                                        # Column names are words and row names are documents.
TDM <- as.matrix(TDM)                                   # It converts to actual matrix having rows and columns
TDM <- sort(rowSums(TDM),decreasing=TRUE)               # It will sum the values of each row and sort it in decreasing order
TDM <- data.frame(word = names(TDM),freq=TDM)           # Converting to data frame

head(TDM,n=10)                                          # Viewing first 10 values 


wordcloud(words = TDM$word, freq = TDM$freq, min.freq = 1,       # words : the words to be plotted
          max.words=200, random.order=FALSE, rot.per=0.35, 
          colors=brewer.pal(8, "Dark2"))                         # freq : their frequencies
                                                                 # min.freq : words with frequency below min.freq will not be plotted
                                                                 # max.words : maximum number of words to be plotted
                                                                 # random.order : plot words in random order. If false, they will be plotted in decreasing frequency
                                                                 # rot.per : proportion of words with 90 degree rotation (vertical text)
                                                                 # colors : color words from least to most frequent. Use, for example, colors =“black” for single color.
barplot(TDM[1:10,]$freq, las = 2, names.arg = TDM[1:10,]$word,   # las :Labels are parallel (=0) or perpendicular(=2) to axis
        col ="lightblue", main ="Most frequent words",
        ylab = "Word frequencies")
