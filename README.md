# ML-CV-Ideas

Like a lot of other machine learning & computer vision researchers, I occasionally stumble across a "wouldn't it be cool it try this?" idea that I normally wouldn't get a chance to work on. I wanted a centralized, non-device specific place to store some of my ramblings, with no expectation of completeness or that it's even remotely not ridiculous, that I could easily share with others to get some feedback (or be told it's already been done before :) ). Below is the list of these ramblings - apologies for any lack of proper grammar, coherency or sanity.

* Learned Augmentation for Domain Adaptation
  * Problem: Given a source dataset with lots of samples and labels, can we learn well on a target with either few samples, or lots of samples but few labels?
  * Many approaches focus on changing the feature extractor (starting from a source) to extract better on the target (like [MinimaxEntropy](https://arxiv.org/abs/1904.06487) or [Fully Test Time Adaptation](https://arxiv.org/abs/2006.10726)). However, changing a feature extractor can be problematic, especially if we have few samples / labels. One solution could be to augment the images in the target domain to more closely resemble the images in the source domain. Given a classifier trained on source data (CS), we want some operation G that takes in an image and outputs some similarly shaped image (like an autoencoder). If G(target data) ~ source data, then CS(G(target data)) will classify on target data. (Classifier is used here for simplicity, although the larger point is the underlying feature extractor rather than the specific task).
  * An initial attemp at this could be: given two labelled datasets (source and target) and a classifier trained on source, train CS(G(target data)), where the weights for CS are either frozen or update very slowly. Obviously some choices need to be made on choosing G, and there's some more room for more significant experimentation.
* Learned Augmentation for Contrastive Learning
  * Something similar to the above idea but learn one (or both) of the augmentations for contrastive learning.
  * One example: Do [FixMatch](https://arxiv.org/abs/2001.07685) but learn the strong augmentation (FixMatch Diagram is below). This can be made more complicated by having various complexities to G (G2 is a stronger augmentation than G1, G3 than G2, ....)  and then stack each successive augmentation. (ex: For the 1st epoch, have rotation be the weak augmentation, G1 be the strong augmentation. 2nd epoch - G1 = weak augmentation, G2 = strong augmentation ...).
 ![FixMatch Diagram](https://raw.githubusercontent.com/google-research/fixmatch/master/media/FixMatch%20diagram.png)
* Hierarchical Explainability
  * Let S be a a set of vectors, and X is a some sample vector. X is explained by E - another set of vectors that can share some parts with S - by some function g(X, S) --> E, meaning g discovers which vectors from S are necessary in explaining X. There may also be a function f(X,E) --> True/False, which checks if X is explained by E. 
  * By this definition, we now have a definition of "expainability", but truthfully, all we've done so far is offload the work to defining the function g, which is the explainable function. For basically any problem, it's unlikely for to be a simple function, or even one we can reasonably find. So the goal is to offload some of the explanability burden off of g(X,S) by hierarchically explaining X, ie explain a system by explaining its parts.
  * We can expand S to be S0...SN, where each level is hierarchically explained by a lower level. At a given St, use S0...t-1 to generate E, which is then added to St. To initialize, we define S0 (can be thought of as first principle knowledge). (TODO: add more extensive explanation and example).
      * Alternatively, it may make sense to reverse this - rather than going bottom up and defining first principle knowledge, start explaining at SN (which is where the data is coming in at).
  * The above may be a nice interface, but obviously the key is the explanation. So the parts that need to be defined are:
      * S0 --> what are these first principle vectors?
      * g(X, S) --> the function that takes in X and a set of vectors S and returns a set of vectors E that explain X. This is really the crucial part of this formulation. Importantly, this codifies "what is explanation" and there may not be an agreed upon answer for any given problem (in fact, there probably isn't). Defining g is effectively the difficult part of this problem.
      * (optionally) f(X,E) --> the function that takes in X and a set of vectors E and checks if X is explained by E.
   * Note: SN doesn't need to be set of vectors, could be any data.
   * This has serious parallels to decision trees. I think a potentially better way of formulating this is a decision tree where it's designing it's own features rather than using given ones.
* Explore underspecification for unsupervised learning and different domain adaptation techniques
  * Specifically extending the work Google published on [underspecification in machine learning](https://arxiv.org/pdf/2011.03395.pdf) on unsupervised learning (particularly contrastive learning) and domain adaptation / test-time adaptation
* Saliency Generation from Gradients
* Explicitly Learned Salience
  * Saliency maps are used for explainability to highlight what's important in the image (what the model is focusing on). Saliency maps share a lot of similarities to segmentation masks. An interesting approach therefore would be to explicitly learn salience, like a segmentation task. 
  * A fairly simple, if almost boring experiment would be to take a segmentation dataset, and train a U-Net (or your choice of segmentation network) on this as a binary task (object vs background, which depending on the dataset may represent salient vs not). Then use the embedded features to train a classifier (ex: a linear layer on top of the U-Net encoder, where the encoder weights are shared between the classifier network and segmentation network). The classifier and segmenter could either be trained in series or parallel (if done in parallel, some thought will need to be given on handling this elegantly). The mildly interesting part of the experiment would be to see how training order affects classification and segmentation results and generalizability (3 options - train classifier then segmenter. train segmenter then classifier, or train both in parallel). At inference, given an image, we can run it through the classifier to get its classification, and also through the whole segmentation network to get it's salience. (This can be extended to add object detection as well for fun).
  * The above by itself isn't particularly interesting (basically given a segmentation task can we do classification, which doesn't really add value). What would be a lot more fun would be - given a classifier, can we train a segmenter that gives us salience? Going beyond this, use the concatenations in the U-Net to learn saliency.
* Inverted U-Net
  * Playing off the saliency map idea, given a segmentation map, can we reconstruct the original image? (Basically train a unet as normal, but flip the inputs and outputs). Not entirely sure what the benefit of this would be other than just curious what would happen (we wouldn't expect it to be able to reconstruct a full image from just a mask, as there are a lot of extraneous details that you just can't get - the problem is severely underspecified.)
* Do features in deep learning follow the peter principle?
 * The Peter Principle was a satirical observation that, in organizations, people are promoted to their "level of incompetence". In a normal organization, the criteria for promotion involves the succes of the individual at their currentl level, and the lack of success precludes them from being promoted. So the success level of the employee at their level is used to predict their success at the next level. However, as skills / talents in one level don't necessarily equate to sucess at the next level, people will be promoted to levels they won't be suited for, and then will not be successfull there (hence rising to their "level of incompetence"). So to summarize - people are promoted based on their competence, and therefore will keep getting promoted until they reach a job they are incompetent at, which results in the people getting stuck at a job they are incompetent at.
 * I don't really have any concrete ideas on how this can be related to ML, but I wanted to toss in two rough, kernels of ideas around this applied to machine learning that maybe I'll have some inspiration from at some later date.
     * 1) An explanation for why prediction on stress tests / real world deployment is poor: As noted above, models often do well on test sets but then struggle when deployed in the real world (often due to either data shift or underspecification). To reformulate the problem in the Peter Principle fashion - employees = trained & deployed model, promotion = training process, competence = how well the models do on some data (usually defined by some loss criterion). So is the following true - models are trained based on their loss criterion on some data, and therefore will optimize on seen data, until they reach data on which the optimization based on the previous seen data using the loss criterion doesn't help the model do well, which results in the model's poor performance. Formulated this way, this is basically data shift, so not insightful in any way. I was hoping there may be a way to reformulate this into something with a little more insight, so I tossed this in here, but as it stands right now this isn't interesting.
     * 2) Poor rewarding of features: if in the  Peter Principle fashion, employees are interpreted as features, the promotion is the updating of the feature weights in the learning process (i,e training), and competence is instead the loss criterion, then the question becomes is this true - the weights of features are adjusted based on how well they fit the loss criterion (gradients in NN's), and therefore features will keep getting rewarded until they reach a state where increasing their weights no longer benefit minimizing or maximizing the loss criterion. This formulation as is doesn't make a ton of sense either - in the Peter Principle, past competence is a bad predictor for future success, whereas minimizing / maximizing loss criterion is usually a good predictor of future success in most problems.
* Copernican Learner
  * J Richard Gott developed the Copernican Principle (learn more [here](https://towardsdatascience.com/how-to-be-less-wrong-5d6632a08f). Basically, given no other information about an object other than 1) it's existence is finite, 2) how long the object has existed, you can assign a confidence interval to how long will continue to exist. The very rough idea here is to apply this same principle to feature learning, except instead of in the time dimension, apply it to the "importance" dimension. Some vague examples of what this could look like - boosting different learners, combining / learning directly from features, reducing dimensionality of data (like t-sne or PCA). While this sounds like there may be some fun avenues down this path, the biggest thing stopping me from going more into depth is "why use this?". In Gott's formulation it's used when you have literally no other information / priors, which you are very unlikely to have in any realistic machine learning setting.

