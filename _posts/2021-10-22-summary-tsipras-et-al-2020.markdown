---
title:  "Summary of »From ImageNet to Image Classification: Contextualizing
Progress on Benchmarks«, Tsipras et al. 2020"
---
In this blog post I summarize the conference paper [*From ImageNet to Image
Classification: Contextualizing Progress on Benchmarks*][paper], written by
Dimitris Tsipras, Shibani Santurkar, Logan Engstrom, Andrew Ilyas & Aleksander
Mądry, which appeared at ICML 2020.

Tsipras et al. revisit the creation process of
the ImageNet dataset. Essentially, [Deng et al.][imagenet] picked 1000 words
from [WordNet][wordnet] as classes and used them to query search engines for
images. To verify the correctness of the candidate labels (the query), for each
class they ask Amazon MTurk annotators to select from a pool of images which of
them contain an object matching the class description. Each image-label pair
receiving »enough« votes for the candidate label, counts as correct image-label
pair. The above verification procedure is called CONTAINS task throughout this
post (and also in Tsipras et al.'s paper).

Tsipras et al. make the following observations regarding the creating process:

  * The candidate labels are shown in isolation, only asking whether an image
    contains an object that fits. Images containing multiple objects are not
    truly captured by this setup. It might even be the case that the assigned
    candidate label does not represent the main object of an image.
  * It is not revealed what other (related) labels exist. Since the CONTAINS
    task is set up class-wise, the candidate label is fixed in the first place
    and annotators are not able to select labels that suit a specific image
    better.

To better understand the ground truth of the ImageNet images (and to see how it
might deviate from the assigned ImageNet labels), Tsipras et al. select a
stratified subset of size 10000 from the ImageNet validation set. They obtain
multiple candidate labels per image by querying 10 existing ImageNet
classification models and combining their top-5 labels and the original ImageNet
label. Then they present the image-labels combinations to several annotators
(again Amazon MTurk) and let them perform the CONTAINS task with one important
difference: Each image may receive votes for every one of its candidate labels
(whereas during the ImageNet creation process an Image is only allowed to
receive votes for its only candidate label). To reduce workload for the next
step in the pipeline, Tsipras et al. restrict the amount of candidate labels per
image to the 5 most confirmed labels (plus the original ImageNet label).

Now follows the CLASSIFY task. Annotators are presented an image along with its
candidate labels. They are asked for each candidate label to mark it as
valid, if the image contains such a distinct object and to select among the
candidate labels, which represents the main object of that image.
The votes are aggregated among the different annotators.
<figure>
  <img src="{{site.url}}/images/2021-10-22-summary-tsipras-et-al-2020/pipeline.png" alt="pipeline"/>
  <figcaption>
    The CONTAINS and CLASSIFY dataset creation pipeline.
    Figure obtained from <em>From ImageNet to Image Classification:
    Contextualizing Progress on Benchmarks</em>, Tsipras et al. 2020. Copyright
    by Tsipras et al. 2020.
  </figcaption>
</figure>

The result of the above pipeline is a more detailed labeling of the images,
providing for each one a main label and allowing additional labels. Tsipras et
al. observe, that roughly 20% of the images have multiple labels.
They question, whether the common practice of evaluating image classifiers by
top-1 accuracy is too pessimistic, penalizing classifiers selecting another
valid label, different from the (only) main label. A common alternative to fix
this issue, the top-5 accuracy, is in their eyes too optimistic for the other
80% of the images, forgiving errors in situations where there's only one valid
label. They suggest using top-1 accuracy, but with respect to all valid labels.
How ImageNet models perform using that metric is depicted in the following image:
<figure>
  <img src="{{site.url}}/images/2021-10-22-summary-tsipras-et-al-2020/top-1_main_label_accuracy.png" alt="pipeline"/>
  <figcaption>
    Benchmark results obtained by using another metric on multi-class images.
    Figure obtained from <em>From ImageNet to Image Classification:
    Contextualizing Progress on Benchmarks</em>, Tsipras et al. 2020. Copyright
    by Tsipras et al. 2020.
  </figcaption>
</figure>

Another observation is that roughly one third of the 2000 multi-label images
have a main label (according to CLASSIFY), which differs from the original
ImageNet label. Yet, trained models are good at classifying those images
correctly according to the ImageNet labels. Tsipras et al. suppose that such
models learn ImageNet idiosyncrasies (which might be negative considering the
actual task a model should solve).

Tsipras et al. further notice that the »correction step« of the original
ImageNet creation process (filtering out images which do not receive enough
votes in the CONTAINS task) does not work reliable, as roughly 40% of the
images have associated labels, which receive at least as many votes during the
CONTAINS task as the original ImageNet label. Tsipras et al. highlight, that
this phenomenon also occurs for single object images.
Having this in mind, they argue the ImageNet labels may be biased by how the
search engines respond to the search queries.

From the CONTAINS and the CLASSIFY tasks Tsipras et al. performed, they are able
to define two metrics to evaluate ImageNet models, which they deem to align
better with the underlying task of object recognition. The first is the
*selection frequency* (SF; the relative frequency of annotators confirming for a
given image-label combination that the label is present in the image). The
second is the top-1 main label accuracy.
Using these metrics, they assess how the top-1 accuracy, with respect to
the original ImageNet test set and the ImageNet labels, of selected ImageNet
models relates to several properties:

  * **indistinguishable selection frequencies** The predictions of models
    achieving more than 80% top-1 accuracy have almost the same SF as the
    ImageNet labels themselves (that means annotators are almost equally likely
    to validate the predicted label as the ImageNet label – note that
    the predicted label might or might not be the same as the ImageNet label).
    Tsipras et al. argue this indicates that high performance models approach
    what non-expert annotators may recognize as ground truth. When accuracy
    increases further, it will become difficult to attribute the gains to
    progress on image recognition or to overfitting on ImageNet labels.
    <figure>
      <img src="{{site.url}}/images/2021-10-22-summary-tsipras-et-al-2020/selection_frequency.png" alt="selection frequency" />
      <figcaption>
        The top-1 ImageNet accuracy of several models compared to their
        redictions' selection frequency.
        Figure obtained from <em>From ImageNet to Image Classification:
        Contextualizing Progress on Benchmarks</em>, Tsipras et al. 2020. Copyright
        by Tsipras et al. 2020.
      </figcaption>
    </figure>
  * **increasing ImageNet accuracy increases main label accuracy** Models that
    perform better at the original ImageNet benchmark also perform better at
    recognizing the main label of Tsipras et al.'s subset. Note however, at a
    flatter rate.
    <figure>
      <img src="{{site.url}}/images/2021-10-22-summary-tsipras-et-al-2020/main_label_accuracy.png" alt="main label accuracy" />
      <figcaption>
        The top-1 ImageNet accuracy of several models compared to their
        main label accuracy.
        Figure obtained from <em>From ImageNet to Image Classification:
        Contextualizing Progress on Benchmarks</em>, Tsipras et al. 2020. Copyright
        by Tsipras et al. 2020.
      </figcaption>
    </figure>
  * **increasing ImageNet accuracy decreases noticeable error** Obviously do
    models make less mistakes (according to the ImageNet label) when their
    ImageNet accuracy increases, but also the distribution of their
    misclassifications related to the selection frequencies changes. The errors
    of low-accuracy models like Alexnet have a peak at SF between 0% and 20%,
    whereas the errors of Efficientnet B7 have a peak at SF between 80% and
    100%. That means their errors become »more reasonable« from the human
    annotators' point of view.
    <figure>
      <img src="{{site.url}}/images/2021-10-22-summary-tsipras-et-al-2020/prediction_errors.png" alt="prediction errors" />
      <figcaption>
        The distribution of prediction errors of three selected models,
        categorized by the selection frequency (i.e. how often it is still
        considered as a valid label by human annotators).
        Figure obtained from <em>From ImageNet to Image Classification:
        Contextualizing Progress on Benchmarks</em>, Tsipras et al. 2020. Copyright
        by Tsipras et al. 2020.
      </figcaption>
    </figure>

All in all Tsipras et al. point out that the ImageNet dataset has flaws,
probably related to how the dataset was created. They provide a creation
pipeline addressing those flaws. Based on that, they show how progress on
ImageNet accuracy relates to progress on the underlying object recognition task.



[paper]: https://proceedings.mlr.press/v119/tsipras20a.html
[imagenet]: https://ieeexplore.ieee.org/abstract/document/5206848
[wordnet]: https://doi.org/10.1145/219717.219748
