---
layout: post.html
title: "Part 3: JSON Parsing"
tags: [api, generator]
url: "/api/part-3/"
---

Parse the response from the API call into something meaningful.

### Generate Plot function

Because we _love_ data visualization, and couldn’t get enough of matplotlib, let’s see what we can produce with the combination of CPI data and platform prices from Giantbomb!

We’ll need to add import statements for both matplotlib and numpy like in our previous tutorial:

```python
import matplotlib.pyplot as plt
import numpy as np
```

Now for the `generate_plot` function (outside of the `GiantbombAPI` class), we’ll take in the yielded platforms, as well as an output file so we can save our constructed plot (instead of just `plt.show()`).

Comments on the process are inline:

```python
def generate_plot(platforms, output_file):
    """Generates a bar chart out of the given platforms and writes the output
    into the specified file as PNG image.

    """
    # First off we need to convert the platforms in a format that can be
    # attached to the 2 axis of our bar chart. "labels" will become the
    # x-axis and "values" the value of each label on the y-axis:
    labels = []
    values = []
    for platform in platforms:
        name = platform['name']
        adapted_price = platform['adjusted_price']
        price = platform['original_price']

        # Skip prices higher than 2000 USD simply because it would make the
        # output unusable.
        if price > 2000:
            continue

        # If the name of the platform is too long, replace it with the
        # abbreviation. list.insert(0, val) inserts val at the beginning of
        # the list.
        if len(name) > 15:
            name = platform['abbreviation']
        labels.insert(0, u"{0}\n$ {1}\n$ {2}".format(name, price,
                                                     round(adjusted_price, 2)))
        values.insert(0, adapted_price)

    # Let's define the width of each bar and the size of the resulting graph.
    width = 0.3
    ind = np.arange(len(values))
    fig = plt.figure(figsize=(len(labels) * 1.8, 10))

    # Generate a subplot and put our values onto it.
    ax = fig.add_subplot(1, 1, 1)
    ax.bar(ind, values, width, align='center')

    # Format the X and Y axis labels. Also set the ticks on the x-axis slightly
    # farther apart and give then a slight tilting effect.
    plt.ylabel('Adjusted price')
    plt.xlabel('Year / Console')
    ax.set_xticks(ind + 0.3)
    ax.set_xticklabels(labels)
    fig.autofmt_xdate()
    plt.grid(True)

    plt.savefig(output_file, dpi=72)
```

You can elect to have `plt.show(dpi=72)` instead of `plt.savefig(output_file, dpi=72)` if you do not want to save the file, and have the graph just pop up when running the script (or both!).

### Generate CSV function

We can also make a function that takes the yielded data and produces a CSV file for us.

Let’s use a new library, `tablib` to help handle the CSV production.  [tablib](http://docs.python-tablib.org/en/latest/), written in Python by the same author of [requests](http://twitter.com/kennethreitz) that allows you to format the output of data into a tabular dataset (among other things).

We should add an import statement for it too:

```python
import tablib
```

Similiar to `generate_plot` function, we’ll take two parameters: `platforms` which is the yielded data from our API class, and `output_file` to save the formatted data where we want to. We use the `tablib` module to assign headers to a dataset, then append each item of our platform data to the dataset. Last, we write to a file that passed in as a parameter using the `.csv` method that `tablib` gives us. Comments on process are inline:

```python
def generate_csv(platforms, output_file):
    """Writes the given platforms into a CSV file specified by the output_file
    parameter.

    The output_file can either be the path to a file or a file-like object.

    """
    dataset = tablib.Dataset(headers=['Abbreviation', 'Name', 'Year', 'Price',
                                      'Adjusted price'])
    for p in platforms:
        dataset.append([p['abbreviation'], p['name'], p['year'],
                        p['original_price'], p['adjusted_price']])

    # If the output_file is a string it represents a path to a file which
    # we will have to open first for writing. Otherwise we just assume that
    # it is already a file-like object and write the data into it.
    if isinstance(output_file, basestring):
        with open(output_file, 'w+') as fp:
            fp.write(dataset.csv)
    else:
        output_file.write(dataset.csv)
```

Let’s put the final touches of this script so we can see it in action. [The final part brings the logic together &rarr;]( {{ get_url("/api/part-4/")}})