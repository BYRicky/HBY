---
layout: post
title: 'Linear classifier - perceptron classifier'
tags: ["matlab", "classifier"]
---

Here is an approach to implement simple perception classifier (<font color="#dd0000">only for two linearly separable class</font>). I will show you guys the entire steps for this implementation. <br>

We choose the Iris dataset. This dataset contains 3 classes of 50 instances each, where each class refers to a type of Iris plant. The first two classes are linearly separable from each other, whereas the last class is not linearly separable from each other. We only take the first two linearly separable classes. There are four steps to implement perceptron classifier as follows.

## Step 1: Load dataset <br>
In matlab, we use `textread` to load the dataset with text format. In this dataset, there are four attributes except for the class name and each attribute is separated by comma. Note that in `textread`, the second parameter is the data type of the dataset (in Iris dataset, the first four features are floating point number and the last one is a string), and the fourth one is the delimiter used in the dataset. As the result, we can write the followiong code.
```matlab
[atrb1,atrb2,atrb3,atrb4,className]=textread(filename,'%f%f%f%f%s','delimiter,',');
```
---

## Step 2: Split training data and testing data <br>
Assume that we have 100 instances, and serve 90 samples as training data. The straightforward way is to arbitrarily take 90 instances for training and the rest ones for testing. However, it cannot <font color="#dd0000">gurantee that these two subsets have similar distributions</font>. In matlab, there is a built-in function `cvpartition`; it gives an systemical way to split training and testing data.
```matlab
%use KFold cross vaildation
k=10;
cv=cvpartition(className,'KFold', k);
%the splitting result (1/0 represents training/testing data)
idx=test(cv,1); 
%the ratio of training samples to testing samples 
ratio=(k-1)/k;
%get the number of features except for class name
Tuple={atrb1,atrb2,atrb3,atrb4}
[dataCount,atrb]=size(Tuple);
%initialize array
dataTrain=zeros(floor(length(name)*ratio),atrb);
dataTrainName=cell(floor(length(name)*ratio),1);
dataTest=zeros(length(className)-length(dataTrain),atrb);
dataTestName=cell(length(className)-length(dataTrainName),1);

cnt=1;cnt2=1;
for k=1:length(className)
    if(idx(k)==1)
    	dataTrain(cnt,:)=Tuple(k,:); dataTrainName(cnt,1)=name(k,1); cnt=cnt+1;
    else
        dataTest(cnt2,:)=Tuple(k,:); dataTestName(cnt2,1)=name(k,1); cnt2=cnt2+1;
    end
end
```
---

## Step 3: Feature space <br>
Typically, <font color="#dd0000">each instance is represented by a feature vector (i.e., a row vector [atrb1, atrb2, atrb3, atrb4])</font>, and all feature vectors span the feature space. Each feature is a dimension of the feature space, and therefore Iris dataset forms a 4-dimensional feature space. We reduce the 4D space into multiple 2D spaces for clear visualization. Following the steps below, we can generate the feature space.

1. use `subplot` to display multiple plots in one figure. 
2. choose each two features to form a 2D space.
3. use `gscatter` to create a scatter plot of 2D points, grouped by className.

```matlab
figure(1);
Title={'Speal.Length','Sepal.Width','Petal.Length','Petal.Width'};
for i=1:atrb
  for j=1:atrb
    if(i==j); continue; end
    subplot(atrb,atrb,(i-1)*atrb+j);
    gscatter(Tuple(:,i),Tuple(:,j),className,'rg','.',8);
    xlabel(Title(i)); ylabel(Title(j));
    legend('off');
  end
end
```
---

## Step 4: Establish a perceptron classifier <br>
We give the entire concept of the perceptron classifier as follows. <br>

### 1. What is the perceptron classifier? <br>
It is one of <font color="#dd0000">the linear classifier which has linear decision bounderies.</font> A linear classifier provides multiple linear equations to divide the classes into several regions. <br><br>
Now, we focus on the two-class problem, that is, only one linear equation is generated. Here is an example. Assume the point (2, 2) belongs to class1 whereas the point (0, 0) is an element in class2 and there is a decision boundary 2x + 3y - 4 = 0. 

### 2. How to verify the correctness of this equation? <br>
The point (2, 2) gives the result 2 * 2 + 3 * 2 - 4 > 0 while the point (0, 0) gives the result 2 * 0 + 3 * 0 - 4 < 0. One can find that the point (2, 2) gets the result "> 0" whereas the point (0, 0) gets "< 0" (i.e., they are on the different sides of the boundary). <br><br>
The coefficients 2, 3, and 4 (x, and y) can be viewed as the weight vector W: [w<sub>1</sub>, w<sub>2</sub>, w<sub>3</sub>] (instances X: [x<sub>1</sub>, x<sub>2</sub>]). As the result, the equation 2x + 3y - 4 = 0 can be rewritten as W·X<sup>T</sup>=0, where "·" stands for inner product. However, we want to use this equation to do evaluation, it cannot be assigned to zero. <font color="#dd0000">It should be: W·X<sup>T</sup> > 0 if X is in class1 and W·X<sup>T</sup> < 0 if X is in class2. </font><br>

### 3. X is known information, but how to find the suitable W? <br>
We start from the initial weight W=[0, 0, 0]. <font color="#dd0000">If this evaluation function W·X<sup>T</sup> gives the incorrect result, then W is updated by X.</font> There are two situation. One is that the point in class1 is misclassified into class2, and the new weight vector is [0 + x<sub>1</sub>, 0 + x<sub>2</sub>, 0 + 1]. The other is that the point in class2 is then classified into class1, and the updated weight vector is [0 - x<sub>1</sub>, 0 - x<sub>2</sub>, 0 - 1]. Each misclassified instance leads to one update.

### 4. What is the general form of W and X?
If there are n attributes in one dataset, then X is [x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n</sub>] and W is [w<sub>1</sub>, w<sub>2</sub>, ..., w<sub>n+1</sub>]. You will get n-dimensional decision boundaries.

In conclusion, a perception classifier generates multiple linear decision boundaries. <font color="#dd0000">Each misclassified instance leads to one update (+[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n</sub>, 1] or -[x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n</sub>, 1]).</font> Please refer to the following codes. 

```matlab
correct=0; % the number of correctly classified data
w=[0 0 0 0 0]; %initial weight vector

%to find the value of the weight vector
%all the training data are correctly classified
while(correct~=length(dataTrain))
    for k=1:length(dataTrain)
        %update weight vector if it misclassified a class2 instance
        if(w(1)*dataTrain(k,1)+w(2)*dataTrain(k,2)+w(3)*dataTrain(k,3)+w(4)*dataTrain(k,4)+w(5)>0)
            if(strcmp(dataTrainName(k),'Iris-setosa'))
                w=w-[dataTrain(k,1),dataTrain(k,2),dataTrain(k,3),dataTrain(k,4),1];
                correct=0;
                break;
            else
                correct=correct+1;
            end
        end
        
        %update weight vector if it misclassified a class1 instance
        if(w(1)*dataTrain(k,1)+w(2)*dataTrain(k,2)+w(3)*dataTrain(k,3)+w(4)*dataTrain(k,4)+w(5)<=0)
           if(strcmp(dataTrainName(k),'Iris-versicolor'))
                w=w+[dataTrain(k,1),dataTrain(k,2),dataTrain(k,3),dataTrain(k,4),1];
                correct=0;
                break;
           else
               correct=correct+1;
           end
        end
    end
end
```

After several adjustments for the weight vector, it generated the final decision boundary that is w<sub>1</sub>x + w<sub>2</sub>y + w<sub>3</sub>z + w<sub>4</sub>a + w5 = 0. Some testing data located at one side of this hyperplane are labeled as class1, and the rest ones located at the other side are labeled as class2.

Here is the classification result with 5% of dataset for training and three features involved. The gray plane is the decision boundary. It gives the classification accuracy around to 96%. That is, only 4% of the testing data are misclassified.
<p align="center">
<img src="../img/20190404PR.png" width="80%" hegiht="80%"/>
</p>


