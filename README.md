# Backfill GitHub Security Advisories with Missing Source Code Links

Approximately 42% of advisories are missing their respective source code link (e.g., [GHSA-m6xf-fq7q-8743](https://github.com/advisories/GHSA-m6xf-fq7q-8743)) when in an ideal situation the advisory should contain the source code link (e.g., [GHSA-qr97-v87p-x965](https://github.com/advisories/GHSA-qr97-v87p-x965)). Here we provide a simplistic method for obtaining a portion of the missing source code links for the respective package within an advisory. 

Source code is contained here: [./find_missing_source_code_link.ipynb](https://github.com/tdunlap607/ghsa-source-code-link-backfill/blob/main/find_missing_source_code_link.ipynb) (~1 hour to complete)

Final Results: 

* CSV: [./missing_source_links_20230111.csv](https://github.com/tdunlap607/ghsa-source-code-link-backfill/blob/main/missing_source_links_20230111.csv)
* JSON: [./missing_source_links_20230111.json](https://github.com/tdunlap607/ghsa-source-code-link-backfill/blob/main/missing_source_links_20230111.json)

Data Info:


|              **Column** |                   **Description** |
|------------------------:|----------------------------------:|
|                id (str) |                           GHSA ID |
| package_ecosystem (str) | Ecosystem for associated  GHSA ID |
|      package_name (str) |    Package for associated GHSA ID |
|       github_repo (str) |           GitHub Source Code Link |

## Ecosystem Breakdown of GitHub Reviewed Advisories and Found Missing Source Code Links

To replicate results checkout commit **d1775b46df53105da55cbd8e78ee19e2251e88c1** within your locally cloned [advisory-database](https://github.com/github/advisory-database). We specifically target the advisories that have been reviewed by GitHub.

|  **Ecosystem** | **GHSA Count** | **Missing Source Links** | **Found Source Links** |
|---------------:|---------------:|-------------------------:|-----------------------:|
|          Maven |          3,080 |           1,364 (44.29%) |            251 (18.4%) |
|            npm |          2,813 |           1,711 (60.82%) |         1,186 (69.32%) |
|           PyPI |          1,494 |             467 (31.26%) |           187 (40.04%) |
|      Packagist |           1224 |             337 (27.53%) |           329 (97.63%) |
|             Go |            807 |             238 (29.49%) |           230 (96.64%) |
|      crates.io |            558 |               34 (6.09%) |            30 (88.24%) |
|       RubyGems |            536 |             261 (48.69%) |           244 (93.49%) |
|          NuGet |            242 |              166 (68.6%) |           143 (86.14%) |
|            Hex |             20 |                 0 (0.0%) |                      - |
| GitHub Actions |              6 |                 0 (0.0%) |                      - |
|            Pub |              3 |                 0 (0.0%) |                      - |
|      **Total** |     **10,783** |       **4,578 (42.46%)** |     **2,600 (56.79%)** |


## Process for identifying the missing source code links

Each ecosystem contains its respective online registry (e.g., PyPI -> https://pypi.org/). Using the package name from a GHSA Advisory we can do a direct lookup in the respective online registry for the package project links (e.g., Source Code/Issues/Homepage) that point to GitHub.

### PyPI Example:
* [GHSA-m6xf-fq7q-8743](https://github.com/advisories/GHSA-m6xf-fq7q-8743)
* Extract package name: bleach
* Parse project links on respective online registry: [https://pypi.org/project/bleach/](https://pypi.org/project/bleach/)
    * Homepage -> [https://github.com/mozilla/bleach](https://github.com/mozilla/bleach)
* Return link that points to a GitHub Repository


### Maven Example:
* Maven projects were non-trivial to handle and we had the least success with. If others have a better solution I'm open to it. 
* Search for the project using the following API (https://search.maven.org/solrsearch/select?q={groupId}+AND+a:{artifactId}&rows=10&wt=json)
    * GHSA: [GHSA-v35c-49j6-q8hq](https://github.com/advisories/GHSA-v35c-49j6-q8hq)
    * Package: org.springframework.security:spring-security-core
    * Search API: [https://search.maven.org/solrsearch/select?q=org.springframework.security+AND+a:spring-security-core&rows=10&wt=json](https://search.maven.org/solrsearch/select?q=org.springframework.security+AND+a:spring-security-core&rows=10&wt=json)
* Match based on the groupId and artifactID parsed from the package name.
* Pull the latest version for the package from the response
    * Latest Version: 6.0.1
* Pull the POM file for the latest version using the following API (https://search.maven.org/remotecontent?filepath={groupId}/{artifactId}/{latest_version}/{artifactId}-{latest_version}.pom)
    * [https://search.maven.org/remotecontent?filepath=org/springframework/security/spring-security-core/6.0.1/spring-security-core-6.0.1.pom](https://search.maven.org/remotecontent?filepath=org/springframework/security/spring-security-core/6.0.1/spring-security-core-6.0.1.pom)
* We then search the POM file for the SCM tag that points to a GitHub repository

    ```xml
    <scm>
        <connection>scm:git:git://github.com/spring-projects/spring-security.git</connection>
        <developerConnection>scm:git:ssh://git@github.com/spring-projects/spring-security.git</developerConnection>
        <url>https://github.com/spring-projects/spring-security</url>
    </scm>
    ```
    * Repo: https://github.com/spring-projects/spring-security


The remaining examples are fairly trivial. The code can be seen here [./find_missing_source_code_link.ipynb](https://github.com/tdunlap607/ghsa-source-code-link-backfill/blob/main/find_missing_source_code_link.ipynb)


# The remaining missing source code links

Our process was only able to obtain ~56% of the missing source code links. The other advisories will need more thought (primarily for Maven packages). Feel free to add more suggestions. 

NPM Notes: 330 of the advisories had matches but we excluded because NPM set the source code repository to (https://github.com/npm/security-holder) due to the package being yanked for malicious activity.