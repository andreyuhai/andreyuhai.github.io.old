---
layout: post
title: "Coursera - Machine Learning - Ex4 - Predicting Your Handwriting"
excerpt: How to predict your own handwriting using multiclass classifiers given in programming exercise 4 of machine learning course by Andrew Ng on coursera.
modified:
categories: articles
tags: [octave, matlab, machine learning, multiclass classifiers]
image:
  feature: 2019-09-25-coursera-machine-learning-predicting-your-handwriting/cover.png 
  credit: Kyle Glenn
  creditlink: https://unsplash.com/photos/kvIAk3J_A1c
comments: true
share: true
published: true
aging: true
---

### Coursera - Machine Learning Predicting Your Handwriting

I've been studying Andrew Ng's [machine learning course][1] on Coursera for a while now. I've just finished 4th week assignment which is about training multiclass classifiers and using already trained neural networks to predict handwritten digits.

The classifiers are trained on 5000 20x20 grayscale images.  

After I've completed the assignment, I was wondering whether I could predict my own handwriting using the trained classifiers.

---

#### How to predict your own handwriting?

To predict your own handwriting successfully, you should make sure you follow these two guidelines below that I've figured out by trial and error and also asking on [discussion forum][2]:

- The size of the digit in the image should not be too big or too small.
- The background pixels should be a consistent shade of gray for almost every pixel or should be as much consistent as possible, not to mention that the digit pixels should be white :).

---

#### How do I make my background a shade of gray and the digit white?
Of course, it is difficult to find a gray paper and a white pen to ensure the guideline above. 
Instead we will use a light colored paper (I've used a yellow post-it) and a black or blue pen, then we will map our pixel values between 0 and 1 (considering the quoted tip below) after [converting our image to grayscale][3] in MATLAB which will result in the background being gray and the digit being white.

I will explain this mapping process in detail in this post as well.

>The image pixels are scaled (or normalized) so that -1.0 is black, 0.0 is grey, and +1.0 is white. However, nearly all of the pixels are in the 0.0 to +1.0 range. The backgrounds are grey, and the image "pen strokes" are white.[^1]

---

#### Predicting your own handwriting

After you write down a digit on a paper, take a photo of it and upload it to your computer, you will first [read the image][4] in MATLAB.
I will use the image below for predicting my handwriting and then, at the end of this post, I will share a few more images of different digits that I've successfully predicted so you can try to predict them on your own, if you would like to, or just try your own handwriting since that's what this post is about, haha.

<figure>
  <a href="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/digit_five.jpg" class="image-popup"><img src="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/digit_five.jpg" alt="Digit five"></a>
  <figcaption>The image of my handwriting that will be used for prediction.</figcaption>
</figure>

```matlab
% Reading the image
img = imread('/path/to/digit_five.jpg');
% Converting RGB img to grayscale
img = rgb2gray(img);
% Resizing it to be 20x20
img = imresize(img, [20, 20]);
% Display img
imshow(img)
```
Now you should see your 20x20 grayscale image, or maybe you can't because it is too small, haha. You can display before resizing anyway.

##### But there is a problem

In the image, our background color is close to white and the digit is black contrary to images in the dataset, also our pixel values are between `0` and `255` whereas pixel values of images in the dataset are between `0` and `1` roughly (-0.1320 and 1.1277 actually). 

This is where we should map our pixel values to comply with the images in the dataset.

---

To better visualize the problem that I've explained above take a look at the image below where I've compared the image of my handwriting and a random image from the dataset.

<figure>
  <a href="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/comparison_digit_five_digit_two.png" class="image-popup"><img src="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/comparison_digit_five_digit_two.png" alt="Comparison of the image of my handwriting and an image from the database"></a>
  <figcaption>Comparison of my handwriting (on the left) and a random image from the dataset (on the right).</figcaption>
</figure>

As you can see from the image we need to map our pixel values between `0` and `1` so that our image will comply with pixel values of images in the dataset which are all between `0` and `1` roughly. 

Moreover we need to cross map our pixel values (I've just made up this term, just please correct me in the comments) meaning that we need to map a pixel value of `0` from our own image to `1` so that our black pixels will turn white and we should map a pixel value of `255` to `0` so that our white pixels will turn gray, in other words, this process will turn our digit white and our background gray which is exactly what we want to achieve.

#### Finally! The mapping part

>If your number X falls between A and B, and you would like Y to fall between C and D, you can apply the following linear transform:[^2]
>```
>Y = (X-A)/(B-A) * (D-C) + C
>which is the same as :
>new_value = (old_value - old_bottom) / (old_top - old_bottom) * (new_top - new_bottom) + new_bottom;
>```

Using the formula above we will map all our pixels. Instead of using `255` and `0` as the `old_top` and `old_bottom`, I've assumed the minimum value in my image matrix to be my `old_bottom` and maximum to be my `old_top`. 

```matlab
absolute_max = max(img(:)); % 192 which is B in the formula or old_top
absolute_min = min(img(:)); % 104 which is A in the formula or old_bottom
```

In other words, I've assumed the minimum pixel value in my image to be black and the maximum pixel value to be white because this way I can map my pixel values to get my digit white and the background gray. Take a look at the image below to see the difference.

<figure>
  <a href="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/comparison_different_old_bottom_and_old_top.png" class="image-popup"><img src="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/comparison_different_old_bottom_and_old_top.png" alt="Comparison of the image of my handwriting and an image from the database"></a>
  <figcaption>Comparison of different old_bottom and old_top values used in mapping pixel values</figcaption>
</figure>

Our `new_top` will be `0` and `new_bottom` will be `1` as we want our white color to turn gray and gray to turn white.

```matlab
% Mapping pixel values
new_top = 0;
new_bottom = 1;
mapped_img = (img - absolute_min) / (absolute_max - absolute_min) * (new_top - new_bottom) + new_bottom;
```

After this mapping process, we can finally use `displayData` function to display our image.

```matlab
displayData(mapped_img(:)');
```

Which will display the image below.

<figure>
  <a href="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/digit_five_mapped.png" class="image-popup"><img src="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/digit_five_mapped.png" alt="Mapped digit five"></a>
  <figcaption>Grayscale image of my handwriting with the pixels mapped.</figcaption>
</figure>

We can now try to predict our handwriting using `predictOneVsAll` with already trained classifiers `all_theta`.

```matlab
predictOneVsAll(all_theta, mapped_img(:)')
```
<figure>
  <a href="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/prediction_of_the_image_digit_five.png" class="image-popup"><img src="{{ site.url}}/images/2019-09-25-coursera-machine-learning-predicting-your-handwriting/prediction_of_the_image_digit_five.png" alt="Prediction of the image digit five"></a>
  <figcaption>Prediction of the handwritten digit in our image.</figcaption>
</figure>

### Oops! Something went wrong.
Well, actually nothing went wrong for this particular image. As I've mentioned at the beginning of my post. This prediction relies on the background color and its consistency through background pixels, so if you are trying your own handwriting you might not have gotten the background color right, but this might not be the only reason. 

#### What should I do then?
After I've tried to predict all the digits in different sizes and colors on post-its without fiddling with the pixel values. I've decided to fiddle with the pixel values to try to get the right background color.

I've come up with the function below to increment all pixel values and try to predict the digit with the incremented pixel values. Ones it predicts the digit right the function returns the number that has been added to all pixel values, otherwise `-1` is returned.

```matlab
% This function tries to find the right value which makes the 
% background color the closest to the one in the dataset images
% so that our "predictOneVsAll" function can predict our handwriting correctly.
% If predicted correctly with the number summed, the number returned. -1 otherwise.

function p = findRightBgColor(img, all_theta, y)
  max_value = max(img(:));
  min_value = min(img(:));
  p = -1;
  mapped_img = (img - min_value) / (max_value - min_value) * (-1) + 1;
  for i = -0.9:0.01:1
    temp = mapped_img + i;
    prediction = predictOneVsAll(all_theta, temp(:)');
    if prediction == y
      p = i;
      break;
    end
  end
end
```


[^1]: [Tips for classifying your own images][5] - ML:Programming Exercise 4:Neural Networks Learning
[^2]: [Mapping numbers][6] - Stack Overflow

[1]: https://www.coursera.org/learn/machine-learning
[2]: https://www.coursera.org/learn/machine-learning/discussions/weeks/4/threads/Jk_M4t1AEemXpRLspIcZXA
[3]: https://www.mathworks.com/help/matlab/ref/rgb2gray.html
[4]: https://www.mathworks.com/help/matlab/ref/imread.html
[5]: https://www.coursera.org/learn/machine-learning/resources/Uuxg6
[6]: https://stackoverflow.com/a/345204/4796762
