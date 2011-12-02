---
layout: writer
title: Code@Makalu
---

# Some Thoughts on Code @ Makalu

I've put together some thoughts on a few topics related to software development at Makalu. At a minimum, I hope this will start useful discussions. Thank you for reading.

Justin

## Coding Style (or) Coloring Within the Lines

To start with I'd like to make a few suggestions about coding style as we move forward. I have no desire to make claims on what the BEST coding style is. Only a couple of points of focus I think aid productivity, developer happiness and collaboration.

### Readability Counts

One thing I admire about Python is it's focus on simplicity and readability above all else. [The Zen of Python](http://www.python.org/dev/peps/pep-0020/) is a good read for any developer and, in my opinion, applies to any language. Mostly, though, it's a good thing to keep in mind that every line of code will be read far many more times than it will be written. Comment where appropriate. Use descriptive variable names. The shortest way to write something is not always the best.

I also recommend anyone who has a current Apple developer account watch this presentation (if you're not an Apple developer, and you'd like to see this, let me know):

[Writing Easy-To-Change Code: Your Second-Most Important Goal as a Developer](http://developer.apple.com/videos/wwdc/2011/#writing-easy-to-change-code-your-second-most-important-goal-as-a-developer)

### Code to the Style of the Project

When working on an existing code-base, the second most helpful thing a developer can do is mimic the overall style of the existing codebase. This includes using existing patterns or conventions (unless a good case can be made to break from the norm). Ruby provides a lot of flexibility in how something might be accomplished, but readability and maintainability are better served by consistency.

### External Dependencies Are Expensive

The developer community has greatly benefited from the availability of reusable code on the internet. We should try to remain aware, however, that every external dependency we add to a project has a cost. There is cost in learning to use a new, possibly poorly documented, API. There is also a cost in having to update both the new library and it's own dependencies. This is especially true on iOS where we can not control the deployment platform. An OS update that breaks a 3rd-party library can mean a lot of extra work and wasted time. I'm not championing re-inventing the wheel every time. I just suggest we always weigh the pros and cons of an in-house implementation and/or using an official lower-level API to accomplish the same task.

## GitHub, Branching and Pull Requests

Next I propose we standardize on a new workflow, for our Rails projects, where the Master branch is deployed to production and any non-trivial development takes place on feature or release branches. These branches would be deployed to the staging server individually for testing and review. Having the ability to easily reset staging, both code and data, to be identical to production will be key for testing migrations and avoiding conflicts (Niall has indicated this shouldn't be an unreasonable request). I've also been using a modification to our deploy recipe that allows me to deploy the currently checked-out branch to stage when executing 'cap stage deploy' (this would only work for stage of course, production would always be deployed from Master) that I think would be helpful.

Once the developer (or developers) feel ready to share what they're working on, they should create a pull request from the development branch against Master. Pull requests not only provide a convenient repository for code reviews, comments and other meta-data, they store a history of what changed, when and why. They also provide automatic notifications that will help keep everyone up on changes to our individual projects. Lastly, this history will be useful in the future when things need to be updated or modified and motives may not be clear from the code. Pull requests should be used for nearly all deployed code. Little things like spelling corrections would be tested locally, pushed on Master and deployed.

Once commenting, testing and review (if needed) is complete, the pull request will be merged with Master and deployed. This can be done by the original developer, the project manager or whomever is overseeing the code-base. After this, the source branch should be removed from the repository (to avoid clutter).

The biggest advantages to this approach would be an increased ability to develop features in parallel, a richer history, and more transparency. It would also avoid the issue where production gets ahead of master because of trivial changes pushed directly to production. The disadvantage would be possible collisions on the stage server (developers would need to coordinate staging deploys), and the the small overhead of maintaining branches and generating pull requests.

## Exceptional

Lastly, I'd like to suggest we all try and make a better attempt to keep our Exceptional queues clean for all of our apps. For a while now Exceptions thrown by testing new features are left open among real production errors and real exceptions aren't always closed when they're fixed. Recently I've gone through most of the queues (MLK still needs review, I'm hoping Josh can help me with this.) and cleared out the old and (from what I can tell) no longer relevant. This also means every developer needs access to our account. It might be a good goal to *try* to have all queues empty of development errors and any low hanging fruit, by the end of the day on Friday each week.

## Comments and Suggestions

…are most welcome.
