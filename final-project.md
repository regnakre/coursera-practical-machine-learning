#Practical Machine Learning - Final Project
<hr/>
##Data Preparation and Exploration
  
First, let's install/include the required libraries

    library(lattice); library(ggplot2); library(caret); library(randomForest); 
    library(rpart); library(rpart.plot);library(RColorBrewer);library(rattle)
    
Then, let's read the training and testing data.

    training <- read.csv("pml-training.csv", na.strings=c("NA","#DIV/0!", ""))
    testing <- read.csv("pml-testing.csv", na.strings=c("NA","#DIV/0!", ""))

    # Show the colums with NA count
    data.frame(colSums(is.na(training)))

Clean the NA rows:

    # remove the columns that contains only NA values
    training <- training[, colSums(is.na(training)) == 0]
    testing <- testing[, colSums(is.na(testing)) == 0]
    
Delete the variables that are irrelevant: user_name, raw_timestamp_part_1, raw_timestamp_part_,2 cvtd_timestamp, new_window, and  num_window (columns 1 to 7). 

    training   <- training[,-c(1:7)]
    testing <- testing[,-c(1:7)]
    
Partition the data so that 70% of the training dataset into training and the remaining 30% to testing

    intrain <- createDataPartition(y = training$classe, p = 0.7, list=FALSE)
    training_train <- training[intrain,]
    training_test <- training[-intrain,]
    
Let's plot the number of items in each classe with a bar graph

    plot(training_train$classe, col="gray", main="# of Classes at Each Level", xlab="classe", ylab="frequency")
    
![Alt text](/plot1.png?raw=true "Frequencies of Classes")

As seen in the figure above, level A is the most frequent while level D is the least frequent.
  
## Predicting with Decision Tree

I will first try decision tree to fit the model.

Cross validation with 5-fold will be used:

    control_dt <- trainControl(method="cv", 5)
    mod_dt <- train(classe~., data = training_train, method = "rpart",
                trControl = control_dt)

    pred_dt <- predict(mod_dt, training_test)
    confusionMatrix(pred_dt, training_test$classe)

Here are the Decision Tree Classification results:

    pred_dt <- predict(mod_dt, training_test, type = "class")
    confusionMatrix(pred_dt, training_test$classe)

              Reference
    Prediction    A    B    C    D    E
             A 1531  483  474  445  145
             B   21  384   36  162  142
             C  118  272  516  357  303
             D    0    0    0    0    0
             E    4    0    0    0  492

    Overall Statistics
                                              
                   Accuracy : 0.4967          
                     95% CI : (0.4838, 0.5095)
        No Information Rate : 0.2845          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.3419          
     Mcnemar's Test P-Value : NA              

    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9146  0.33714  0.50292   0.0000  0.45471
    Specificity            0.6326  0.92394  0.78391   1.0000  0.99917
    Pos Pred Value         0.4974  0.51544  0.32950      NaN  0.99194
    Neg Pred Value         0.9491  0.85311  0.88192   0.8362  0.89052
    Prevalence             0.2845  0.19354  0.17434   0.1638  0.18386
    Detection Rate         0.2602  0.06525  0.08768   0.0000  0.08360
    Detection Prevalence   0.5230  0.12659  0.26610   0.0000  0.08428
    Balanced Accuracy      0.7736  0.63054  0.64342   0.5000  0.72694

## Predicting with Random Forest

Second, I will use Random Forest algorithm to fit the model.

Again, cross validation with 5-fold will be used:

    control_rf <- trainControl(method = "cv", 5)
    mod_rf <- train(classe ~., data = training_train, method = "rf",  
                    trControl=control_rf, ntree=250)
    
    pred_rf <- predict(mod_rf, training_test)
    confusionMatrix(pred_rf, training_test$classe)
  
Here are the Random Forest Classification results:
  
              Reference
    Prediction    A    B    C    D    E
             A 1531  483  474  445  145
             B   21  384   36  162  142
             C  118  272  516  357  303
             D    0    0    0    0    0
             E    4    0    0    0  492
    
    Overall Statistics
                                              
                   Accuracy : 0.4967          
                     95% CI : (0.4838, 0.5095)
        No Information Rate : 0.2845          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.3419          
     Mcnemar's Test P-Value : NA              
    
    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9146  0.33714  0.50292   0.0000  0.45471
    Specificity            0.6326  0.92394  0.78391   1.0000  0.99917
    Pos Pred Value         0.4974  0.51544  0.32950      NaN  0.99194
    Neg Pred Value         0.9491  0.85311  0.88192   0.8362  0.89052
    Prevalence             0.2845  0.19354  0.17434   0.1638  0.18386
    Detection Rate         0.2602  0.06525  0.08768   0.0000  0.08360
    Detection Prevalence   0.5230  0.12659  0.26610   0.0000  0.08428
    Balanced Accuracy      0.7736  0.63054  0.64342   0.5000  0.72694
      
## Choosing the Prediction Model and Testing
Since accuracy for Random Forest model is greater than the accuracy achieved with Decision Tree model, the Random Forests model is choosen for predicting the test data.

    The Results of Final Prediction on Testing Data Set
      
    pred_final <- predict(mod_rf, testing)
    pred_final
    
    [1] B A B A A E D B A A B C B A E E A B B B
    Levels: A B C D E
