---
layout: post
modal-id: 3
title: Trajectory Prediction
subtitle : EY Nextwave Data Science Challenge 2019
date: 2019-05-01
image: "img/trajectoryphoto.png"
alt: image-alt
project-date: May 2019
coverimage : eycover.jpg
githublink : https://github.com/vvoluom/Ey_DataScience_Challenge
tags: [DataScience, A.I]
---

<img src="../images/eynextwave/coverphoto.jpg" alt="linearly separable data">

# Resources 
The Github Repository can be found here : [EY Data Science Challenge](https://github.com/vvoluom/Ey_DataScience_Challenge)  
EY Link With Results : [NextWave](https://www.ey.com/en_in/careers/nextwave-data-science-challenge)

# Abstract
This is my solution for the EY Nextwave Data Science Challenge 2019 where it was rewarded 1st Place in Malta and 67t Place Globally. This is a trajectory prediction problem with regards to cars in the densly populated city of Atlanta, Georgia US. Given a car journey containing multiple trajectories the objective is to predict the final exit point of that journey. A trajectory is defined as a route taken by a person, with an entry point (x , y) at a time entry and an exit point (x , y) at a time exit.

# Context of The Challenge

The EY NextWave Data Science Challenge 2019 focuses on how data can help the next smart city thrive, and boost the mobility of the future. Global urbanization is on the rise, with more than 50% of the world’s population living in cities; according to the UN, that number will reach 60% by 2030 – that’s nearly 1.5 billion more than in 2010.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; While this trend creates great opportunities for cities, it also presents challenges to governments on how to upgrade infrastructure, alleviate congestion and address pollution.  
Electric and autonomous vehicles, along with the explosion of the ride sharing economy, are helping to address these challenges which also disrupt mobility and demand innovative solutions.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; In parallel, public authorities have more information than ever on how citizens move around in the city. However, a gap exists between having this data and using it to improve the user travel experience for citizens. Forward-looking authorities have a chance to innovate infrastructure to make their city a better place to live in a better working world.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Here’s your chance to narrow that gap. As a challenge participant, you will be able to download a dataset with a vast number of anonymous geolocation records from the US city of Atlanta (Georgia), during October 2018. Your task is to produce a model that helps authorities to understand the journeys of citizens while they move in the city throughout the day. If you dig deep enough, your work could inspire solutions that help city authorities anticipate disruptions, make real-time decisions, design new services, and reshape infrastructures in order that cities as smart as their citizens.

## Data Description

The data contains the anonymized geolocation data of multiple mobile devices in the City of Atlanta (US) for 11 working days in October 2018. The devices’ ID resets every 24 hours; therefore, you will not be able to trace the same device across different days. Therefore, every device ID represents a 1-day journey.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Each journey is formed by several trajectories. A trajectory is defined as the route of a moving person in a straight line with an entry and an exit point. See an example below of one trajectory from one of the devices:

<img src="../images/eynextwave/trajectory.png" alt="linearly separable data" class="center">

As you can see, trajectories are a simplification of the real path of a person.
A trajectory ends when a person stops moving and stays in the same place for a while and when the device stops recording for some time.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  For each device you will get multiple trajectories. The set of all trajectories of a device represents a simplification of the journey of one person for 24 hours. The graphic below shows a full journey of a device. 

<img src="../images/eynextwave/trajectories.png" alt="linearly separable data" class="center">

Trajectories are separated. In the graph, this separation is shown as a dotted line between the exit point of a trajectory and the entry point of the next one. These dotted lines represent blind parts of the journey where the device did not record the location.

## Dataset Details

There are approximately 210,000 devices and 11 columns in the database. The dataset was provided in two files a training set (data_train.csv) and testing set (data_test.csv). The train dataset contains 80% of the records, while the test dataset contains 20%. The variables in the dataset are as follows:

<img src="../images/eynextwave/datavariables.png" alt="linearly separable data" class="center">

## The Goal
The Goal is to predict how many people are in the city center between 15:00 and 16:00. The test dataset contains a number of devices where the trajectories after 15:00 have been removed. All but one: After 15:00, you will find one last trajectory, with (1) entry location, (2) entry time and an exit time that is between 15:00 and 16:00. But the exit point has been removed. The task is to predict the location of this last exit point and whether this device is within the city center or not.

<img src="../images/eynextwave/atlantaTraj.png" alt="linearly separable data" class="center">

After an estimation is made with regards to the position of each target those estimated coordinates will have to be classified whether they are located inside the city center or not. The city center is located within the coordinates below:

<img src="../images/eynextwave/centerCoordinates.png" alt="linearly separable data" class="center">

If the coordinates land within the center they are marked as (1) and if they land outside they are marked as (0). The submission file is a 2 column file with trajectory_id as the first column and whether it ends in the city center (1) or not (0). Submissions were evaluated using the F1-score between the predicted and the observed target.

# Methodoloy

## Data Preparation
The data entries are grouped by hash values which correspond to mobile device
ID’s. One hash value will have multiple trajectories which make up a full journey.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Given trajectories T1, T2, T3, T4 in on journey one could make multiple smaller journeys from this group of trajectories.
For Example :
* Journey One  : T1->T2  
* Journey Two  : T1->T2->T3
* Journey Four : T1->T2->T3->T4

### Data Transformation

Using the Python Pandas library the Training and Testing CSV files had been loaded as can be seen in the code snippet below:

```python
    #Load the training and testing data
    df_train =pd.read_csv('data_train.csv')
    df_test =pd.read_csv('data_test.csv')
```

The dataset had variables that contain time, the time_entry (When the device started recording gps movement) and the time_exit (When the device stopped recording gps movement). This time has been saved in the format HH:MM:SS (Hours:Miniutes:Seconds) which needed to be converted into an integer number. This was done by using the function to_timedelta() from the Pandas library as can be seen in the code below.

```python
    #Changing the TIME to Hours
    df_train['time_entry']=pd.to_timedelta(df_train['time_entry'])/pd.offsets.Hour(1)
    df_train['time_exit']=pd.to_timedelta(df_train['time_exit'])/pd.offsets.Hour(1)

    df_test['time_entry']=pd.to_timedelta(df_test['time_entry'])/pd.offsets.Hour(1)
    df_test['time_exit']=pd.to_timedelta(df_test['time_exit'])/pd.offsets.Hour(1)
```

### Feature Engineering

The single trajectories were then grouped by hash making them full length journeys. From the grouped trajectories, clusters had been extracted for the x_entry and the y_entry coordinates using [MeanShift Clustering](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.MeanShift.html).  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Consider data points on a 2D plane, those data points would be concentrated in certain areas more than others (High Density). You can think of this as a hill, the denser the certain area is the higher the hill.  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  Mean shift clustering aims to discover hills in this 2D plane. This centroid-based algorithm will update the center of a hill by choosing the mean of all the points inside the given region. The code described in the above proceedure can be seen below:

MeanShift Clustering function used below from the library sklearn.
```python
    def get_clusters(coords):
        clusters = pd.DataFrame({
            'approx_latitudes': coords[:,0].round(4),
            'approx_longitudes': coords[:,1].round(4)
        })
        #Removing duplicate coordinates
        clusters = clusters.drop_duplicates(['approx_latitudes', 'approx_longitudes'])
        clusters = clusters.as_matrix()
        #Setting the bandwidth of clusters
        bandwidth = estimate_bandwidth(clusters, quantile=0.00002)
        ms = MeanShift(bandwidth=bandwidth, bin_seeding=True).fit(clusters)
        #return Meanshift object
        return ms
```
Function to create a journey from trajectories
```python
    def create_polyline_x(x_entry):
        t_x=np.array(x_entry)
        return [t_x]
```
Putting it all together to create a MeanShift Clustering Object which will be able to predict under which cluster a given coordinate will land.
```python
    #Group Trajectories by Hash Making them a full Journey
    hash_group=df_train.groupby(['hash'])
    #Taking all the coordinates to create clusters.
    polyline =hash_group[['x_entry','y_entry']].apply(lambda x:create_polyline_x(x))
    feats = polyline.to_frame(name='polyline')
    latlong = np.array([[p[0][0][1], p[0][0][0]] for p in feats['polyline'] if len(p)>0])
    #Retrieves the MeanShift Object which can now run predictions
    ms = get_clusters(latlong)
```

In the image below the clusters can be visualized on the roads of Atlanta, as the color of the clusters is Red-shifted it means that there is a high density of coordinates there, the opposite is true for Clusters that are shifting towards yellow. One can see that the City Center is densily populated. 

<img src="../images/eynextwave/clusteringFeatures.png" alt="linearly separable data">

After the MeanShift Clustering model is created, it is used to predict the cluster numbers of the entry and exit coordinates. These cluster numbers are then stored in the data file as additional features as seen in the code below:

```python
    #Set points to Clusters and add to dataframes
    df_train.loc[:, 'pickup_cluster'] = ms.predict(df_train[['y_entry', 'x_entry']])
    df_train.loc[:, 'dropoff_cluster'] = ms.predict(df_train[['y_exit', 'x_exit']])
```

The raw data including the clusters is taken and features are crafted from them, to be able to train the model on. A list of features was then crafted to describe the data as best as possible.

## Model Used

### Extreme Gradient Boosting (XGBoost)
XGBoost is a supervised learning algorithm based on decision tree boosting. Supervised learning is when the training data X (containing multiple features)
predicts the target variable Y. Decision trees are flowchart-like structures with decision points as nodes and branches as weighted answers, an example can be seen in the image below:

<img src="../images/eynextwave/decisiontree.png" alt="linearly separable data" class="center">

The image above showed a shallow decision tree which would be called a weak learner. Weak learner means individually that tree would be inaccurate but better than
random guessing. Decision Tree Boosting is the process of taking many individual weak learners and combining them into one strong learner.  

Different models have been used for this problem. This problem was taken as both a Classification and a Regression problem. Since XGBoost doesn’t offer functionality for multi-label/multi-regression outputs two regression models needed to be trained, one for the X exit points and another for the Y exit points. The Regression model was trained on the X and Y Exit points while the Classification model had the X and Y points converted to a single binary label stating whether or not those points are in the city center or not. These models had been trained for 100 Epochs with a learning rate of 0.2 and a maximum tree depth of 14 using the XGboost python library.

Training the data as a Classification problem:
```python
    xgbRegClass =xgb.XGBClassifier(n_estimators=100,
                            max_depth=14,
                            learning_rate=0.2,
                            gamma=0.1,
                            early_stopping_rounds=3)

    xgbRegClass.fit(X_train,y_train,eval_metric='error',eval_set=[(X_test,y_test)],verbose=True)
```

Training the data as a Regression Problem for Latitude Coordinates.
```python
    #This is a Latitude Regressor, it should be used to predict the latitutde of the trajectories alone
    xgbRegY = xgb.XGBRegressor(n_estimators=100, learning_rate=0.2, gamma=0, subsample=0.75,
                            colsample_bytree=1, max_depth=7)

    xgbRegY.fit(X_train,y_train,eval_metric='rmse',eval_set=[(X_test,y_test)],verbose=True)
```
Training the data as a Regression Problem for Longitude Coordinates.
```python
    #This is a Longitude Regressor, it should be used to predict the longitude of the trajectories alone
    xgbRegX = xgb.XGBRegressor(n_estimators=100, learning_rate=0.2, gamma=0, subsample=0.75,
                            colsample_bytree=1, max_depth=7)

    xgbRegX.fit(X_train,y_train,eval_metric='rmse',eval_set=[(X_test,y_test)],verbose=True)
```

Running the predictions on the Engineered Features from the Test Data:

```python
    predictionsClass = xgbRegClass.predict(Testing_X)
    predictionsY = xgbRegY.predict(Testing_X)
    predictionsx = xgbRegx.predict(Testing_X)
```
A custom F1-Score function was created to measure the accuracy of the models by comparing the training and testing. The code below shows the function to evaluate the Regression Model.

```python
    #Function to evaluate the Regressor
    def evaluate_predictions_Reg(predictionsX, predictionsY, ActualsY, Label_X):
        TP = 0
        FP = 0
        FN = 0
        for i in range(len(predictionsX)):
            if str(Label_X[i]) != "nan":
                if (3750901.5068 <= predictionsX[i]) and (predictionsX[i] <= 3770901.5068) and ( -19268905.6133 <= predictionsY[i]) and (predictionsY[i] <= -19208905.6133):
                    Answer = 1
                else:
                    Answer = 0
                if Answer == ActualsY[i] and ActualsY[i] == 1:
                    TP +=1 
                elif Answer != ActualsY[i] and ActualsY[i] == 1:
                    FN +=1 
                elif Answer != ActualsY[i] and Answer == 1:
                    FP +=1
        
        return TP,FP,FN
    
    #Retrieve the True Positive, False Positive and False Negatives of the regressor
    TP,FP,FN = evaluate_predictions_Reg(predictionsX,predictionsY,Output,Label_X)
```

The same was done to evalutate the Classification model as can be seen below:

```python
#Evaluate the predictions made by the Classifier
    def evaluate_predictions(predictionsX,ActualsY,Label_X):
        TP = 0
        FP = 0
        FN = 0
        for i in range(len(predictionsX)):
            if str(Label_X[i]) != "nan":
                if predictionsX[i] > 0.5:
                    predictionsX[i] = 1
                else:
                    predictionsX[i] = 0
                
                if int(predictionsX[i]) == ActualsY[i] and ActualsY[i] == 1:
                    TP +=1 
                elif int(predictionsX[i]) != ActualsY[i] and ActualsY[i] == 1:
                    FN +=1 
                elif int(predictionsX[i]) != ActualsY[i] and int(predictionsX[i]) == 1:
                    FP +=1
        
        return TP,FP,FN
        #Retrieve the True Positive, False Positive and False Negatives of the Classifier
        TP,FP,FN = evaluate_predictions(predictions,Output_Y,Label_X)
```

The retrieved True Positive, False Positive and False Negative values were then passed through the function below to get a score with regards to the models performance.

```python
    #F1 Metric that is used by the competition.
    def F1_Metrics(TP,FP,FN):
        if (TP+FP) == 0:
            p = 0
        else:
            p = (TP/(TP+FP))
        if (TP+FN) == 0:
            r = 0
        else:
            r = (TP/(TP+FN))
        F1 = 2*((p*r)/(p+r))
        return F1
    
    # Takes an input of True Positive, False Positive and False Negatives and returns the F1-Score.
    F1 = F1_Metrics(TP,FP,FN)
```

The predicted answers were then written on a CSV file and submitted to the online portal for ranking.

## Results
It was noticed during training that the algorithm had an extremely higher Root Mean Square Error for the Y Regressor than the X Regressor. This meant that it would be more inaccurate in predicting the Latitude compared to the Longitude.  
The Classification model achieved a greater F1-Score compared to that of the Regression model. This meant that generalizing the output to a selected region would yield better prediction results.  
Highest F1-Scores : Regression = 0.85458 , Classification = 0.88976  
Even though the Classifier achieved better results the Regressor would be more useful in application

## Improvements
More features can be explored such as the Haversine Formula to account for the curvature of the earth.  
External data such as road directions, speed limits, public transportation routes, car parks, workplace and school locations could be used to further improve trajectory prediction.  
The Regression model returns a cartesian coordinate, this coordinate could possibly land where there is a building. Using the exact known locations of roads that coordinate can be corrected to the closest road using map matching algorithms.

<img src="../images/eynextwave/mapmatching.png" alt="linearly separable data"  class="center">
