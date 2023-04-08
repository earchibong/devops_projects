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

- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#create-a-local-repository-for-docker">Create A Local Repository For Docker</a>
- 

<br>


## Create A Local Repository For Docker


- get a free trial license for artifactory <a href="https://jfrog.com/start-free/#saas">here</a>

<br>

<img width="1226" alt="repos" src="https://user-images.githubusercontent.com/92983658/230719776-6530e367-9d52-4da6-87a0-e25de475cda7.png">

<br>

- Since we are exploring Local Repositories, type `Docker` in the search box and select the `Docker icon`

<br>

<img width="1221" alt="search_docker" src="https://user-images.githubusercontent.com/92983658/230719907-acd7695a-0740-46d2-9a67-e55132b21a2d.png">

<br>

<img width="1230" alt="docker_artifactory" src="https://user-images.githubusercontent.com/92983658/230719910-39e141d6-28aa-4dc0-a71a-ce3229de1f59.png">

<br>

<img width="1222" alt="docker_minted" src="https://user-images.githubusercontent.com/92983658/230719914-1d38c4db-199a-4d98-9c07-8a8f80dfffdc.png">

<br>

### Create A Virtual Repository
A virtual repository aggregates several repositories under a common URL. You can get artifacts from it but you cannot deploy anything to it.

- Select virtual Repository 
- give the virtual Repository any name you like, 
- click on the create Virtual Repository button

<br>

<img width="1231" alt="virtual" src="https://user-images.githubusercontent.com/92983658/230720599-620c13e4-8cd2-4554-a972-3f3c55acc6f1.png">

<br>

<img width="1224" alt="generic" src="https://user-images.githubusercontent.com/92983658/230720602-8afe7c0d-6c9f-4e68-847b-fd11976647fd.png">

<br>

<img width="1189" alt="generic_name_repo" src="https://user-images.githubusercontent.com/92983658/230720610-4a3780f5-cd38-4dac-a5be-d7ae539d8a5f.png">

<br>

- Scroll down the page to see the local repositories, select which local repository that will be part of the virtual repository. Click on the double arrows to move them.

<br>

<img width="945" alt="new_items" src="https://user-images.githubusercontent.com/92983658/230720697-24e687e9-b8af-4008-b911-ee0382ee8cc4.png">

<br>

<img width="944" alt="included_items" src="https://user-images.githubusercontent.com/92983658/230720700-13d2fac5-a366-4a69-a0c8-9ab8fc549872.png">

<br>


