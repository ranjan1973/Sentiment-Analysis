
                   ####### *********************** #######
########################## IMDB SENTIMENT ANALYSIS ############################
                   ######  *********************** #######


                   ###### SENTIMENT CLASSIFICATION #######
                              ### BASED ON ###
                   ##### SUPERVISED MACHINE LEARNING #####
                 ##### UNSUPERVISED MACHINE LEARNING #####

######################### SET WORKING DIRECTORY ###############################
                   
setwd("C:\\Data Analytics Projects\\Capstone Project\\aclImdb")

######################## LOADING REQUIRED LIBRARIES ##########################
# LOADING LIBRARIES
library(readtext)
library(plyr)
library (dplyr)
library(textstem)
library(ggplot2)
library(tm)
library(wordcloud)
library(RColorBrewer)
library(caret)
library(stringr)
library(tidytext)
library (cluster)
library (data.table)
library(reshape2)
library(tidyr)
library (cluster)
library (fpc)
library (topicmodels)
                   

################################### PHASE ONE #######################################

####### PROBLEM 1 #########
# Read the labelled data from respective folder and store in data frames

#reading text file from train\\pos folder
train_pos <- readtext("C:\\Data Analytics Projects\\Capstone Project\\aclImdb\\train\\pos\\*.txt")

#Looking at the data frame
head(train_pos)

#Getting the number of rows in dataframe
nrow(train_pos) 

#Extracting ID and score from first column using word () function
#ID extraction
train_pos$ID <- sub(".txt", "", train_pos$doc_id)
train_pos$ID

#Score extraction
train_pos$score <- sapply(strsplit(train_pos$ID, "_"), "[", 2)
train_pos$score

#Adding a new column label positive)
train_pos$label <- "positive"
train_pos$label

#Creating data frame for positive reviews
train_pos <- train_pos [, c(2:5)]
head(train_pos)

#Rearranging the data frame for positive review
train_pos <- train_pos [, c(2, 3, 4, 1)]

#Changing the name of the last column from "text" to "Review"
names(train_pos) [4] <- "review"
names(train_pos) 

#reading text file from train\\neg folder
train_neg <- readtext("C:\\Data Analytics Projects\\Capstone Project\\aclImdb\\train\\neg\\*.txt")
train_neg 

#Looking at the data frame
head(train_neg)

#Getting the number of rows in dataframe
nrow(train_neg) 

#Extracting ID and Label from first column using word () function
#ID extraction
train_neg$ID <- sub(".txt", "",train_neg$doc_id)
train_neg$ID

#Label extraction
train_neg$score <- sapply(strsplit(train_neg$ID, "_"), "[", 2)
train_neg$score

#Adding a new column Sentiment negative
train_neg$label <- "negative"
train_neg$label
str(train_neg)

#Creating data frame for test negative reviews
train_neg <- train_neg [, c(2:5)]

#Rearranging the data frame for positive review
train_neg <- train_neg [, c(2, 3, 4, 1)]

#Changing the name of the last column from "text" to "Review"
names(train_neg) [4] <- "review"

names(train_neg) 

#Combining the two dataframes together to create df_train
train <- rbind(train_pos, train_neg)


#reading text file from test\\pos folder
test_pos <- readtext("C:\\Data Analytics Projects\\Capstone Project\\aclImdb\\test\\pos\\*.txt")

#Extracting ID and Label from first column using word () function
test_pos$ID <- sub(".txt", "", test_pos$doc_id)

#Label extraction
test_pos$score <- sapply(strsplit(test_pos$ID, "_"), "[", 2)

#Adding a new column label as positive
test_pos$label <- "positive"

#Creating data frame for positive reviews in test
test_pos <- test_pos [, c(2:5)]

#Rearranging the data frame for positive review
test_pos <- test_pos [, c(2, 3, 4, 1)]

#Changing the name of the last column from "text" to "Review"
names(test_pos) [4] <- "review"

#reading text file from test\\neg folder

test_neg <- readtext("C:\\Data Analytics Projects\\Capstone Project\\aclImdb\\test\\neg\\*.txt")

#Extracting ID and Label from first column using word () function
test_neg$ID <- sub(".txt", "", test_neg$doc_id)

test_neg$score <- sapply(strsplit(test_neg$ID, "_"), "[", 2)

#Adding a new column label as negative
test_neg$label <- "negative"

#Creating test data frame for test negative reviews
test_neg <- test_neg [, c(2:5)]

#Rearranging the data frame for negative review
test_neg <- test_neg [, c(2, 3, 4, 1)]

#Changing the name of the last column from "text" to "Review"
names(test_neg) [4] <- "review"

#Combining the two dataframes together to create df_test
test <- rbind(test_pos, test_neg)


#Create csv file to save in the directory for later use during the project
write.csv(train, "train.csv")
write.csv(test, "test.csv")

#Combing two dataframes together to create labelled data
labelled <- rbind (train, test)


##### STEP TWO #####
#Data cleaning with stemming and lemmatization
#Removing data of https and html tag
labelled$review <- gsub("[^\\x{00}-\\x{7f}]", "", labelled$review , perl = TRUE)
labelled$review  <- gsub("(\n|<br />)"," ", labelled$review )

#Checking a particular document
labelled$review  [22]

#Corpus creation
lab_corp <- Corpus(VectorSource(labelled$review))

#Start preprocessing

toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
lab_corp <- tm_map( lab_corp, toSpace, "https*")
lab_corp <- tm_map( lab_corp, toSpace, "aaa*")

#Remove punctuation - replace punctuation marks with " "
lab_corp <- tm_map(lab_corp, removePunctuation)

#Transform to lower case
lab_corp <- tm_map(lab_corp,content_transformer(tolower))

#Strip digits
lab_corp <- tm_map(lab_corp, removeNumbers)

#Remove stopwords from standard stopword list
lab_corp <- tm_map(lab_corp, removeWords, stopwords("SMART"))

#Applying stemming 
lab_corp <- tm_map (lab_corp, stemDocument)

#Strip whitespace
lab_corp <- tm_map(lab_corp, stripWhitespace)

#Checking the final cleaned version for 5 documents
inspect(lab_corp[5:10])

##############STEP THREE ###################
#Apply feature selection to select most important words/features and drop others

# Create a document-term matrices and remove sparse terms
lab_dtm <- DocumentTermMatrix(lab_corp)
dim(lab_dtm)

# Checking the DTM
lab_dtm

# Applying feature selection to remove terms which is currently 122583

# Step 1: Looking at most frequent words and check their relevance
findFreqTerms(lab_dtm)

# Above steps gives words like movie, movies, film, films due to the domain
# We will make our own stop words and perform on our corpus

mystopwords <- c("movie", "movies", "film", "films", "plot")

#Remove custom stop words
lab_corp <- tm_map(lab_corp, removeWords, mystopwords)

# Again creating DTM
lab_dtm <- DocumentTermMatrix(lab_corp)
lab_dtm


### Step 2 ###
#Remove sparse terms
lab_dtm <- removeSparseTerms(lab_dtm, 0.95)
lab_dtm

#### We have now 174 terms to use for our further analysis ####

########################## END OF PHASE ONE ##################################

                     ### ****************** ###

############################# PHASE TWO ######################################

######### PROBLEM 1 #########
# Find the most common words associated with positive and negative categories

#  To solve this problem we will convert our DTM to tidy format
labelled_td <- tidy(lab_dtm)
labelled_td

# Extract sentiment terms from labelled_td
labelled_sent <- labelled_td %>%
  inner_join(get_sentiments("bing"), by = c(term = "word"))

labelled_sent

# Extracting top "positive" category words
positive_words <- labelled_sent %>%
  filter(sentiment == "positive") %>%
  count (term, sort = TRUE)

print(positive_words)

# Extract top "negative" category words
negative_words <- labelled_sent %>%
  filter(sentiment == "negative") %>%
  count (term, sort = TRUE)

print(negative_words)


######### PROBLEM 2 ##########
# Discover the lowest frequency and highest frequency terms
freq_words <- findFreqTerms(lab_dtm)

# Lowest frequency terms
tail(freq_words, 20)

# Highest frequency terms
head(freq_words, 20)


########## PROBLEM 3 ##########
#Read unlabeled data from respective folder (unsup) and store in unsup_df


#Read the unsup files from train folder
df_train_unsup <- readtext("C:\\Data Analytics Projects\\Capstone Project\\aclImdb\\train\\unsup\\*.txt")

#Extracting ID and Label from first column using word () function
#ID extraction
df_train_unsup$ID <- sub(".txt", "", df_train_unsup$doc_id)

#Label extraction
df_train_unsup$label <- sapply(strsplit(df_train_unsup$ID, "_"), "[", 2)

#Creating data frame for unsupervised
df_train_unsup <- df_train_unsup [, c(2:4)]
head(df_train_unsup)

#Rearranging the data frame 
df_train_unsup <- df_train_unsup [, c(2, 3, 1)]
head(df_train_unsup)

#Changing the name of the last column from "text" to "Review"
names(df_train_unsup) [3] <- "review"
str(df_train_unsup)

#writing CSV to save the data in CSV format for later convenient use
write.csv(df_train_unsup, "unsup.csv")
df_train_unsup <- fread(file = "unsup.csv", stringsAsFactors = FALSE)
head(df_train_unsup)
str(df_train_unsup)

####### PROBLEM 4 #########
## Create a cluster to separate positive and negative words (bonus) using 
## k-means algorithm

# To solve this problem we will have to create a corpus and then prepare a DTM

# Cleaning and corpus creation
df_train_unsup$review <- gsub("[^\\x{00}-\\x{7f}]", "", df_train_unsup$review, perl = TRUE)
df_train_unsup$review <- gsub("(\n|<br />)"," ", df_train_unsup$review)
df_train_unsup$review [22]

unlab_corp <- Corpus(VectorSource(df_train_unsup$review))

#Start preprocessing

toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
unlab_corp <- tm_map( unlab_corp, toSpace, "https*")
unlab_corp <- tm_map( unlab_corp, toSpace, "aaa*")

#Remove punctuation - replace punctuation marks with " "
unlab_corp <- tm_map(unlab_corp, removePunctuation)

#Transform to lower case
unlab_corp <- tm_map(unlab_corp,content_transformer(tolower))

#Strip digits
unlab_corp <- tm_map(unlab_corp, removeNumbers)

#Remove stopwords from standard stopword list
unlab_corp <- tm_map(unlab_corp, removeWords, stopwords("SMART"))

#Applying stemming
unlab_corp <- tm_map(unlab_corp, stemDocument)

# Doing Feature selection
#Remove custom stop words
mystopwords <- c("movie", "movies", "film", "films", "plot")

unlab_corp <- tm_map(unlab_corp, removeWords, mystopwords)

#Strip whitespace 
unlab_corp <- tm_map(unlab_corp, stripWhitespace)


#Checking the final cleaned version for 5 documents
inspect(unlab_corp[5:10])

##############STEP THREE ###################
#DTM Creation

unlab_tdm <- TermDocumentMatrix (unlab_corp)
dim (unlab_tdm)
inspect(unlab_tdm)

unlab_tdm <- removeSparseTerms (unlab_tdm, 0.95)
dim (unlab_tdm)

# build vector to identify non-empty docs
a0 = (apply(unlab_tdm, 1, sum) > 0)   

# drop empty docs
unlab_tdm = unlab_tdm[a0,]                  

dim(unlab_tdm)

#Matrix Creation
m <- as.matrix(unlab_tdm)
head(m)
rownames(m)

values <- sort(rowSums(m), decreasing=TRUE)

# Data Frame creation
Unlab_df <- data.frame(word = names(values), freq = values)

head (Unlab_df)

### Normalizing the vectors 
norm_eucl <- function(m) m/apply(m, MARGIN=1, FUN=function(x) sum(x^2)^.5)
m_norm <- norm_eucl(m)

### K-means clustering into 3 clusters
cl <- kmeans(m_norm, 2)

### Getting the cluster distribution
cl$size

cl$centers

# Creating data frame to assign cluster values
x <- data.frame(rownames(m), K = cl$cluster)
x$class <- ifelse(x$K==1, "positive", "negative")




############################### END OF PHASE TWO ##############################

                     
                       ### ********************** ###

################################ PHASE THREE ###################################

###### PROBLEM 1 ########
# Create a word cloud with positive and negative words after cleansing without feature selection
#inspect frequent words
unlab_tdm_unfeatured <- DocumentTermMatrix(unlab_corp)

unlab_td_unf <- tidy(unlab_tdm_unfeatured)
unlab_td_unf

# word cloud
unlab_td_unf %>%
  count(term) %>%
  with(wordcloud(term, n, random.order = FALSE, rot.per=0.35,
                 colors=brewer.pal(8, "Dark2"), max.words = 100))

#comparison cloud
unlab_td_unf  %>%
  mutate(word = term) %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("#F8766D", "#00BFC4"),
                   max.words = 50)


######## PROBLEM 2 ###########
## Visualise the positive and negative words distribution


#Getting sentiments from the total data
unlab_sentiments_unf <- unlab_td_unf %>%
  mutate(word = term) %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  ungroup()
unlab_sentiments

# Plotting sentiment distribution
unlab_sentiments_unf %>%
  group_by(sentiment) %>%
  top_n(25) %>%
  ungroup() %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(y = "Contribution to sentiment",
       x = NULL) +
  coord_flip()

####### PROBLEM 3 #####
# Repeat visualization step 1 and 2 after feature selection

# Doing Feature selection
#Remove custom stop words related to movie domain
mystopwords <- c("movie", "movies", "film", "films", "plot")
unlab_corp_feat <- tm_map(unlab_corp, removeWords, mystopwords)


unlab_tdm_feat <- TermDocumentMatrix (unlab_corp_feat)
dim (unlab_tdm_feat)

#Remove sparse terms
unlab_tdm_feat <- removeSparseTerms (unlab_tdm_feat, 0.95)
dim (unlab_tdm_feat)

# build vector to identify non-empty docs
a0 <- (apply(unlab_tdm_feat, 1, sum) > 0)   

# drop empty docs
unlab_tdm_feat <- unlab_tdm_feat[a0,]                  

dim(unlab_tdm_feat)

m_feat <- as.matrix(unlab_tdm_feat)
v_feat <- sort(rowSums(m_feat), decreasing=TRUE)
Unlab_df_feat <- data.frame(word = names(v_feat), freq = v_feat)
head (Unlab_df_feat)

# Creating tidy text for feature DTM
unlab_td <- tidy(unlab_tdm_feat)
unlab_td

# word cloud
unlab_td %>%
  count(term) %>%
  with(wordcloud(term, n, random.order = FALSE, rot.per=0.35,
                 colors=brewer.pal(8, "Dark2"), max.words = 100))

#comparison cloud
unlab_td  %>%
  mutate(word = term) %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("#F8766D", "#00BFC4"),
                   max.words = 50)


######## PROBLEM 2 ###########
## Visualise the positive and negative words distribution


#Getting sentiments from the total data
unlab_sentiments_f <- unlab_td %>%
  mutate(word = term) %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  ungroup()
unlab_sentiments_f

# Plotting sentiment distribution
unlab_sentiments_f %>%
  group_by(sentiment) %>%
  top_n(25) %>%
  ungroup() %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(y = "Contribution to sentiment",
       x = NULL) +
  coord_flip()


############################### END OF PHASE 3 ###############################
 
                             ### ************** ###

############################### PHASE FOUR ###################################

##### PROBLEM #######
#Create Hypothesis involving relationships between dependent and independent 
#variables using parametric/non-parametric tests for various machine learning 
#algorithms such as k-means clustering,  classification algorithms.




################################### PHASE FIVE ################################

### PROBLEM 1 ###
#Supervised Learning: Build a sentiment analysis model to predict positive and negative classes 

### Getting the previously saved train and test data
setwd ("C:\\Data Analytics Projects\\Capstone Project\\aclImdb")

train <- fread(file = "train.csv", stringsAsFactors = FALSE)
test <- fread (file = "test.csv", stringsAsFactors = FALSE)
str(train)
str (test)
train$label <- as.factor(train$label)
test$label <- as.factor(test$label)


#Combining df_train and df_test to get labelled data frame
labelled <- rbind(train, test)
str(labelled)
labelled <- labelled[, c(4, 5)]
str(labelled)


#PREPROCESSING, DATA CLEANING, CORPUS AND DTM
#Creating corpus
labelled$review <- gsub("[^\\x{00}-\\x{7f}]", "", labelled$review, perl = TRUE)
labelled$review <- gsub("(\n|<br />)"," ", labelled$review)
labelled$review [22]

lab_corp <- Corpus(VectorSource(labelled$review))
lab_corp

#Start preprocessing
toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
lab_corp <- tm_map( lab_corp, toSpace, "https*")
lab_corp <- tm_map( lab_corp, toSpace, "aaa*")


#Remove punctuation - replace punctuation marks with " "
lab_corp <- tm_map(lab_corp, removePunctuation)

#Transform to lower case
lab_corp <- tm_map(lab_corp,content_transformer(tolower))
#Remove stopwords from standard stopword list
lab_corp <- tm_map(lab_corp, removeWords, stopwords("SMART"))

#Remove custom stop words related to movie domain
mystopwords <- c("movie", "movies", "film", "films", "ive", "hes", "make", "made",
                 "makes", "plot")
lab_corp <- tm_map(lab_corp, removeWords, mystopwords)

#Strip digits
lab_corp <- tm_map(lab_corp, removeNumbers)

# stemming
lab_corp <- tm_map (lab_corp, stemDocument)


#Strip whitespace
lab_corp <- tm_map(lab_corp, stripWhitespace)

#DTM creation
lab_dtm <- DocumentTermMatrix (lab_corp)
dim(lab_dtm)

lab_dtm_tfidf <- weightTfIdf(lab_dtm)

#Removing sparse terms
lab_dtm_tfidf <- removeSparseTerms (lab_dtm_tfidf, 0.95)
dim (lab_dtm_tfidf)
lab_dtm_tfidf


# Splitting DTM into training, validation and test sets
tw_dtm <- as.data.frame(as.matrix(lab_dtm_tfidf))
dim(tw_dtm)
lab_df <- cbind(labelled[, 1], tw_dtm)
dim(lab_df)

train_dtm <- lab_df[1:25000, ]
test_dtm <- lab_df[25001:50000, ]
tst_dtm <- cbind(test$label, test_dtm)

# Splitting train_dtm into train and validation sets
set.seed (123)

ind <- createDataPartition(train_dtm$label, p = 0.75, list = FALSE)

#Trainiing DTM
tr_dtm <- train_dtm [ind, ]
dim(tr_dtm)
table(tr_dtm$label)
tr_dtm$label <- as.factor(tr_dtm$label)

#Validation DTM
val_dtm <- train_dtm [-ind, ]
dim(tr_dtm)
table(val_dtm$label)
val_dtm$label <- as.factor(val_dtm$label)

##### MODEL CREATION AND COMPARISION OF DIFFERENT MODEL PERFORMANCE ##########

## Creating and evaluating Naive Bayes model
library(e1071)
class_nb <- naiveBayes(label ~ ., data = tr_dtm, laplace = 100, na.action)
summary(class_nb)

# Doing prediction
pred_nb <- predict(class_nb, newdata = val_dtm)
summary(pred_nb)

#checking accuracy
tab_nb <- table(pred_nb, val_dtm$label)

confusionMatrix(tab_nb)

# Naive Bayes Acccuracy = 76.19 % on 95% CI


# Creating and evaluating Ctree method
library(party)


class_ct <- ctree(label ~ ., data = tr_dtm)
summary(class_ct)
plot(class_ct, type = "simple")

# Doing prediction
pred_ct <- predict(class_ct, newdata = val_dtm)
summary(pred_ct)

#checking accuracy
tab_ct <- table(pred_ct, val_dtm$label)

confusionMatrix(tab_ct)

# Ctree Model Accuracy = 75.02%

# SVM Model
library (kernlab)

class_svm <- ksvm(label ~ ., data = tr_dtm)
summary(class_svm)


# Doing prediction
pred_svm <- predict(class_svm, newdata = val_dtm)
summary(pred_svm)

#checking accuracy
tab_svm <- table(pred_svm, val_dtm$label)

confusionMatrix(tab_svm)

# SVM Model Accuracy = 81.09

#  Random Forest Model

library (randomForest)
class_rf <- randomForest(label ~ ., data = tr_dtm, ntree =500, importance = TRUE, mtry =3)
summary(class_rf)


# Doing prediction
pred_rf <- predict(class_rf, newdata = val_dtm)
summary(pred_rf)

#checking accuracy
tab_rf <- table(pred_rf, val_dtm$label)

confusionMatrix(tab_rf)

# Random Forest Accuracy = 80.99

# Selecting the best model and predicting on Test data
# Got best accuracy from SVM model

pred <- predict(class_svm, newdata = tst_dtm)

#Checking accuracy
tab <- table(pred, tst_dtm$label)
confusionMatrix(tab)

#Creating submission csv
submission_sup <- data.frame('review' = test$review, 'label' = as.character(class_rf))
write.csv(submission_sup, "supervised.csv")


#### PROBLEM TWO ####
# Unsupervised learning
# Build a clustering model consisting of 2 clusters based on positive and negative reviews 

df_train_unsup <- fread(file = "unsup.csv", stringsAsFactors = FALSE)
head(df_train_unsup)
str(df_train_unsup)

####### PROBLEM 4 #########
## Create a cluster to separate positive and negative words (bonus) using 
## k-means algorithm

# To solve this problem we will have to create a corpus and then prepare a DTM

# Cleaning and corpus creation
df_train_unsup$review <- gsub("[^\\x{00}-\\x{7f}]", "", df_train_unsup$review, perl = TRUE)
df_train_unsup$review <- gsub("(\n|<br />)"," ", df_train_unsup$review)
df_train_unsup$review [22]

unlab_corp <- Corpus(VectorSource(df_train_unsup$review))

#Start preprocessing

toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
unlab_corp <- tm_map( unlab_corp, toSpace, "https*")
unlab_corp <- tm_map( unlab_corp, toSpace, "aaa*")

#Remove punctuation - replace punctuation marks with " "
unlab_corp <- tm_map(unlab_corp, removePunctuation)

#Transform to lower case
unlab_corp <- tm_map(unlab_corp,content_transformer(tolower))

#Strip digits
unlab_corp <- tm_map(unlab_corp, removeNumbers)

#Remove stopwords from standard stopword list
unlab_corp <- tm_map(unlab_corp, removeWords, stopwords("SMART"))

mystopwords <- c("movie", "movies", "film", "films", "ive", "hes", "make", "made",
                 "makes")
unlab_corp <- tm_map(unlab_corp, removeWords, mystopwords)

#stemming the document
unlab_corp <- tm_map (unlab_corp, stemDocument)


#Strip whitespace 
unlab_corp <- tm_map(unlab_corp, stripWhitespace)

inspect(unlab_corp[5:10])

##############STEP THREE ###################
#Apply feature selection to select most important words/features and drop others

unlab_tdm <- DocumentTermMatrix (unlab_corp)
dim (unlab_tdm)

#Removing sparse terms
unlab_tdm <- removeSparseTerms (unlab_tdm, 0.98)


#applying tf-idf
unlab_tdm <- weightTfIdf(unlab_tdm)

################## K-MEANS CLUSTERING #######################
#converting to matrix
m1<- as.matrix(unlab_tdm)

#normalizing the matrix by euclidean distance
norm_eucl <- function(m1)
  m1/apply(m1,1,function(x) sum(x^2)^.5)

m_norm1 <- norm_eucl(m1)

# Running k-means algorithm
set.seed (221)
k <- 2
kmeansResult <- kmeans(m_norm1, k)
length(kmeansResult$cluster)
kmeansResult$size

#visualizing the cluster
kmeansResult$cluster[1:10]

# Counting the documents in each cluster
table(kmeansResult$cluster)

# Getting the result
unsup <- data.frame (ID = df_train_unsup$V1, review = df_train_unsup$review, label = ifelse(kmeansResult$cluster == 1, "positive", "negative"))
head(unsup)

# Writing the csv
write.csv(unsup, "unsupervised.csv")

#########################  END OF K-MEANS CLUSTERING #####################


#########################  LDA CLUSTERING #######################

#Latent Dirichlet allocation (LDA) is a particularly popular method for fitting 
#a topic model. It treats each document as a mixture of topics, and each topic 
#as a mixture of words. This allows documents to "overlap" each other in terms 
#of content, rather than being separated into discrete groups, in a way that 
#mirrors typical use of natural language.

#Apply feature selection to select most important words/features and drop others

unlab_tdm1 <- DocumentTermMatrix (unlab_corp)
dim (unlab_tdm1)

#Removing sparse terms
unlab_tdm1 <- removeSparseTerms (unlab_tdm1, 0.98)

#LDA topic clustering
unlab_topic <- LDA(unlab_tdm1, 2)
topics(unlab_topic)

# Counting the documents in each topic
table(topics(unlab_topic))


# Getting the result
unsup_lda <- data.frame (ID = df_train_unsup$V1, review = df_train_unsup$review, label = ifelse(topics(unlab_topic) == 1, "positive", "negative"))
head(unsup_lda, 1)

# Writing the csv
write.csv(unsup_lda, "unsupervised_lda.csv")

############################# END OF LDA ####################################


############################ DICTIONARY BASED CLUSTERING ####################
###### COMPARISION OF CLUSTERING MODEL ##############

### K-means cluster table
table(kmeansResult$cluster)
prop.table(table(kmeansResult$cluster))


### LDA cluster table
table(topics(unlab_topic))

prop.table(table(topics(unlab_topic)))

###RESULT:  LDA gives almost equivalent distribution of 48.4% and 51.6% , so LDA 
# can be considered a better model in this problem

######################## END OF CLUSTER COMPARISON ############################

###### PROBLEM ######

#Divide the data into 4 clusters to enable finding more classes. Analyse each 
#cluster and try to find the correct label for the new cluster. Repeat clustering
#until 4 new labels can be found, other than the original labels (positive and negative)
head(unsup_lda, 10) 

#Split the labelled dataset into labels
mylist <- split(unsup_lda, unsup_lda$label)

unsup_pos <- as.data.frame(mylist$positive)
unsup_neg <- as.data.frame(mylist$negative)

nrow(unsup_pos)
nrow(unsup_neg)

# Clustering the unsup_pos and unsup_neg into 3 clusters based on LDA

unlab_corp1 <- Corpus(VectorSource(unsup_pos$review))

#Start preprocessing

toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
unlab_corp1 <- tm_map( unlab_corp1, toSpace, "https*")
unlab_corp1 <- tm_map( unlab_corp1, toSpace, "aaa*")

#Remove punctuation - replace punctuation marks with " "
unlab_corp1 <- tm_map(unlab_corp1, removePunctuation)

#Transform to lower case
unlab_corp1 <- tm_map(unlab_corp1,content_transformer(tolower))

#Strip digits
unlab_corp1 <- tm_map(unlab_corp1, removeNumbers)

#Remove stopwords from standard stopword list
unlab_corp1 <- tm_map(unlab_corp1, removeWords, stopwords("SMART"))

mystopwords <- c("movie", "movies", "film", "films")
unlab_corp1 <- tm_map(unlab_corp1, removeWords, mystopwords)

#stemming the document
unlab_corp1 <- tm_map (unlab_corp1, stemDocument)


#Strip whitespace 
unlab_corp1 <- tm_map(unlab_corp1, stripWhitespace)


#unsup_neg
unlab_corp2 <- Corpus(VectorSource(unsup_neg$review))

#Start preprocessing

toSpace <- content_transformer( function(x, pattern) gsub(pattern," ",x) )
unlab_corp2 <- tm_map( unlab_corp2, toSpace, "https*")
unlab_corp2 <- tm_map( unlab_corp2, toSpace, "aaa*")

#Remove punctuation - replace punctuation marks with " "
unlab_corp2 <- tm_map(unlab_corp2, removePunctuation)

#Transform to lower case
unlab_corp2 <- tm_map(unlab_corp2,content_transformer(tolower))


#Strip digits
unlab_corp2 <- tm_map(unlab_corp2, removeNumbers)

#Remove stopwords from standard stopword list
unlab_corp2 <- tm_map(unlab_corp2, removeWords, stopwords("SMART"))

mystopwords <- c("movie", "movies", "film", "films", "ive", "hes", "make", "made",
                 "makes")
unlab_corp2 <- tm_map(unlab_corp2, removeWords, mystopwords)

#stemming the document
unlab_corp2 <- tm_map (unlab_corp2, stemDocument)


#Strip whitespace 
unlab_corp2 <- tm_map(unlab_corp2, stripWhitespace)

# LDA clustering of positive reviews
unlab_tdm_ <- DocumentTermMatrix (unlab_corp1)
dim (unlab_tdm_pos)

#Removing sparse terms
unlab_tdm_pos <- removeSparseTerms (unlab_tdm_pos, 0.98)
unlab_topic1 <- LDA(unlab_tdm_pos, 3)

# Counting the documents in each topic
table(topics(unlab_topic1))

# Getting the result
unsup_lda_pos <- data.frame (ID = unsup_pos$ID, review = unsup_pos$review, label = (ifelse(topics(unlab_topic1) == 1, "positive", 
          ifelse(topics(unlab_topic1) == 2, "very positive", "highly positive"))))
head(unsup_lda_pos)
table(unsup_lda_pos$label)


# LDA clustering of negative reviews
unlab_tdm_neg <- DocumentTermMatrix (unlab_corp2)
dim (unlab_tdm_neg)

#Removing sparse terms
unlab_tdm_neg <- removeSparseTerms (unlab_tdm_neg, 0.98)
unlab_topic2 <- LDA(unlab_tdm_neg, 3)

# Counting the documents in each topic
table(topics(unlab_topic2))

# Getting the result
unsup_lda_neg <- data.frame (ID = unsup_neg$ID, review = unsup_neg$review, label = (ifelse(topics(unlab_topic2) == 1, "negative", 
        ifelse(topics(unlab_topic2) == 2, "very negative", "highly negative"))))
head(unsup_lda_neg)
table(unsup_lda_neg$label)

#combining the unsupervised positive and negative into unsupervised data
unsupervised <- rbind(unsup_lda_pos, unsup_lda_neg)
nrow(unsupervised)
head(unsupervised, 10)

write.csv(unsupervised, "unlabelled.csv")


########################### END OF PROJECT ####################################

