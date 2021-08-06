# Chapter 4: Encoding and Evolution
In large applications, code changes cannot happen instantaneously: 

* With server-side applications you may want to perform a *rolling upgrade* (also known as a *staged rollout*), deplying the new version to a gew nodes at a time, cheching whether the new version is running smoothly, and gradually working your way through all the nodes. This allows new versions to be deployed without service downtime, and thus encourages more frequent releases and better evolvability. 

* With client-sode applications you're at the mercy of the user, who may or may not install the update for some time.

That means that old and new versions of the code and new data formats, may potentially coexist in the system at the same time. In order for the system to continue running smoothly, we need to maintain compatibility in both directions:

*Backward compatibility*: newer code can read data that was written by older code

*Forward compatibility*: Older code can read data that was written by newer code