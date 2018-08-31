---
title: "Exporting TensorFlow models to ML Kit"
layout: post
date: 2018-08-31 10:00
image: /assets/images/coffee.jpg
headerImage: true
tag:
- android
- firebase
- tensortlow
- machine learning
category: blog
author: miquel
description: Exporting TensorFlow models to ML Kit. Using toco convert to export models directly from Python code
---

ML Kit is the new tool from the Firebase family to bring TensorFlow into your
mobile applications. With ML Kit, it is easier than even to load and run
trained models on Android, if you want to get started using ML Kit, I recommend
you the following codelab: [Identify objects in images using custom machine
learning models with ML Kit for
Firebase](https://codelabs.developers.google.com/codelabs/mlkit-android-custom-model/#0).

In this article, you will learn a way to export your custom TensorFlow models
to the tflite format to run on ML Kit.

---

The steps to export a model are generally these:

1. Export the graph (pb file format)
2. Export the variables values (in the form of checkpoints)
3. Freeze the grapth with the variables values
4. Convert the frozen graph to TF Lite.

You can read more about the process in the Developer Guide for TF Lite.

However, I find more convenient if I can do those steps directly in my Jupyter
Notebooks in a more streamlined way.

---

To be able to export models directly from our code, we can use the `toco_convert`
method to convert the TensorFlow session graph to a TF Lite model.

This example here shows how to use `toco_convert`:

<script src="https://gist.github.com/miquelbeltran/efd06894772cf65d4ef7d57fce94a4ab.js"></script>

What you are seeing, is a simple TensorFlow model that has a single float input
and a single float output, and performs a +1 operation. It is essentially a `++`
operator implemented in TensorFlow.

The interesting part is the call to `toco_convert`, which converts the model to a
TF Lite model, then we call to the write method to store it.

---

What happens when we incorporate variables into the mix? that just calling to
`toco_convert` would not work.

In this example, we do a similar operation, our input is a 1x1 Tensor and we
multiply it by another 1x1 Tensor (a variable w initialised to 0, which later
is changed to 2).

<script src="https://gist.github.com/miquelbeltran/b9f4b17b8380aaa36c985c1b553494d5.js"></script>

When we try to export this example, we will get lots of errors in the console,
which will look like these:

```
Converting unsupported operation: Variable...
```

That’s essentially because we are trying to convert variables to TF Lite,
rather than their “frozen” values to constants.

---

To fix that, we need to freeze our graph first:

<script src="https://gist.github.com/miquelbeltran/b73f88f620ffe6e7e7ae743b96e4298e.js"></script>

`freeze_session` is a modified version of the tool `freeze_graph` from
TensorFlow, and can be found here:
[https://stackoverflow.com/a/45466355/673294](https://stackoverflow.com/a/45466355/673294)

What `freeze_session` is doing internally is calling to
`convert_variables_to_constants` to store the current variables values into fixed
ones, so the exported model contains those values.

For example, if you train a model, you want to export the current values for
your neural network weights so you can use it in your application.

In this example, the variable w keeps the value of 2 after exporting to TF Lite
format.

---

In short, using `toco_convert` and `freeze_session` together, simplify exporting
TensorFlow models to ML Kit directly from your Python code.

Just keep in mind, that not all operations are supported by TOCO/TF Lite, so
you may have problems exporting certain neural networks like RNNs with LSTM
cells. In that case, we can only hope and wait for them to be supported.

---

Are you interested in incorporating deep neural networks to your products? Do
you have a need for Machine Learning optimised for mobile? Let’s talk! I am
looking for future freelancing opportunities in mobile and machine learning.
Check for more info: [http://beltran.work/with-me/](http://beltran.work/with-me/)

