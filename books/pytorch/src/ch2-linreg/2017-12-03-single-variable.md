---
layout: bookpost
title: Single Variable Linear Regression
date: 2017-12-03
categories: PyTorch
isEditable: true
editPath: books/pytorch/src/ch2-linreg/2017-12-03-single-variable.md
subscribeName: PyTorch
---

# Single Variable Regression

Since this is the very first tutorial in this guide and no knowledge is assumed about machine learning or PyTorch, this tutorial is a bit on the long side. This tutorial will give you an overview of how to do machine learning work in general, a mathematical understanding of single variable linear regression, and how to implement it in PyTorch. If you already feel comfortable with the mathematical concept of linear regression, feel free to skip to the PyTorch [implementation](#implementation).

## Motivation

Single variable linear regression is one of the fundamental tools for any interpretation of data. Using linear regression, we can predict continuous variable outcomes given some data, if the data has a roughly linear shape, i.e. it generally has the shape a line. For example, consider the plot below of 2015 US homicide deaths per age[^fn1], and the line of best fit next to it.

Original data              |  Result of single variable linear regression
:------------------------------:|:-------------------------:
![Homicide Plot][homicide] | ![Homicide Regression Plot][homicide_fit]

Visually, it appears that this data is approximated pretty well by a "line of best fit". This is certainly not the only way to approximate this data, but for now it's pretty good. Single variable linear regression is the tool to find this line of best fit. The line of best fit can then be used to guess how many homicide deaths there would be for ages we don't have data on. By the end of this tutorial you can run linear regression on this homicide data, and in fact solve any single variable regression problem.

## Theory

Since we don't have any theory yet to understand linear regression, first we need to develop the theory necessary to program it.

### Data set format

For regression problems, the goal is to predict a continuous variable output, given some input variables (also called **features**). For single variable regression, we only have one input variable, called \\(x\\), and our *desired* output \\(y\\). Our data set \\(D\\) then consists of many examples of \\(x\\) and \\(y\\), so:
\\[
    D = \\{ (x_1, y_1), (x_2, y_2), \\cdots, (x_m, y_m) \\}
\\]
where \\(m\\) is the number of examples in the data set. For a concrete example, the homicide data set plotted above looks like:
\\[
    D = \\{ (21, 652), (22, 633), \\cdots, (50, 197) \\}
\\]
We will write code to load data sets from files later.

### Model concept

So, how can we mathematically model single linear regression? Since the goal is to find the perfect line, let's start by defining the **model** (the mathematical description of how predictions will be created) as a line:
\\[
    y'(x, a, b) = ax + b
\\]
where \\(x\\) is an input, \\(y'\\) is the prediction for the input \\(x\\), and \\(a\\) and \\(b\\) are **model parameters**. Note that although this is an equation for a line with \\(x\\) as the variable, the values of \\(a\\) and \\(b\\) determine what specific line it is. To find the best line, we just need to find the best values for \\(a\\) (the slope) and \\(b\\) (the y-intercept). For example, the line of best fit for the homicide data above has a slope of about \\(a \\approx -17.69\\) and a y-intercept of \\(b \\approx 1000\\). How we find the magic best values for \\(a\\) and \\(b\\) we don't know yet, but once we find them, prediction is easy, since we just use the formula above.

So, how do we find the correct values of the model parameters \\(a\\) and \\(b\\)? First, we need a way to define what the "best line" is exactly. To do so, we define a **loss function** (also called a cost function), which measures how bad a particular choice of \\(a\\) and \\(b\\) are. Values of \\(a\\) and \\(b\\) that seem poor (a line that does not fit the data set) should result in a large value of the loss function, whereas good values of \\(a\\) and \\(b\\) (a line that fits the data set well) should result in small values of the loss function. In other words, the loss function should measure how far the predicted line is from each of the data points, and add this value up for all data points. We can write this as:
\\[
    L(a, b) = \\sum_{i=1}^m (y'(x_i, a, b) - y_i)^2
\\]
Recall that there are \\(m\\) examples in the data set, \\(x_i\\) is the i'th input, and \\(y_i\\) is the i'th desired output. So, \\((y'(x_i, a, b) - y_i)^2\\) measures how far the i'th prediction is from the i'th desired output. For example, if the prediction \\(y'\\) is 7, and the correct output \\(y\\) is 10, then we would get \\((7 - 10)^2 = 9.\\) Squaring it is important so that it is always positive.  Finally, we just add up all of these individual losses. Since the smallest possible values for the squared terms indicate that the line fits the data as closely as possible, the line of best fit (determined by the choice of \\(a\\) and \\(b\\)) occurs exactly at the smallest value of \\(L(a, b)\\). For this reason, the model is also called [least squares regression](https://en.wikipedia.org/wiki/Least_squares).

> **Note:** The choice to square \\(y'(x_i, a, b) - y_i\\) is somewhat arbitrary. Though we need to make it positive, we could achieve this in many ways, such as taking the absolute value. In a sense, the choice of models and loss functions is one of the creative aspect of machine learning, and often a certain loss function is chosen simply because it produces satisfying results. Manipulating the loss function to achieve more satisfying results will be done in a later chapter.
> However, there is also a concrete mathematical reason for why we specifically square it. It's too much to go into here, but it will be explored in [bonus chapter 2.8](/books/pytorch/book/ch2-linreg/mle.html)

Creating loss functions (and this exact loss function) will continue to be used throughout this guide, from the most simple to more complex models.

### Optimizing the model

At this point, we have fully defined both our model:
\\[
    y'(x, a, b) = ax + b
\\]
and our loss function, into which we can substitute the model:
\\[
    L(a, b) = \\sum_{i=1}^m (y'(x_i, a, b) - y_i)^2 = \\sum_{i=1}^m (a x_i + b - y_i)^2
\\]
We crafted \\(L(a, b)\\) so that it is smallest exactly when each predicted \\(y'\\) is as close as possible to actual data \\(y\\). When this happens, since the distance between the data points and predicted line is as small as possible, using \\(a\\) and \\(b\\) produces the line of best fit. Therefore, our goal is to find the values of \\(a\\) and \\(b\\) that minimize the function \\(L(a, b)\\). But what does \\(L\\) really look like? Well, it is essentially a 3D parabola which looks like:

![Minimum Plot][minimum]

The red dot marked on the plot of \\(L\\) shows where the desired minimum is. We need an algorithm to find this minimum. From calculus, we know that at the minimum \\(L\\) must be entirely flat, that is the derivatives are both \\(0\\):
\\[
    \\frac{\\partial L}{\\partial a} = \\sum_{i=1}^m 2(ax_i + b - y_i)x_i = 0 \\\\
    \\frac{\\partial L}{\\partial b} = \\sum_{i=1}^m 2(ax_i + b - y_i) = 0 \\
\\]
If you need to review this aspect of calculus, I would recommend [Khan Academy videos](https://www.khanacademy.org/math/differential-calculus/analyzing-func-with-calc-dc). Now, for this problem it is possible to solve for \\(a\\) and \\(b\\) using the equations above, like we would in a typical calculus course. But for more advanced machine learning this is impossible, so instead we will learn to use an algorithm called **[gradient descent](https://en.wikipedia.org/wiki/Gradient_descent)** to find the minimum. The idea is intuitive: place a ball at an arbitrary location on the surface of \\(L\\), and it will naturally roll downhill towards the flat valley of \\(L\\) and thus find the minimum. We know the direction of "downhill" at any location since we know the derivatives of \\(L\\): the derivatives are the direction of greatest upward slope (this is known as the [gradient](https://en.wikipedia.org/wiki/Gradient)), so the opposite (negative) derivatives are the most downhill direction. Therefore, if the ball is currently at location \\((a, b)\\), we can see where it would go by moving it to location \\((a', b')\\) like so:
\\[
    a' = a - \\alpha \\frac{\\partial L}{\\partial a} \\\\
    b' = b - \\alpha \\frac{\\partial L}{\\partial b} \\\\
\\]
where \\(\\alpha\\) is a constant called the **learning rate**, which we will talk about more later. If we repeat this process then the ball will continue to roll downhill into the minimum. An animation of this process looks like:

![Gradient Descent Animation][descent_fast]

When we run the gradient descent algorithm for long enough, then it will find the optimal location for \\((a, b)\\). Once we have the optimal values of \\(a\\) and \\(b\\), then that's it, we can just use them to predict a rate of homicide deaths given any age, using the model:
\\[
    y'(x) = ax + b
\\]

## Implementation

Let's quickly review what we did when defining the theory of linear regression:

1. Describe the data set
2. Define the model
3. Define the loss function
4. Run the gradient descent optimization algorithm
5. Use the optimal model to make predictions
6. Profit!

When coding this we will follow the exact same steps. So, create a new file `single_var_reg.py` in the text editor or IDE of your choice (or experiment in the Python REPL by typing `python` at command line), and download the [homicide death rate data set][data] into the same directory.

### Importing the data

First, we need to import all the modules we will need:

```python
import pandas as pd
import matplotlib.pyplot as plt
import torch
import torch.optim as optim
```

We use Pandas to easily load the CSV homicide data, and convert them to a PyTorch `tensor`:

```python
D = pd.read_csv("homicide.csv")
x_dataset = torch.tensor(D.age.values, dtype=torch.float)
y_dataset = torch.tensor(D.num_homicide_deaths.values, dtype=torch.float)
```

Note that `x_dataset` and `y_dataset` are *not* single numbers, but are actually what are called tensors. Roughly, a tensor is just an \\(n\\) dimensional grid of numbers, so a 1-tensor is a [vector](https://en.wikipedia.org/wiki/Vector_space), i.e. a single list of numbers. Both `x_dataset` and `y_dataset` are 1-tensors (vectors). The vectors are each 30 numbers long, since there are 30 data points in the CSV file. So, `(x_dataset[0], y_dataset[0])` would be \\((x_1, y_1) = (21, 652)\\). When we look at multi variable regression later, we will have to work much more with matrices and linear algebra, but for now you only need to worry about vectors.

Whenever possible, I would recommend plotting data, since this helps you verify that you loaded the data set correctly and gain visual intuition about the shape of the data. This is also pretty easy using matplotlib:

```python
plt.plot(x_dataset.numpy(), y_dataset.numpy(), 'x') # The 'x' means that data points will be marked with an x
plt.xlabel('Age')
plt.ylabel('US Homicide Deaths in 2015')
plt.title('Relationship between age and homicide deaths in the US')
plt.show()
```

To plot the data in the PyTorch tensors, we need to convert them to NumPy arrays (since that is what matplotlib expects). We achieve this with `.numpy()`. If all goes well, the plot should look like this:

![Homicide Plot][homicide]

You need to close the plot for your code to continue executing.

### Defining the model

We have our data prepared and plotted, so now we need to define our model. Recall that the model equation is:
\\[
    y' = ax + b
\\]
Before, we thought of \\(x\\) and \\(y'\\) as single numbers. However, we just loaded our data set as vectors (lists of numbers), so it will be much more convenient to define our model using vectors instead of single numbers. If we use the convention that \\(x\\) and \\(y'\\) are vectors, then we don't need to change the equation, just our interpretation of it. Multiplying the vector \\(x\\) by the single number \\(a\\) just multiplies every number in \\(x\\) by \\(a\\), and likewise for adding \\(b\\). So, the above equation interpreted using vectors is the same thing as:
\\[
    \\begin{bmatrix}
           y_{1}', &
           y_{2}', &
           \\dots, &
           y_{m}'
    \\end{bmatrix} = \\begin{bmatrix}
           ax_{1} + b, &
           ax_{2} + b, &
           \\dots, &
           ax_{m} + b
         \\end{bmatrix}
\\]

Fortunately, PyTorch does the work for us of interpreting the simple equation \\(y' = ax + b\\) as the more complicated looking vector equation. First, we define our model parameters \\(a\\) and \\(b\\):

```python
a = torch.randn(1, requires_grad=True)
b = torch.randn(1, requires_grad=True)
```

This says that we create `a` and `b` to be PyTorch **trainable variables**. When we define the model as \\(y' = ax + b\\) there is a crucial point that \\(a\\) and \\(b\\) are the variables which we must update during training, while \\(x\\) is not updated. This means that we want PyTorch to compute the gradients of the loss function with respect to \\(a\\) and \\(b\\), but not with respect to \\(x\\). We have to tell PyTorch to keep track of gradients for `a` and `b` by setting `requires_grad=True`. *In short, make sure you use `requires_grad=True` for any variable that you want to be updated during training.*

Lastly, the function `torch.randn(1, ...)` creates `a` (and `b`) to be a vector of length 1 (i.e. just a number), with the initial value chosen randomly from the [normal distribution](https://en.wikipedia.org/wiki/Normal_distribution) (the precise initial values don't matter).

With the model parameters defined, we can now define \\(y'\\), the output of our model:

```python
def model(x_input):
    return a * x_input + b
```

Note that `model` is a Python function, which will take as input a vector `x_input`, multiply it by the single number `a` and add on the single number `b`. This is where PyTorch does the work for us of reinterpreting our model equation to work using vectors. Finally, the result that is computed is a PyTorch tensor (in this case a vector).

And that's it to define the model!

### Defining the loss function

We have the model defined, so now we need to define the loss function. Recall that the loss function is how the model is evaluated (smaller loss values are better), and it is also the function that we need to minimize in terms of \\(a\\) and \\(b\\). Previously we said the loss function was:
\\[
    L(a, b) = \\sum_{i=1}^m (y'(x_i, a, b) - y_i)^2
\\]

However, \\(y'\\) and \\(y\\) are now being interpreted as vectors. We can rewrite the loss function as:
\\[
    L(a, b) = \\mathrm{sum}((y' - y)^2)
\\]

Note that since \\(y'\\) and \\(y\\) are vectors, \\(y' - y\\) is also a vector that just contains every number stored in \\(y'\\) minus every corresponding number in \\(y'\\). Likewise, \\((y' - y)^2\\) is also a vector, with every number individually squared.  Then, the \\(\\mathrm{sum}\\) function (which I just made up) adds up every number stored in the vector \\((y' - y)^2\\). This is the same as the original loss function, but is a vector interpretation of it instead. We can code this directly as a Python function:

```python
def loss(y_predicted, y_target):
    return ((y_predicted - y_target)**2).sum()
```

The `.sum()` function is an operation which adds up all the numbers stored in a vector. With just these two lines of code we have defined our loss function.

### Minimizing the loss function with gradient descent

With our model and loss function defined, we are now ready to use the gradient descent algorithm to minimize the loss function, and thus find the optimal \\(a\\) and \\(b\\). Fortunately, PyTorch has already implemented the gradient descent algorithm for us, we just need to use it. The algorithm acts almost like a ball rolling downhill into the minimum of the function, but it does so in discrete time steps. PyTorch does not handle this aspect, we need to be responsible for performing each time step of gradient descent. So, roughly in pseudo-code we want to do this:

```python
for t in range(10000):
    # Tell PyTorch to do 1 time step of gradient descent
```

We can't do this yet, since we don't yet have a way to tell PyTorch to perform 1 time step of gradient descent. To do so, we create an optimizer with a learning rate (\\(\\alpha)\\) of \\(0.2\\):

```python
optimizer = optim.Adam([a, b], lr=0.2)
```

The `optim.Adam` optimizer knows how to perform the gradient descent algorithm for us (actually a faster version of gradient descent). Note that this *does not yet minimize \\(L\\)*. This code only create an optimizer object which we will use to minimize \\(L\\). Note that we indicate which variables we want the optimizer to optimize (that is, modify). For an explanation of the `lr=0.2` parameter (and the `10000` loop iterations), see the end of this tutorial.

Using our new `optimizer`object we are ready to write the optimization loop pseudo-code that we originally wanted. Let's look at the code first, and then break it down:

```python
for t in range(10000):
    optimizer.zero_grad() # 1.
    y_predicted = model(x_dataset) # 2.
    current_loss = loss(y_predicted, y_dataset) # 3.
    current_loss.backward() # 4.
    optimizer.step() # 5.
    print(f"t = {t}, loss = {current_loss}, a = {a}, b = {b}") # 6.
```

Let's walk through each of the 6 steps to understand all that is happening here:

1. Under the hood, PyTorch keeps track of a gradient for each variable and step of the model and loss computation. The first thing we do is set all of these stored gradients to 0, so that we don't reuse any previous, old gradient computations.
2. Using the current values of `a` and `b` we compute the predictions of the model.
3. We then compute the value of our loss function for the predictions we just made.
4. At this point we have the current value of the loss \\(L\\). What we want to do is compute \\(\\frac{\\partial L}{\\partial a}\\) and \\(\\frac{\\partial L}{\\partial b}\\). We ask PyTorch to do this for us using `.backward()`. The name comes from the fact that in order to find the derivatives, PyTorch works "backward", starting with the loss and working back to \\(a\\) and \\(b\\). However, the details of how PyTorch computes it are not that important right now. What matters is that `.backward()` does this desired computation, and stores the results somewhere (you can see the results by doing `a.grad` if you wish).
5. Crucially though, `.backward()` does NOT actually update the values of `a` and `b`. Instead, we ask the `optimizer` to update `a` and `b`, based on the currently computed gradients.
6. Finally we just optionally print out some current info so we can observe the training.

What we want to see from the print statements is that the gradient descent algorithm **converged**, which means that the algorithm stopped making significant progress because it found the minimum location of the loss function. When the last few print outputs look like:

```
t = 9992, loss = 39295.83984375, a = -17.27234649658203, b = 997.331298828125
t = 9993, loss = 39295.8359375, a = -17.272356033325195, b = 997.3316040039062
t = 9994, loss = 39295.84765625, a = -17.27235984802246, b = 997.3319091796875
t = 9995, loss = 39295.828125, a = -17.272371292114258, b = 997.3322143554688
t = 9996, loss = 39295.8359375, a = -17.272380828857422, b = 997.33251953125
t = 9997, loss = 39295.8359375, a = -17.272382736206055, b = 997.3328247070312
t = 9998, loss = 39295.84375, a = -17.272397994995117, b = 997.3331298828125
t = 9999, loss = 39295.82421875, a = -17.272401809692383, b = 997.3334350585938
```

then we can tell that we have achieved convergence, and therefore found the best values of \\(a\\) and \\(b\\).

### Using the trained model to make predictions

At this point we have a fully trained model, and know the best values of \\(a\\) and \\(b\\). In fact, the equation of the line of best fit is just:
\\[
    y' = -17.2711x + 997.285
\\]

The last remaining thing for this tutorial is to plot the predictions of the model on top of a plot of the data. First, we need to create a bunch of input ages that we will predict the homicide rates for. We could use `x_dataset` as the input ages, but it is more interesting to create a new vector of input ages, since then we can predict homicide rates even for ages that were not in the data set. Outside of the training `for` loop, we can use the function `linspace` to create a bunch of evenly spaced values between 20 and 55:

```python
# x_test_data has values similar to [20.0, 20.1, 20.2, ..., 54.9, 55.0]
x_test_data = torch.linspace(20, 55)
```

Then, we can compute the model's prediction:

```python
y_test_prediction = model(x_test_data).detach()
```

Note that we must use `.detach()` to tell PyTorch not to perform gradient calculations for this computation. Finally, we can plot the original data and the line together:

```python
plt.plot(x_dataset.numpy(), y_dataset.numpy(), 'x')
plt.plot(x_test_data.numpy(), y_test_prediction.numpy())
plt.xlabel('Age')
plt.ylabel('US Homicide Deaths in 2015')
plt.title('Age and homicide death linear regression')
plt.show()
```

This yields a plot like:

![Homicide Linear Regression Plot][homicide_fit]

# Concluding Remarks

Through following this post you have learned two main concepts. First, you learned the *general form of supervised machine learning workflows*:

1. Get your data set
2. Define your model (the mechanism for how outputs will be predicted from inputs)
3. Define your loss function
4. Minimize your loss function (usually with a variant of gradient descent, such as `optim.Adam`)
5. Once your loss function is minimized, use your trained model to do cool stuff

Second, you learned how to implement linear regression (following the above workflow) using PyTorch. Let's briefly discuss the above 5 steps, and where to go to improve on them.

## 1. The Data Set

This one is pretty simple: we need data sets that contain both input and output data. However, we need a data set that is large enough to properly train our model. With linear regression this is fairly easy: this data set only had 33 data points, and the results were pretty good. With larger and more complex models that we will look at later, this becomes much more of a challenge.

## 2. Defining the model

For single variable linear regression we used the model \\(y' = ax + b\\). Geometrically, this means that the model can only guess lines. Since the homicide data is roughly in the shape of a line, it worked well for this problem. But there are very few problems that are so simple, so soon we will look at more complex models. One other limitation of the current model is it only accepts one input variable. But if our data set had both age and ethnicity, for example, perhaps we could more accurately predict homicide death rate. We will also discuss soon a more complex model that handles multiple input variables.

## 3. Defining the loss function

For single variable regression, the loss function we used, \\(L = \\sum (y' - y)^2\\), is the standard. However, there are a few considerations: first, this loss functions is suitable for this simple model, but with more advanced models this loss function isn't good enough. We will see why soon. Second, the optimization algorithm converged pretty slowly, needing about \\(10000\\) iterations. One cause is that the surface of the loss function is almost flat in a certain direction (you can see this in the 3D plot of it). Though this isn't inherently a problem with the formula for the loss function, the problem surfaces in the loss function. We will also see how to address this problem soon, and converge much faster.

## 4. Minimizing the loss functions

Recall that we created and used the optimizer like so:

```python
optimizer = optim.Adam([a, b], lr=0.2)
for t in range(10000):
    # Run one step of optimizer
```

You might be wondering what the magic numbers of `lr=0.2` (\\( \\alpha \\)) and `10000` are.  Let's start with the learning rate. In each iteration, gradient descent (and variants of it) take one small step that is determined by the derivative of the loss function. The learning rate is just the relative size of the step. So to take larger steps, we can use a larger learning rate. A larger learning rate can help us to converge more quickly, since we cover more distance in each iteration. But a learning rate too large can cause gradient descent to diverge that is, it won't reliably find the minimum.

So, once you have chosen a learning rate, then you need to run the optimizer for enough iterations so it actually converges to the minimum. The easiest way to make sure it runs long enough is just to monitor the value of the loss function, as we did in this tutorial.

Lastly, we didn't use normal gradient descent for optimization in this tutorial. Instead we used `optim.Adam`. With small scale problems like this, there isn't much of a qualitative difference to intuit. In general the [Adam optimizer](https://medium.com/@nishantnikhil/adam-optimizer-notes-ddac4fd7218) is faster, smarter, and more reliable than vanilla gradient descent, but this comes into play a lot more with harder problems.

## 5. Use the trained model

Technically, using the trained model is the easiest part of machine learning: with the best parameters \\(a\\) and \\(b\\), you can simply plug new age values into \\(x\\) to predict new homicide rates. However, trusting that these predictions are correct is another matter entirely. Later in this guide we can look at various statistical techniques that can help determine how much we can trust a trained model, but for now consider some oddities with our trained homicide rate model.

One rather weird thing is that it accepts negative ages: according to the model, 1083 people who are -5 years old die from homicide every year in the US. Now, clearly this makes no sense since people don't have negative ages. So perhaps we should only let the model be valid for people with positives ages. Ok, so then 980 people who are 1 year old die from homicide every year. While this isn't impossible, it does seem pretty high compared to the known data of 652 for 21 year olds. It might seem possible (likely even) that fewer homicides occur for 1 year olds than 21 year olds: but we don't have the data for that, and even if we did, our model could not predict it correctly since it only models straight lines. Without more data, we have no basis to conclude that the number of \\(1\\) year old homicides is even close to 980.

While this might seem like a simple observation in this case, this problem manifests itself continually in machine learning, causing a variety of ethical problems. For example, in 2016 Microsoft released a chatbot on Twitter and [it quickly learned to say fairly horrible and racist things](https://www.theverge.com/2016/3/24/11297050/tay-microsoft-chatbot-racist). More seriously, machine learning is now being used to predict and guide police in cracking down on crime. While the concept might be well-intentioned, the results are despicable, as shown in [an article by The Conversation](https://theconversation.com/why-big-data-analysis-of-police-activity-is-inherently-biased-72640):

> Our recent study, by Human Rights Data Analysis Group’s Kristian Lum and William Isaac, found that predictive policing vendor PredPol’s purportedly race-neutral algorithm targeted black neighborhoods at roughly twice the rate of white neighborhoods when trained on historical drug crime data from Oakland, California.
> [...]
> But estimates – created from public health surveys and population models – suggest illicit drug use in Oakland is roughly equal across racial and income groups. If the algorithm were truly race-neutral, it would spread drug-fighting police attention evenly across the city.

With examples like these, we quickly move from a technical discussion about machine learning to a discussion about ethics. While the study of machine learning is traditionally heavily theoretical, I strongly believe that to effectively and *fairly* apply machine learning in society, we must spend significant effort evaluating the ethics of machine learning models.

This is an open question, and one that I certainly don't have an answer to right now. For the short term we can focus on the problem of not trusting a simple linear regression model to properly predict data outside of what it has been trained on, while in the long term keeping in mind that "with great power comes great responsibility."

# Challenge Problems

Feel free to complete as many of these as you wish, to get more practice with single variable linear regression. Note that for different problems you might have to adjust the learning rate and / or the number of training iterations.

1. Learn how to use PyTorch to generate an artificial data set that is appropriate for single variable linear regression, and then train a model on it. As a hint, for any \\(x\\) value you could create an artificial \\(y\\) value like so: \\(y = ax + b + \\epsilon \\), where \\(\\epsilon\\) is a random number that isn't too big, and \\(a\\) and \\(b\\) are fixed constants of your choice. If done correctly, your trained model should learn by itself the numbers you chose for \\(a\\) and \\(b\\).
2. Run single variable linear regression on a data set of your choice. You can look at [my list of regression data sets](https://donaldpinckney.com/ml.html#regression) for ideas, you can search [Kaggle](https://www.kaggle.com/datasets), or you can search online, such as I did for the homicide data set. Many data sets might have multiple input variables, and right now you only know how to do single variable linear regression. We will deal with multiple variables soon, but for now you can always use only 1 of the input variables and ignore the rest.
3. Experiment with altering the loss function, and observe the effects on the trained model. For example, you could change \\((y' - y)^2\\) to \\(\\mid y' - y \\mid \\) (you will need to lookup PyTorch documentation for the corresponding PyTorch functions), or really anything you can think of.

# Complete Code

The [complete example code is available on GitHub](https://github.com/donald-pinckney/donald-pinckney.github.io/blob/src/books/pytorch/src/ch2-linreg/code/single_var_reg/single_var_reg.py), as well as directly here:

```python
import pandas as pd
import matplotlib.pyplot as plt
import torch
import torch.optim as optim

# Load the data
D = pd.read_csv("homicide.csv")
x_dataset = torch.tensor(D.age.values, dtype=torch.float)
y_dataset = torch.tensor(D.num_homicide_deaths.values, dtype=torch.float)

# Plot the data
plt.plot(x_dataset.numpy(), y_dataset.numpy(), 'x') # The 'x' means that data points will be marked with an x
plt.xlabel('Age')
plt.ylabel('US Homicide Deaths in 2015')
plt.title('Relationship between age and homicide deaths in the US')
plt.show()



### Model definition ###

# First we define the trainable parameters a and b 
a = torch.randn(1, requires_grad=True) # requires_grad means it is trainable
b = torch.randn(1, requires_grad=True)

# Then we define the prediction model
def model(x_input):
    return a * x_input + b


### Loss function definition ###

def loss(y_predicted, y_target):
    return ((y_predicted - y_target)**2).sum()

### Training the model ###

# Setup the optimizer object, so it optimizes a and b.
optimizer = optim.Adam([a, b], lr=0.2)

# Main optimization loop
for t in range(10000):
    # Set the gradients to 0.
    optimizer.zero_grad()
    # Compute the current predicted y's from x_dataset
    y_predicted = model(x_dataset)
    # See how far off the prediction is
    current_loss = loss(y_predicted, y_dataset)
    # Compute the gradient of the loss with respect to a and b.
    current_loss.backward()
    # Update a and b accordingly.
    optimizer.step()
    print(f"t = {t}, loss = {current_loss}, a = {a.item()}, b = {b.item()}")


### Using the trained model to make predictions ###

# x_test_data has values similar to [20.0, 20.1, 20.2, ..., 54.9, 55.0]
x_test_data = torch.linspace(20, 55)
# Predict the homicide rate for each age in x_test_data
# .detach() tells PyTorch to not find gradients for this computation.
y_test_prediction = model(x_test_data).detach()

# Plot the original data and the prediction line
plt.plot(x_dataset.numpy(), y_dataset.numpy(), 'x')
plt.plot(x_test_data.numpy(), y_test_prediction.numpy())
plt.xlabel('Age')
plt.ylabel('US Homicide Deaths in 2015')
plt.title('Age and homicide death linear regression')
plt.show()
```

# References

[^fn1]: Centers for Disease Control and Prevention, National Center for Health Statistics. Underlying Cause of Death 1999-2015 on CDC WONDER Online Database, released December, 2016. Data are from the Multiple Cause of Death Files, 1999-2015, as compiled from data provided by the 57 vital statistics jurisdictions through the Vital Statistics Cooperative Program. Accessed at [https://wonder.cdc.gov/ucd-icd10.html](https://wonder.cdc.gov/ucd-icd10.html) on Nov 22, 2017 2:18:46 PM.

[homicide]: /books/pytorch/book/ch2-linreg/assets/homicide.png
[homicide_fit]: /books/pytorch/book/ch2-linreg/assets/homicide_fit.png
[minimum]: /books/pytorch/book/ch2-linreg/assets/minimum.png
[data]: /books/pytorch/book/ch2-linreg/code/homicide.csv
[descent_fast]: /books/pytorch/book/ch2-linreg/assets/descent_fast.gif
