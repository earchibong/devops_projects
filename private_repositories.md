# Setting up private repositories and Preparing CI pipelines with Jenkins

In Artifactory, either local, remote or virtual repositories can be set up. It doesn’t matter if it is for docker, helm, or standard software binaries.

Artifactory hosts two kinds of repositories: `local and remote`. Both local and remote repositories can be aggregated under virtual repositories, in order to create controlled domains for artifacts resolution and search.

**Local Repositories:** are physical locally-managed repositories that one can deploy artifacts into. Artifacts under a local repository are directly accessible via a URL pattern such as : `https://tooling.artifactory.archibong.link/artifactory/<local-repository-name>/<artifact-path>`

**Remote Repositories:** serve as a caching proxy for a repository managed at a remote URL (including other Artifactory remote repository URLs). Artifacts are stored and updated in remote repositories according to various configuration parameters that control the caching and proxying behavior. Cached artifacts can be removed from remote repository caches, but you cannot manually deploy anything into a remote repository.

Artifacts under a remote repository are directly accessible via a URL pattern of

`https://tooling.artifactory.archibong.link/<remote-repository-name>/<artifact-path>` or
`https://tooling.artifactory.archibong.link/artifactory/<remote-repository-name>-cache/<artifact-path>`

The second URL will only serve already cached artifacts while the first one will fetch a remote artifact in the cache URL (2nd one) only if it is not already stored in the first URL.

**Virtual Repositories:** or `repository group` aggregates several repositories under a common URL. The repository is virtual in the sense that you can resolve and get artifacts from it but you cannot deploy anything to it.

By default, Artifactory uses a global virtual repository that is available at: `https://tooling.artifactory.archibong.link/artifactory/repo`

This repository contains all local and remote repositories.

## Labs


## Create A Local Repository For Docker
