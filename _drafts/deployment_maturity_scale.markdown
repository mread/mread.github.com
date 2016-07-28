
Here's a maturity scale for automated deployment. I'd like to expand it eventually to show the kind of solutions you need to put in place to achieve each one. Perhaps also the criteria for achieving a particular stage.

1. manual ad-hoc deployment
1. deploy the latest version of all components in parallel
1. deploy only the components that have changed in a parallel
1. specify simple upgrade paths such that particularly components are advanced in version, tested for compatibility with older dependencies and then only the desired components are deployed. Can only ever go forward.
1. ability to specify any combination of component versions, test this combination together, deploy it
1. run multiple different but compatible combinations in live
1. zero downtime deployments

N.B. difference between continuous delivery and continuous deployment
