---
title: Quality and Quantity
date: 2022-02-01T04:07:25.021Z
draft: true
longform: true
cover: media/john-schnobrich-flpc9_vocj4-unsplash.jpg
---
One of the best ways I’ve found to gauge one’s experience level in software is to consider the size of work that can be accomplished while still maintaining the level of quality you consider “good code”.

For me I’ve found this to be around 1k SLOC where I can maintain high quality in a combination of code, documentation, and testing. For larger projects I can maintain reasonably quality for at least 10k SLOC as an IC.

For the concept of "good code" and how it relates to experience I'd like to elaborate on some points I've learned:

* Chasing "Best Practices"
* Reliance on Tooling
* Dealing with Disappointment

###  Chasing "Best Practices"

When you focus on improving your skill level it can be very helpful to watch conference talks on subjects related to your part of the industry. Many of these talks often proclaim various patterns, styles, or techniques to be a "Best Practice". While it's good to try and follow along with widely adopted practices for the sake of creating inter-organizational standards, it's a common mistake to take these as the only way to work. 

When developing within an existing codebase it's more important to rely on following along with existing conventions to maintain cohesivity. Having a codebase that is self consistent decreases the complexity of having multiple people developing on it simultaneously and lowers the overhead for new developers to become acquainted with it.

### Reliance on Tooling

In recent years the popularity of tooling such as code formatters and static analysis has drastically increased. Editors and development environments have also seen their capabilities to assist users also improve. Anyone who has been able to make use of these tools can attest to their ability to improve productivity. However, outside of environments where the developer tooling is explicitly specified and enforced tooling can become an inhibitor to those not already initiated.

Consider a case of a hypothetical programmer developing on your codebase using your system equivalent of notepad. Is it possible to navigate your codebase without a "go anywhere" shortcut? Do you have comments littered with control sequences for your formatters or analyzers? Are there instructions on how to build and deploy your software without clicking a "build" button? 

If your organization doesn't have standardized tooling for all developers, or if that tooling is present but isn't used then you need to make sure that it is still possible to develop using processes other than your own. To promote adoption tooling should be something developers want to use instead of something that gets in the way of them doing their job.

### Dealing with Disappointment

One factor that is sure to vary from individual to individual is the standard of what they consider is "good". As you gain experience this standard is sure to increase so when looking back on old projects it's okay to feel some disappointment to see that something you were once proud of is no longer up to snuff.

When looking back at old projects, whether your own or others, try and understand the context they were developed in. Sometimes things that seem unnecessary are because at the time the code was written, the better alternative you're comparing it against now didn't exist.