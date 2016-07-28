
There's a maturity scale for automated deployment

1. deploy the latest version of all components in parallel
2. deploy only the components that have changed in a parallel
3. specify simple upgrade paths such that particularly components are advanced in version, tested for compatibility with older dependencies and then only the desired components are deployed. Can only ever go forward.
4. ability to specify any combination of component versions, test this combination together, deploy it
5. run multiple different but compatible combinations in live
6. zero downtime deployments

N.B. difference between continuous delivery and continuous deployment
