+++
date = '2024-11-22T13:00:51-03:00'
draft = false
title = 'Lessons from Building ‘Pandas for Temporal Data’ with Google'
type = 'post'
categories = ["blogpost"]
tags = ["python","c++","temporal data","machine learning"]
+++


Time-series data is everywhere, from tracking user behavior to monitoring IoT sensors. Last year, while working at Tryolabs, I had the privilege of collaborating with Google to build an open-source Python library named Temporian. If Pandas is the go-to library for tabular data, Temporian aims to fill that role for temporal data.

{{< figure src="/images/temporian/temporian_diagram.avif" alt="Diagram showing Temporian as part of a data pipeline" caption="The role of Temporian in a data pipeline." width="70%" >}}

Here’s a glimpse into my experience as a developer on this incredible project and the lessons I learned along the way.

## The Problem

There are many libraries for time-series preprocessing, but Temporian is for time-sequences in general. The difference is subtle, a time-series is uniformly sampled (i.e: has a fixed sampling rate), while a time-sequence is just a set of events sampled at any possible timestamp.

{{< figure src="/images/temporian/temporian_data.avif" alt="Different types of temporal data" caption="Basically anything with a timestamp can be thought as an event in a time-sequence." width="70%" >}}

There are also libraries particularly for univariate data, or only numerical data, among other particularities.
But Temporian is intended to allow modeling anything with a timestamp, basically:
 - Uniformly and non-uniformly sampled data
 - Single and multi-source events
 - Complex multivariate time-sequences

{{< figure src="/images/temporian/temporian_plots.avif" alt="Plots with different types of temporal data" caption="Different types of temporal data." width="100%" >}}

On top of that, it's intended to be blisteringly fast even with tons of data. I mean, it's backed by Google. So the engineering design is an important aspect, as well as writing the critical parts in C++ to optimize processing.

Finally, it's intended to avoid common mistakes when handling temporal data, which usually arise when other libraries like pandas or numpy are adapted for these purposes. The most common mistake (precisely because it's hard to spot sometimes) is future leakage. Which means leaking data from events in the future, to build features if previous events. Temporian is specifically designed in a way that avoids that, ensuring robust, error-free feature engineering—essential for machine learning applications.

### From Python to C++ and Beyond

For me as a developer, there were many interesting challenges in this project, from which I learned a lot:
 - **Writing Performant Operators in C++**:
One contribution I particularly remember was the `tick_calendar` operator, which generates timestamps using a cron-like syntax (e.g., “every day at 2:15 AM”). This required diving deep into C++ for optimal performance—a challenging but deeply rewarding experience.
 - **Ensuring Quality**:
Working with Google’s open-source standards meant adhering to the highest levels of code quality and test-driven development. Every release was meticulously reviewed to match the rigor of libraries like Pandas or NumPy.
 - **Intuitive Design through UX Testing**:
Another highlight of the project was our focus on usability. We created test protocols that challenged developers to solve problems using Temporian’s documentation alone. Watching real users interact with our library revealed surprising gaps—and ultimately shaped a much better product.

## What I Learned
Each of the points mentioned above can be mapped to some specific learnings:

 - Cross-Language Development is a Superpower: Again, combining Python for ease of use and fast development and C++ for performance.
 - Test-Driven Development is Non-Negotiable: When building a library meant to last, rigorous testing isn’t just a good practice—it’s the foundation.
 - User Feedback is Priceless: Designing for intuition is hard. Real-world testing with developers showed how they interpreted our documentation and APIs, often in ways we didn’t expect.

## Final thoughts

Temporian is open-source, hosted on [Google’s GitHub](https://github.com/google/temporian) repository, and even announced on [Google’s TensorFlow blog](https://blog.tensorflow.org/2023/09/forecasting-with-tensorflow-decision-forests-and-temporian.html) as a complement for ML pipelines. Seeing the library achieve this level of impact is one of the most gratifying experiences of my career.