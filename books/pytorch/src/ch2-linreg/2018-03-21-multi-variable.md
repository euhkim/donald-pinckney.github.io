---
layout: bookpost
title: Multi Variable Linear Regression
date: 2018-03-21
categories: PyTorch
isEditable: true
editPath: books/pytorch/src/ch2-linreg/2018-03-21-multi-variable.md
subscribeName: PyTorch
---

# Multi Variable Regression

In [chapter 2.1](/books/pytorch/book/ch2-linreg/2017-12-03-single-variable.html) we learned the basics of PyTorch by creating a single variable linear regression model. In this chapter we expand this model to handle multiple variables. Note that less time will be spent explaining the basics of PyTorch: only new concepts will be explained, so feel free to refer to previous chapters as needed.

## Motivation

Recall that a single variable linear regression model can learn to predict an output variable \\(y\\) under these conditions:

1. There is only one input variable, \\(x\\)
2. There is a linear relationship between \\(y\\) and \\(x\\), that is, \\(y \\approx ax + b\\)

In practice, the above conditions are very limiting: if you have a simple data set then by all means you should try using single variable linear regression, but in most cases we have significantly more complex data. For example, consider using the following (abbreviated) [data from the 1990 census](https://www.kaggle.com/camnugent/california-housing-prices) to learn to predict housing prices. Note that each row represents a single housing district:

| House Median Age     | Total Rooms | Total Bedrooms | ... | Median House Value |
|-------------|------|-------------|------------|-----------------|
| 41.0 | 880.0  | 129.0  | ...     | 452600.0    |
| 21.0 | 7099.0 | 1106.0 | ...     | 358500.0    |
| 52.0 | 1467.0 | 190.0  | ...     | 352100.0    |
| 52.0 | 1274.0 | 235.0  | ...     | 341300.0    |
| 52.0 | 1627.0 | 280.0  | ...     | 342200.0    |
| 52.0 | 919.0  | 213.0  | ...     | 269700.0    |
| ...  | ...    | ...    | ...     | ...         |

To predict the values of houses, we have at least 3 real-valued variables (age, number of rooms, number of dedrooms) that could potentially be useful. To analyze this sort of complex, real-world data we need to learn to handle multiple input variables.

One approach to handling multiple variables would be to reduce the number of input variables to only 1 variable, and then training a single variable linear regression model using that. In fact, an important area of research in machine learning (and one that will be covered later) called **[dimensionality reduction](https://en.wikipedia.org/wiki/dimensionality_reduction)** deals with this problem of reducing the number of variables. However, it's important to realize that the number of variables can only be reduced so far, and its extremely rare that you can reduce a data set to only 1 variable. For now you need to take this statement on faith, but in later chapters we will investigate it more thoroughly.

So, it seems that we will have to deal with training models that can handle multiple variables. In this chapter we learn how to allow multiple input variables in our linear regression model. Such a model is called multi variable linear regression, or just linear regression.

## Theory

Most of the theory is similar to the theory for single variable linear regression, but we will need to augment and generalize it to handle multiple variables.

### Data set format

Previously we defined our data set \\(D\\) as consisting of many example pairs of \\(x\\) and \\(y\\), where \\(m\\) is the number of examples:
\\[
    D = \\{ (x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}), \\cdots, (x^{(m)}, y^{(m))} \\}
\\]

Note that I have changed the notation compared to before. The notation \\(x^{(i)}\\) refers to the \\(i\\)'th \\(x\\) training example, it does *NOT* mean \\(x\\) to the \\(i\\)'th power, which would be written as \\(x^i\\). I promise the notation change will be useful shortly.

Alternatively, we can write \\(D\\) as 2 vectors of shape 1 x \\(m\\):
\\[
    D_x = \\begin{bmatrix}
            x^{(1)},
            x^{(2)},
            \\dots,
            x^{(m)}
    \\end{bmatrix} \\\\
    D_y = \\begin{bmatrix}
            y^{(1)},
            y^{(2)},
            \\dots,
            y^{(m)}
         \\end{bmatrix}
\\]

But now, we need each \\(x^{(i)}\\) example to contain multiple numbers, one for each input variable.  Let \\(n\\) be the number of input variables. Then the easiest way to write this is to let each \\(x^{(i)}\\) be a vector of shape \\(n\\) x 1. That is,
\\[
    x^{(i)} = \\begin{bmatrix}
        x^{(i)}_1 \\\\
        x^{(i)}_2 \\\\
        \\vdots \\\\
        x^{(i)}_j \\\\
        \\vdots \\\\
        x^{(i)}_n
    \\end{bmatrix}
\\]
Note that the notation \\(x^{(i)}_j\\) denotes the \\(j\\)'th input variable in the \\(i\\)'th example data.

Since each \\(x^{(i)}\\) has \\(n\\) rows, and \\(D_x\\) has \\(m\\) columns, each of which is an \\(x^{(i)}\\), we can write \\(D_x\\) as a massive \\(n \\times m\\) matrix:
\\[
    D_x = \\begin{bmatrix}
            x^{(1)},
            x^{(2)},
            \\dots,
            x^{(m)} \\end{bmatrix}
        = \\begin{bmatrix}
            x^{(1)}_1 & x^{(2)}_1  & \\dots & x^{(i)}_1 & \\dots & x^{(m)}_1 \\\\
            x^{(1)}_2 & x^{(2)}_2  & \\dots & x^{(i)}_2 & \\dots & x^{(m)}_2 \\\\
            \\vdots & \\vdots  & \\ddots & \\vdots & \\ddots & \\vdots \\\\
            x^{(1)}_j & x^{(2)}_j  & \\dots & x^{(i)}_j & \\dots & x^{(m)}_j \\\\
            \\vdots & \\vdots  & \\ddots & \\vdots & \\ddots & \\vdots \\\\
            x^{(1)}_n & x^{(2)}_n  & \\dots & x^{(i)}_n & \\dots & x^{(m)}_n \\\\
        \\end{bmatrix}
\\]
So, each column of \\(D_x\\) represents a single input data example. We don't need to change the 1 x \\(m\\) vector \\(D_y\\), since we still only have 1 output variable.

### Model concept

So, we now have an input data matrix \\(D_x\\) with each column vector representing a single input data example, and we have the corresponding \\(D_y\\) row vector, each entry of which is an output data example. How do we define a model which can linearly estimate the output \\(y'^{(i)}\\) given the input data vector \\(x^{(i)}\\)? Let's build it up from simple concepts, and build towards more complex linear algebra.

Since we want \\(y'^{(i)}\\) to depend linearly on each \\(x^{(i)}_j\\) for \\(1 \\leq j \\leq n\\), we can write:
\\[
    y'^{(i)} = a_1 x^{(i)}_1 + a_2 x^{(i)}_2 + \\cdots + a_j x^{(i)}_j + \\cdots + a_n x^{(i)}_n + b
\\]

This is fine mathematically, but it's not very general. Suppose \\(n = 100\\): then we would have to literally write out 100 terms in our PyTorch code. We can generalize this using linear algebra. Let \\(A\\) be a row vector of shape 1 x \\(n\\), containing each \\(a_j\\):
\\[
    A = \\begin{bmatrix}
            a_1,
            a_2,
            \\cdots,
            a_j,
            \\cdots,
            a_n
    \\end{bmatrix}
\\]

Now, let's see what happens if we compute \\(A x^{(i)}\\), as matrix multiplication. Note that \\(A\\) has shape 1 x \\(n\\) and \\(x^{(i)}\\) has shape \\(n\\) x 1. This is perfect! When performing matrix multiplication, the inner dimensions (in this case \\(n\\) and \\(n\\)) have to match, and the outer dimensions (in this case \\(1\\) and \\(1\\)) determine the output shape of the multiplication. So \\(A x^{(i)}\\) will have shape 1 x 1, or in other words, just a single number, in fact it is exactly \\(y'^{(i)}\\). How does this matrix multiplication exactly work? I'll refer you to [this video by Khan Academy](https://www.khanacademy.org/math/precalculus/precalc-matrices/multiplying-matrices-by-matrices/v/matrix-multiplication-intro), and explain it briefly in this case. Here, it is easier since \\(A\\) is a row vector, and \\(x^{(i)}\\) is a column vector. We simply multiply each corresponding entry, and add it all up:
\\[
    A x^{(i)} + b
    = \\begin{bmatrix}
            a_1,
            a_2,
            \\cdots,
            a_j,
            \\cdots,
            a_n
    \\end{bmatrix} \\begin{bmatrix}
        x^{(i)}_1 \\\\
        x^{(i)}_2 \\\\
        \\vdots \\\\
        x^{(i)}_j \\\\
        \\vdots \\\\
        x^{(i)}_n
    \\end{bmatrix} + b
    = a_1 x^{(i)}_1 + a_2 x^{(i)}_2 + \\cdots + a_j x^{(i)}_j + \\cdots + a_n x^{(i)}_n + b
    = y'^{(i)}
\\]

This matrix equation, \\(y'(x, A, b) = Ax + b\\) is exactly what we want as our model. As one final note, recall that in the actual implementation, we don't want \\(x\\) and \\(y'\\) to represent just one input data and predicted output, we want them to represent several. Since \\(x\\) is a column vector, the natural way to represent multiple input data points is with a matrix, very similar to the matrix \\(D_x\\), just not necessarily with *all* the columns of \\(D_x\\), and \\(y'\\) should be a row vector. Specifically, \\(A\\) has shape 1 x \\(n\\), \\(x\\) has shape \\(n\\) x \\(m\\), and \\(y\\) has shape 1 x \\(m\\), where \\(m\\) is the number of data points.

Now defining the loss function is pretty much the same as before, just using the new model:
\\[
     L(A, b) = \\sum_{i=1}^m (y'(x^{(i)}, A, b) - y^{(i)})^2 = \\sum_{i=1}^m (A x^{(i)} + b - y^{(i)})^2
\\]

To minimize the loss function, we use the same process as before, gradient descent. However, previously the gradient descent was altering 2 variables (\\(a\\) and \\(b\\)) so as to minimize the loss function, and so we could plot the loss function and gradient descent progress in terms of \\(a\\) and \\(b\\). However, now the optimization needs to alter many more variables, since \\(A\\) actually contains \\(n\\) variables, the gradient descent must be performed in \\(n+1\\) dimensional space, and we don't have an easy way to visualize this.

With the more general linear algebra formulation of linear regression under our belts, let's move on to actually coding stuff.

## Implementation

As before, we need to: import data, define the model, define the loss function, run gradient descent, and finally make predictions. Many steps will be similar to the single variable case, but for completeness I will walk through them briefly.

For building and testing the implementation we will use a synthetic data set consisting of \\(n=2\\) input variables. You can download [the synthetic data set here][synthetic-data]. By synthetic, I mean that I purposefully created a very nicely behaved data set so that we can practice implementing multi variable linear regression, and verify that we converged to the right answer. In fact, the synthetic data is generated as \\(y = 2x_1 + 1.3x_2 + 4 + \\varepsilon \\) where \\(\\varepsilon\\) is random noise. If we implement multi variable linear regression correctly, then we should obtain approximately \\(A = \\begin{bmatrix} 2, 1.3 \\end{bmatrix}, b = 4\\). This plot illustrates what the data looks like in 3 dimensions, essentially a plane in 3 dimensions with some random fluctuations:

![scatter][scatter]

### Importing the data

As explained above, the input data set can be organized as an \\(n \\times m\\) matrix. Since we will load the entire data set (input and output) from a single CSV file, and we have 2 input variables, the CSV file will contain 3 columns: the first 2 are the input variables, and the last one is the output variable. So, first we load the CSV file into an \\(m\\) x 3 matrix, and then separate the first 2 columns from the last:

```python
import pandas as pd
import matplotlib.pyplot as plt
import torch
import torch.optim as optim

### Load the data

# First we load the entire CSV file into an m x 3
D = torch.tensor(pd.read_csv("linreg-multi-synthetic-2.csv", header=None).values, dtype=torch.float)

# We extract all rows and the first 2 columns, and then transpose it
x_dataset = D[:, 0:2].t()

# We extract all rows and the last column, and transpose it
y_dataset = D[:, 2].t()

# And make a convenient variable to remember the number of input columns
n = 2
```

The syntax `D[:, 0:2]` might be new, particularly if you haven't worked with Python much before. In the single variable implementation we used Panda's functionality to access the columns by column name. This is a great approach, but sometimes you might need to be more flexible in how you access columns of data.

> **Note:** The basic syntax for subscripting a matrix is: `D[3, 6]` (for example), which refers to the row at index 3 and the column at index 6 in the matrix `D`. Note that in Python the row and column indices start at 0! This means that `D[0, 0]` refers to the top-left entry of matrix `D`. If you are coming from a pure math background, or have used MATLAB or R before, it is a common error to assume the indices start at 1. <br /><br />
> Now for slicing, the `:` character is used to indicate a range. If it is used by itself, it indicates the entire range of rows / columns. For example, `D[:, 42]` refers to all rows of `D`, and the column at index 42. If it is used with indices, then `i:j` indicates the range of rows / columns at indices `i`, `i+1`, ..., `j-1`, but *not* including `j`. <br /><br />
> So, `D[:, 0:2]` means to read the values in `D` at all rows and at columns with index `0` and `1` (the entire first 2 columns, i.e. the input data columns). Likewise, `D[:, 2]` means to read the values in `D` at all rows and at the column of index `2` (the entire last column, i.e. the output data column).

This matrix subscripting and slicing is almost what we want, but not quite. The problem is that `D[:, 0:2]`, which contains our \\(D_x\\) data, is a matrix of shape \\(m \\times n\\), but earlier we decided that we wanted \\(D_x\\) to be an \\(n \\times m\\) matrix, so we need to flip it. To do so, we use the [**transpose**](https://en.wikipedia.org/wiki/Transpose) of the matrix. Mathematically we write the transpose of a matrix \\(A\\) as \\(A^T\\), and in Python we can compute it using `A.t()`. Essentially, the transpose of a matrix simply flips it along the diagonal, as shown in this animation:

<center>
<p><a href="https://commons.wikimedia.org/wiki/File:Matrix_transpose.gif#/media/File:Matrix_transpose.gif"><img src="https://upload.wikimedia.org/wikipedia/commons/e/e4/Matrix_transpose.gif" alt="Matrix transpose.gif"></a><br>By <a href="//commons.wikimedia.org/wiki/User:LucasVB" title="User:LucasVB">LucasVB</a> - <a href="https://commons.wikimedia.org/w/index.php?curid=21897854">Link</a></p>
</center>

So, `D[:, 0:2].t()` is a matrix of shape \\(n \\times m\\), and is our correct data input matrix \\(D_x\\). We save this matrix to the variable `x_dataset`. Likewise, we also transpose `D[:, 2]` to correctly compute \\(D_y\\), and save it in `y_dataset`.

At this point we have our \\(m \\times n\\) input data matrix `x_dataset` and our \\(m \\times 1\\) output vector `y_dataset` loaded. In addition, we conveniently have the number of columns stored in `n`, so now we can start defining our model.

### Defining the model

As shown above, we want our model parameters to consist of a matrix \\(A\\) of size \\(1 \\times n\\) and a single number \\(b\\). Then, we define:
\\[
    y'(x, A, b) = Ax + b
\\]

In order to implement this, we define the trainable variables, the output prediction, and the loss function:

```python
### Model definition ###

# First we define the trainable parameters A and b 
A = torch.randn((1, n), requires_grad=True)
b = torch.randn(1, requires_grad=True)

# Then we define the prediction model
def model(x_input):
    return A.mm(x_input) + b


### Loss function definition ###

def loss(y_predicted, y_target):
    return ((y_predicted - y_target)**2).sum()
```

There are two differences with this code and the previous single variable regression code. First, we we define `A`, we give it size of `(1, n)` rather than `1`. This means that now `A` is a matrix of size 1 x `n` (i.e. a row vector) rather than a scalar as before. Second, when we define the model, we can no longer write `A * x` because `*` means *scalar multiplication*. Instead, we use `A.mm(x_input)` to indicate *matrix multiplication*.

### Training the model

At this point, we have our model and loss function all setup. The remaining code to train the model is extremely similar to the code for single variable regression, so I'll simply display it here, and then explain the few differences:

```python
### Training the model ###

# Setup the optimizer object, so it optimizes a and b.
optimizer = optim.Adam([A, b], lr=0.1)

# Main optimization loop
for t in range(2000):
    # Set the gradients to 0.
    optimizer.zero_grad()
    # Compute the current predicted y's from x_dataset
    y_predicted = model(x_dataset)
    # See how far off the prediction is
    current_loss = loss(y_predicted, y_dataset)
    # Compute the gradient of the loss with respect to A and b.
    current_loss.backward()
    # Update A and b accordingly.
    optimizer.step()
    print(f"t = {t}, loss = {current_loss}, A = {A.detach().numpy()}, b = {b.item()}")
```

First, we have a different learning rate than the learning rate used in single variable regression. Even though the training algorithm is the same, since this is a different problem than single variable regression, we need find a good learning rate specific to this problem. A great way to do this for your own problems is using TensorBoard, as explained in the chapter [Optimization Convergence](https://donaldpinckney.com/books/pytorch/book/ch2-linreg/2017-12-27-optimization.html).

Besides this, the only other conceptual difference is that at each step of the optimizer we are modifying the entire vector `A` simultaneously (in addition to `b`), rather than just a single number. However, PyTorch abstracts this away for us, and conceptually we just need to know that we are training the variable `A`.

The final print statements should output something close to:

```
t = 1994, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
t = 1995, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
t = 1996, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
t = 1997, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
t = 1998, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
t = 1999, loss = 1447991.625, A = [[2.0054889 1.3021096]], b = 3.950239419937134
```

At this point we have converged to our approximate solution of \\(A \\approx \\begin{bmatrix}
            2.005,
            1.302
    \\end{bmatrix}, b \\approx 3.95\\). Note that this is not exactly the same as the expected answer of \\(A = \\begin{bmatrix}
            2,
            1.3
    \\end{bmatrix}, b \\approx 4\\), primarily because some random noise was added to each point in the data set.

The model is fully trained, so now given a new input \\(x\\) we could now predict the output \\(y' = Ax + b\\), using the learned information from all input variables.

# Concluding Remarks

Linear regression with multiple variables is only slightly different in essence from single variable linear regression. The main difference is abstracting the linear operation \\(ax\\) where \\(a\\) and \\(x\\) are single numbers to the linear operation \\(Ax\\), where now \\(A\\) is a matrix, \\(x\\) is a vector. In addition, at the implementation level we also have to deal with loading data in a more sophisticated manner, but otherwise the code is mostly the same. In later chapters we will use this abstraction we have built to define even more powerful models.

# Challenge Problems

So far this chapter has used a synthetic data set, `linreg-multi-synthetic-2.csv`, for easy demonstration. The exercises are primarily concerned with getting practice at applying this model to real-world data. Note that in real-world data not all columns are useful, and some might not have a linear relationship with the output variable. Including these unhelpful columns in your model might decrease the accuracy of your model. You should try plotting various columns vs. the output column to determine which seem most helpful in predicting the output, and then only include these useful columns as your input.

In addition, many data sets will have so called *messy data*, which require you to do some manipulation in Python to make sure the data is imported cleanly and properly. For example, some rows might containg missing data: for these your code can not crash or incorrectly import the data. Instead, you need to adopt a strategy to still import the data as best as you can: for example, you can simply ignore any rows that have incomplete data.

Note that we have not discussed how to rigorously evaluate how good a model is yet. For now you can use the value of the loss function, along with some intuition and creating plots. Evaluation will be discussed more in chapter 2.7.

1. Download [this red wine quality data set](https://www.kaggle.com/uciml/red-wine-quality-cortez-et-al-2009), and try to predict the quality of the wine (last column) from the physicochemical input data (other columns).
2. Download [this car MPG data set](https://archive.ics.uci.edu/ml/datasets/Auto+MPG), and try to predict the MPG (first column) based on some of the other columns.
3. Download [this California 1990 Housing Value data set](https://www.kaggle.com/camnugent/california-housing-prices), and try to predict the house values based on various factors.

# Complete Code

The [complete example code is available on GitHub](https://github.com/donald-pinckney/donald-pinckney.github.io/blob/src/books/pytorch/src/ch2-linreg/code/multi_var_reg/multi_var_reg.py), as well as directly here:

```python
import pandas as pd
import matplotlib.pyplot as plt
import torch
import torch.optim as optim

### Load the data

# First we load the entire CSV file into an m x 3
D = torch.tensor(pd.read_csv("linreg-multi-synthetic-2.csv", header=None).values, dtype=torch.float)

# We extract all rows and the first 2 columns, and then transpose it
x_dataset = D[:, 0:2].t()

# We extract all rows and the last column, and transpose it
y_dataset = D[:, 2].t()

# And make a convenient variable to remember the number of input columns
n = 2


### Model definition ###

# First we define the trainable parameters A and b 
A = torch.randn((1, n), requires_grad=True)
b = torch.randn(1, requires_grad=True)

# Then we define the prediction model
def model(x_input):
    return A.mm(x_input) + b


### Loss function definition ###

def loss(y_predicted, y_target):
    return ((y_predicted - y_target)**2).sum()

### Training the model ###

# Setup the optimizer object, so it optimizes a and b.
optimizer = optim.Adam([A, b], lr=0.1)

# Main optimization loop
for t in range(2000):
    # Set the gradients to 0.
    optimizer.zero_grad()
    # Compute the current predicted y's from x_dataset
    y_predicted = model(x_dataset)
    # See how far off the prediction is
    current_loss = loss(y_predicted, y_dataset)
    # Compute the gradient of the loss with respect to A and b.
    current_loss.backward()
    # Update A and b accordingly.
    optimizer.step()
    print(f"t = {t}, loss = {current_loss}, A = {A.detach().numpy()}, b = {b.item()}")
```

[synthetic-data]: /books/pytorch/book/ch2-linreg/code/linreg-multi-synthetic-2.csv
[scatter]: /books/pytorch/book/ch2-linreg/assets/linreg-multi-synthetic-2.png
