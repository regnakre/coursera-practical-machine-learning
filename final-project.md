#Practical Machine Learning - Final Project
<hr/>
##Preparing the Data and Exploration
  
    # install the required libraries
    library(lattice); library(ggplot2); library(caret); library(randomForest); 
    library(rpart); library(rpart.plot);library(RColorBrewer);library(rattle)
    
    training <- read.csv("pml-training.csv", na.strings=c("NA","#DIV/0!", ""))
    testing <- read.csv("pml-testing.csv", na.strings=c("NA","#DIV/0!", ""))

    # Show the colums with NA count
    data.frame(colSums(is.na(training)))

    # remove the columns that contains only NA values
    training <- training[, colSums(is.na(training)) == 0]
    testing <- testing[, colSums(is.na(testing)) == 0]
    
    # Delete variables that are irrelevant: user_name, raw_timestamp_part_1, 
    # raw_timestamp_part_,2 cvtd_timestamp, new_window, and  num_window (columns 1 to 7). 
    training   <- training[,-c(1:7)]
    testing <- testing[,-c(1:7)]
    
    # partition the data so that 70% of the training dataset into training and the remaining 30% to testing
    intrain <- createDataPartition(y = training$classe, p = 0.7, list=FALSE)
    training_train <- training[intrain,]
    training_test <- training[-intrain,]
    
    # plotting the number of items in each classe with a bar graph
    plot(training_train$classe, col="gray", main="# of Classes at Each Level", xlab="classe", ylab="frequency")
    
![Alt text](/plot1.png?raw=true "Frequencies of Classes")

As seen in the figure above, level A is the most frequent while level D is the least frequent.
  
## Predicting with Decision Tree

    ### Predicting with Decision Tree
    mod_dt <- rpart(classe~., data = training_train, method = "class")
    png('plot2_dt.png')
    fancyRpartPlot(mod_dt);dev.off()
    
    pred_dt <- predict(mod_dt, training_test, type = "class")
    confusionMatrix(pred_dt, training_test$classe)


## Results of Decision Tree Prediction
  
    Reference
    Prediction    A    B    C    D    E
             A 1535  172   26   52   15
             B   54  639   52   76   79
             C   47  228  867   96  150
             D   27   74   81  650   87
             E   11   26    0   90  751

    Overall Statistics
                                              
                   Accuracy : 0.7548          
                     95% CI : (0.7436, 0.7657)
        No Information Rate : 0.2845          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.6893          
     Mcnemar's Test P-Value : < 2.2e-16       

    Statistics by Class:
    
                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9170   0.5610   0.8450   0.6743   0.6941
    Specificity            0.9371   0.9450   0.8928   0.9453   0.9736
    Pos Pred Value         0.8528   0.7100   0.6246   0.7073   0.8554
    Neg Pred Value         0.9660   0.8997   0.9646   0.9368   0.9339
    Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    Detection Rate         0.2608   0.1086   0.1473   0.1105   0.1276
    Detection Prevalence   0.3059   0.1529   0.2359   0.1562   0.1492
    Balanced Accuracy      0.9270   0.7530   0.8689   0.8098   0.8338
  
## Predicting with Random Forest
  
    mod_rf <- randomForest(classe ~., data = training_train, method = "class")
    pred_rf <- predict(mod_rf, training_test, type = "class")
    confusionMatrix(pred_rf, training_test$classe)
  
## Results of Random Forest Prediction
  
        Confusion Matrix and Statistics
    
              Reference
    Prediction    A    B    C    D    E
             A 1672    2    0    0    0
             B    2 1135    4    0    0
             C    0    2 1019   12    1
             D    0    0    3  949    0
             E    0    0    0    3 1081
    
    Overall Statistics
                                              
                   Accuracy : 0.9951          
                     95% CI : (0.9929, 0.9967)
        No Information Rate : 0.2845          
        P-Value [Acc > NIR] : < 2.2e-16       
                                              
                      Kappa : 0.9938          
     Mcnemar's Test P-Value : NA              

## Statistics by Class:

                         Class: A Class: B Class: C Class: D Class: E
    Sensitivity            0.9988   0.9965   0.9932   0.9844   0.9991
    Specificity            0.9995   0.9987   0.9969   0.9994   0.9994
    Pos Pred Value         0.9988   0.9947   0.9855   0.9968   0.9972
    Neg Pred Value         0.9995   0.9992   0.9986   0.9970   0.9998
    Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    Detection Rate         0.2841   0.1929   0.1732   0.1613   0.1837
    Detection Prevalence   0.2845   0.1939   0.1757   0.1618   0.1842
    Balanced Accuracy      0.9992   0.9976   0.9950   0.9919   0.9992
      
## Choosing the Prediction Model and Testing
    Since accuracy for Random Forest model (0.995) is greater than the accuracy achieved with Decision Tree model (0.739), the Random Forests model is choosen for predicting the test data.
    The Results of Final Prediction on Testing Data Set
      
    pred_final <- predict(mod_rf, testing, type = "class")
    pred_final
    
    1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    Levels: A B C D E
