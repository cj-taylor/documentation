---
layout: post
title: v1.13.7 Armory Release
order: -20180522174834
hidden: false
---

# 05/22/18 Release Notes
{:.no_toc}
> Note: If you're experiencing production issues after upgrading Spinnaker, rollback to a [previous working version](http://docs.armory.io/admin-guides/troubleshooting/#i-upgraded-spinnaker-and-it-is-no-longer-responding-how-do-i-rollback) and please report issues to [http://go.armory.io/support](http://go.armory.io/support).

* This is a placeholder for an unordered list that will be replaced with ToC. To exclude a header, add {:.no_toc} after it.
{:toc}


## Known Issues

#### Spinnaker can't access some files  
**Resolved in [v1.14.65](/release-notes/armoryspinnaker_v1.14.65)**  
Spinnaker changed the user it runs as in a container from `root` to `spinnaker` for security reasons.
See the [Issue 2606](https://github.com/spinnaker/spinnaker/issues/2606) for more information.

Symptoms:
```
unable to read client-key /opt/spinnaker/certs/spinnaker.key for spinnaker due to open /opt/spinnaker/certs/spinnaker.key: permission denied
```

Workaround for this release:  
The work around for this version is to change the ownership of the file in the running container.
Here's an example for setting the permissions to `spinnaker` (uid 100, gid 65533). We're running this on the ec2 host, but the uid and gid carries over.
```bash
chown 100:65533 /opt/spinnaker/certs/spinnaker.key
```
This can be done in [`/opt/spinnaker/bin/secrets`](https://github.com/armory/spinnaker-config-deb/blob/master/deb-config/spinnaker/bin/secrets) or in your userdata.


#### Armory's Spinnaker Configuration doesn't work
We've changed to a new configurator, but there's an issue with the redirect from `https://spinnaker.your_dns_here/armory/config/` to `https://spinnaker.your_dns_here/#/platform/config`. In Chrome, the service worker will get a redirect to the new url, but Chrome security prevents this, therefore the page stays on `/armory/config`. 

Symptoms:
- loading `https://spinnaker.your_dns_here/armory/config/` load the old Configurator
- cannot save configs using `https://spinnaker.your_dns_here/armory/config/`

There's two available solutions:  
- Manually go to [https://spinnaker.your_dns_here/#/platform/config](https://spinnaker.your_dns_here/#/platform/config) for configurations from now on.
- See [How do I uninstall a Service Worker? (for Chrome)](https://stackoverflow.com/questions/33704791/how-do-i-uninstall-a-service-worker) to remove the service worker for `/armory/config`, then the redirect will now work.


#### Cannot deploy AWS encrypted snapshots
**Resolved in [v1.14.65](/release-notes/armoryspinnaker_v1.14.65)**  
If you're trying to deploy an encrypted snapshot, Spinnaker will throw errors because AWS doesn't allow you to respecify encryption.

Symptoms:  
In the ASG logs, you will see:
```
Parameter encrypted is invalid. You cannot specify the encrypted flag if specifying a snapshot id in a block device mapping. Launching EC2 instance failed.
```

Workaround:  
Select `Defaults for selected instance type` on the deploy stage, which will attach an extra, unencrypted volume at runtime.  
![](https://cl.ly/1V2a0u2n3k2e/Screen%20Shot%202018-07-05%20at%2013.27.33.png)

Fix:  
Fixed in the next version.


#### `ssh_private_ip` field in custom packer templates has been deprecated
`ssh_private_ip` has been deprecated in favor for [`ssh_interface`](https://www.packer.io/docs/builders/amazon-ebs.html#ssh_interface). This should only be an issue if you use custom packer templates for baking.

Symptoms:  
In bake logs you will see:
```
bakes fail due to :`1 error(s) occurred: * unknown configuration key: "ssh_private_ip"`
```

Fix:  
You'll need to update your custom packer template, removing `ssh_private_ip`, and instead setting `ssh_interface` to either `public_ip`, `private_ip`, `public_dns` or `private_dns`.



## Highlighted Updates
### Armory
- We've been adding a lot of things preparing for our Armory Kubernetes Installations! Stay tuned!
- For our Configurator users, you can now visit [https://YOUR_SPINNAKER_URL_HERE/#/platform/config/](https://YOUR_SPINNAKER_URL_HERE/#/platform/config/) for our new style.
-❗️We've deprecated Barometer, Armory's canarying solution, in favor for Kayenta. The docs to configure Kayenta can be found here: https://docs.armory.io/user-guides/kayenta/


###  Spinnaker Community Contributions
There have also been numerous enhancements, fixes and features across all of Spinnaker's other services. See their changes here:  
[Spinnaker's v1.7.0](https://www.spinnaker.io/community/releases/versions/1-7-0-changelog)  
[Spinnaker's v1.7.1](https://www.spinnaker.io/community/releases/versions/1-7-1-changelog)  
[Spinnaker's v1.7.2](https://www.spinnaker.io/community/releases/versions/1-7-2-changelog)  
[Spinnaker's v1.7.3](https://www.spinnaker.io/community/releases/versions/1-7-3-changelog)  
[Spinnaker's v1.7.4](https://www.spinnaker.io/community/releases/versions/1-7-4-changelog)  


#### Igor
Igor added polling safeguards for Docker, Jenkins, Gitlab, and Travis. This shouldn't effect upgrades, unless those service pointers are out of date.

**Symptoms:**
- New Docker, Jenkins, Gitlab, and/or Travis artifacts aren't showing up in the job selector.
- New Docker, Jenkins, Gitlab, and/or Travis artifacts don't trigger pipelines.
- An error message from Igor looking like:
```
ERROR 1 --- [readScheduler-3] c.n.spinnaker.igor.docker.DockerMonitor  : Number of items (1155) to cache exceeds upper threshold (1000) in monitor=DockerMonitor partition=gcr
```

**Fix:**
You'll need to add this into your `igor-local.yml`
```yaml
# Added this to igor-local.yml to up the threshold; otherwise
# igor just doesn't bother looking for anything new.
spinnaker:
  pollingSafeguard:
    itemUpperThreshold: 10000  # find the exact number in Igor logs
```




<br><br><br>
## Detailed Updates
### Armory
### Lighthouse&trade; - 86bfefa...d7395c7
 - Hidden gotcha updating yamltools. (#201)
 - Unpin archaic yamltools lib, update code to match. (#200)
 - Test coverage for get_config_version() (#199)
 - Ignore bad files that were deleted before saving... (#198)
 - Add reference to code coverage output (#197)
 - Full coverage cloud storage (#196)
 - Finish testing cloud storage (#195)
 - Start adding tests/coverage for CloudStorage (#194)
 - ENG-1864 due to a non-existent file with None data. (#193)
 - add endpoint to read config (#192)
 - write nginx.conf to filesystem (#191)
 - add logging for nginx config (#190)
 - base64 encode certs (#189)
 - allow adding multiple certs in one call (#188)
 - fix certs endpoint (#187)
 - moving crts and xml files to config dir (#186)
 - Nginx conf (#185)
 - ENG-1831: add endpoint for nginx https cert (#183)
 - add endpoint to upload certs (#184)
 - ENG-1802: Support upload of files related to SAML (#181)
 - add timeout (#167)
 - Configurator mkdirs (#180)
 - Revert "Configurator adds directories if they don't exist. (#178)" (#179)
 - Configurator adds directories if they don't exist. (#178)
 - integration test code coverage (#177)
 - return http 400 if errored (#176)
 - Remove lru cache (#175)
 - Workaround since gate isnt forwarding PUT (#174)
 - Add PUT endpoint for kubeconfigs (#173)
 - Save docker password as file (#172)
 - Was getting some weird flatdict error (#171)
 - Allow custom s3 url (#170)
 - Add placeholder endpoint for adding docker account (#169)
 - Add LRU cache to config calls (#168)
 - Move to a standard opt/dir (#166)
 - Config From Env (#164)
 - make endpoint names consistent (#165)
 - Pin boto3 (#163)
 - ENG-1791: Fix DNS validator (#162)
 - Pin boto3 (#161)
 - busting cache (#160)
 - Monkey patch for realzzzzzzzz (#159)
 - add route to validate redis (#158)
 - Monkey patch (#157)
 - logging formatting (#156)
 - Spring env (#155)
 - make the dir name explicit to let the user know its only for public certs (#151)
 - made "certs/(*).crt" to be visible/uploadable (#150)
 - endpoint to validate if a dns name is valid (#149)
 - Add GET /v1/configs/accounts/kubernetes (#145)
 - Validate Jenkins configuration (#148)
 - updated comments (#147)
 - If the f arg is a file, default filepath (#146)
 - modify clouddriver with the new account (#144)
 - integration tests
 - ENG-1704 healthcheck dinghy (#142)
 - initial commit for kubernetes accounts endpoint (#141)
 - initial commit for kubernetes accounts endpoint
 - remove reversing of configs (#140)
 - ENG-1500 GCS storage support (#138)
 - Initial attempt to creating a K8s configmap in S3 (#137)

### Barometer&trade; - f12a2f8
No Changes

### Dinghy&trade; - dd09163...2015849
 - add bash to docker container (#57)
 - fix integration tests
 - fix tests
 - allow newlines in template actions
 - monorepo support for github (#53)
 - fixed tests
 - fix build
 - JSON validator with line/char error reporting
 - graceful error handling in preprocessor
 - reorganized tests for naming consistency, easy readability & removed repeated code
 - continue processing other affected dinghyfiles if one of them errors out (#51)
 - support for global vars (#50)
 - debug enhancements (#49)
 - Remove unneeded commit status (#48)
 - new endpoint to post a dinghyfile to spinnaker (#42)
 - in pipelineID function now supports vars for appname & pipeline name
 - fix nil pointer (#45)
 - added debugs for auth user header setting (#44)
 - add fix for empty string
 - delete stale pipelines only when variable is set
 - parse elvis operator
 - fix failing test
 - add test
 - add var template function

### Platform&trade; - 41aefad...ff5e90c
 - add bash
 - App config doesn't have Attributes anymore
 - Forgot this file hadn't included "strings". :P
 - ENG-1773: SLA readout not working because Front50...
 - ignore .vscode/ files
 - Missed the one place we didn't use (Get|Post)JSON (#183)
 - ENG-1593 productionize Manual Judgment support (#182)

### Armory Echo  - d0feee3...00219be
 - ignore dinghy endpoints we don't support (#51)

### Armory Deck  - 896d6c8...f57955c
 - fix(nginx) regex match symbol comes first (#333)
 - fix(nginx) allow anything matching /armory/config to redirect to platform/config (#332)
 - add remaining redux tests (#331)
 - ENG-1917: show modal asking user if they want to redeploy instead of browser popup (#330)
 - stop updating saml on https update if saml not enabled (#329)
 - I need somebody to wire the button up again (#324)
 - ENG-1906: add about 50% test coverage for redux (#328)
 - ENG-1904: set default hostname to current url host (#327)
 - remove configuratorAuth and configuratorCerts feature flags (#325)
 - fix redeploy bug (#326)
 - Add configurator tests (#322)
 - Update yaml-tools version. (#323)
 - ENG-1789 Settings and Step By Step Configurator Style Updates (#321)
 - wizard fixes (#319)
 - Add descriptions and tool tips to config pages. (#320)
 - Revert "ENG-1887: write a comment instead of null to empty config files (#316)" (#318)
 - ENG-1714: add skip button to step by step configurator (#317)
 - ENG-1887: write a comment instead of null to empty config files (#316)
 - Prompt for redeploy (#315)
 - fix a bunch of small things (#314)
 - fix stuff (#309)
 - add gitSources to settings-template (#313)
 - Feature flag SAML (#312)
 - move redis from stepbystep to wizard (#308)
 - Trim the result of the filename prompt. (#307)
 - ENG-1860: add saml configurator (#305)
 - Eng 1858 (#306)
 - ENG-1766: add ca certificates to wizard (#304)
 - Redirect to a full URI based on the scheme & host. (#303)
 - fix undefined error (#301)
 - update nginx.conf when saving (#300)
 - ENG-1849: wire up with backend for saving certs and nginx.conf (#299)
 - fix wizard save button (#298)
 - fix bugs (#297)
 - Add fixes to wizard (#296)
 - fix baseurl not being set (#295)
 - Remove empty kube/docker accounts (#294)
 - add textarea for user to enter docker password (#293)
 - Add http/https configuration to wizard (#291)
 - fix docker passwords and kubeconfigs not saving (#292)
 - Add docker registry accounts to cloud accounts (#290)
 - ENG-1755: Add backend input validation for DNS, Jenkins, Redis steps (#286)
 - fix random things (#289)
 - Fix new-file creation not working (#288)
 - add save button (#287)
 - ENG-1779: Fix other bugs (#283)
 - Add tab to configure Redis (#285)
 - Dockerize bin/build.sh (#284)
 - Fix CORS error, save at end (#282)
 - Jack renames components (#280)
 - Add withCredentials back in (#278)
 - Add styling and launch summary (#275)
 - don't try fetching null versions (#274)
 - ENG-1774: Wire Redux up with Jenkins and DNS (#272)
 - Fixes the error in the regex validating repo name (#271)
 - Remove legacy table leftovers. (#270)
 - Use Redux for configurator data store (#269)
 - Jack config accordion switch (#268)
 - ENG-1770: Fix language selector. (#267)
 - Accordion for cloud accounts (#266)
 - ENG-1749 Spinner (#265)
 - Basic config structure (#263)
 - build docs and cleanup (#258)
 - add components to ArmoryConfigWizard (#260)
 - UI new config (#261)
 - Configuration wizard scaffolding (#259)
 - rename unstable builds to edge builds (#247)
 - ENG-1754: Refactor ArmoryStepByStepConfig into separate components (#256)
 - New config jack (#255)
 - Component to add Kubernetes accounts (#252)
 - Combine apps dropdown component replacement (#250)
 - Rejigger the SLA help text to work in 1.7.x (#249)
 - Something is wrong with the help registry. Removing it (#248)
 - Combine apps (#246)
 - ignore build.properties, its only for jenkins/debugging (#245)
 - use a staging area for building node_modules (#244)
 - 💯 Improve test data structures for step by step (#243)
 - Refactor cloud config into its own tree of components (#242)
 - 🌎 Move all components to globals/styles.scss declarations (#241)
 - indicate the right way of starting deck-armory (#240)
 - Step by step config (#238)
 - ENG-1707 fix sla readout (#239)
 - 🌎 Add Globals
 - seperate out install and run scripts 🚀 (#236)
 - add production builds to webpack (#235)
 - add comment for upgrade to next release branch (#234)
 - Add nice big comment about disabled features. (#233)
 - Wow. Needed to require jquery! (#232)
 - Comment out the Armory modules w/ stages (#231)
 - add helpful reminder on how to run prod version of configurator locally (#229)
 - i think this fixes the configurator, maybe (#228)
 - Import deck-kayenta, clean up how we do config (#227)
 - unpin deck modules, the job is now succesful (#225)
 - removing kayneta hack (#224)
 - Feature/upgrade 1.7 (#222)
 - remove duplicated jekins call to deck-modules (#220)
 - allow building from oss stable or unstable (#219)
 - HACK! install kayenta@v0.0.40 (#218)
 - Fix dead link (#217)
 - unpin deck_modules, we're actually pinned to the latest 1.6.x (#216)
 - ⬆️ Modal (#215)

### Armory Gate  - 0191333...d95a452
 - need openssl for k8s to convert pem keys to p12 (#11)

### Packager - f5ef78e...5e4e88e
 - new base version = 1.13.x (#344)
 - feat(jenkins) add example to version number format (#343)
 - add feature flags (#338)
 - remove call to bin/archive-artifacts.sh, moved to bin/push-packages (#341)
 - version, not versions (#342)
 - archive artifacts to s3 (#340)
 - rename artifact for readability (#339)
 - archive artifacts to s3 (#337)
 - unpin deck-armory (#336)
 - latest deck-armory build (#335)
 - pinning because jenkins deck-armory builds are breaking (#334)
 - Pin deck-armory to last good version before CORS (#333)
 - remove orca ping (#332)
 - Wait for disable fix (#331)
 - Pin orca 4 wait fix (#330)
 - Looks like bintray started .gzipping Packages... (#328)
 - 😫 many levels of json (#327)
 - please stop moving it all the way down 😞 (#326)
 - missed reference (#325)
 - .js was intended to mean json (#324)
 - its string string string, no colon (#323)
 - it should be running trigger tests in trigger tests (#322)
 - add whitespace to rendered resolve.env (#321)
 - add jenkins build number to version.manifests (#320)
 - Default these fields to blank if not set (#319)
 - Adding Keiko config for Orca (#318)
 - Maybe? (#317)
 - Kayenta moved Redis queue config under Keiko (#316)
 - Re-enable Kayenta in the yaml. (#315)
 - OSS Kayenta (#313)
 - fixed dinghyfile and modules (#314)
 - app name (#312)
 - Update dinghyfile (#311)
 - Update dinghyfile (#310)
 - Update dinghyfile (#309)
 - dinghyfile! (#308)
 - add logging driver in docker-compose.yml for dinghy (#307)
 - start with fresh resolve env on each reboot (#304)



###  Spinnaker Community Contributions
### Clouddriver  - 6dfcf4f...f432528
 - fix(provider/kubernetes): sub manifests by namespace (#2641) (#2642)
 - fix(provider/kubernetes): support looking up non-namespaced manis (#2634) (#2635)
 - fix(cache): metrics support may be null (#2622) (#2623)
 - fix(provider/kubernetes): v2 check job failed (#2608) (#2611)
 - fix(provider/google): remove json key from logs during init (#2606)
 - fix S3 being unable to talk to custom API endpoint (#2604)
 - fix(provider/gce): Merge cache relationships in onDemand resolution. (#2517) (#2579)
 - fix(provider/kubernetes): lookup of oldest server group (#2573) (#2575)
 - fix(provider/kubernetesv2): fix secretVolumeReplacer (#2565) (#2572)
 - fix(provider/kubernetes): v2 search omit null (#2568) (#2570)
 - fix(provider/kubernetes): support key ref secrets & cms (#2566) (#2567)
 - fix(provider/kubernetes): fix v1 red/black pod restarts (#2553) (#2554)
 - fix(provider/ecs) Changes to allow ECS to work with AWS provider. (#2500) (#2527)
 - fix(provider/kubernetes): v2 Fix support for clusterrolebindings (#2514)
 - refactor(provider/kubernetes): lay groundwork for "multi-kind" caching agents (#2509)
 - feat(titus): use pagination cursors (#2512)
 - fix(provider/kubernetes): properly synchronize around kind/apiVersion (#2511)
 - chore(provider/gce): Instrument cluster caching. (#2510)
 - fix(core): Optimize `CleanupPendingOnDemandCachesAgent` (#2508)
 - chore(provider/gce): Instrument svg caching agent onDemand handlers. (#2507)
 - fix(titus): Align default caching agent timeout with AWS (every 30s) (#2506)
 - fix(google): Don't fetch image relationships (#2502)
 - feat(provider/kuberntes): deploy priority (#2504)
 - chore(*): Bump spinnaker-dependencies to 1.151.0 (#2505)
 - feat(core): Instrumented Jedis clients (#2503)
 - fix(caching): caching agent metricsSupport being null obscures original exception (#2501)
 - refactor(artifacts): change fetch verb type (#2492)
 - fix(provider/ecs): Update AmazonClientProvider to pass NetflixAmazonCredentials (#2497)
 - feat(aws/loadbalancing) load balancer attributes
 - chore(cats): metric to count onDemand requests (#2499)
 - feat(aws/enchancement) Deploy encrypted EBS vols (#2490)
 - fix(metrics/prometheus): dots are not allowed in labels in prometheus (#2437)
 - fix(openstack/javadoc): Add placeholder javadoc class comment to make gradle javadoc happy. (#2495)
 - fix(amazon/loadBalancer): If multiple subnets in an az, pick the one with most ip space (#2493)
 - chore(build) single-thread ci builds (#2494)
 - feat(provider/kuberentes): expose kubectl executable option (#2486)
 - fix(provider/openstack): corrects behavior of status checkers for openstack operations (#2482)
 - fix(provider/gce): Support regional MIGs in validators. (#2488)
 - fix(groovy): Explicitly return null to avoid class cast exception (#2489)
 - fix(provider/gce): Tolerate locations with no CpuPlatforms entry. (#2487)
 - fix(aws): Make deploy atomic operation more resilient to AWS failures (#2463)
 - fix(aws): Explicitly terminate instances when resizing to zero (#2485)
 - fix(core): Handle reading task results from previous redis delegate (#2484)
 - feat(artifacts): Update google deploy validator to handle artifacts (#2483)
 - fix(provider/gce): Fix deploying instance without autohealing (#2481)
 - feat(artifacts): GCE deploy can consume image artifacts (#2477)
 - chore(docker): run alpine-specific useradd command (#2480)
 - chore(docker): run as spinnaker (#2479)
 - fix(reservations): Support for `Windows BYOL (Amazon VPC)` (#2478)
 - feat(provider/gce): Support nested health checks in autoHealingPolicy. (#2476)
 - feat(provider/kubernetes): allow jobs & pods to "fail" (#2469)
 - feat(artifacts) Implement S3 artifacts (#2468)
 - fix(mdc): fix propagation of exection id (#2475)
 - feat(provider/kubernetes): register batchv1 api (#2473)
 - fix(amazon/deploy): Allow deploying with target groups with two dashes (#2474)
 - fix(provider/kubernetes): guard against empty api & kind (#2472)
 - feat(titus): pass an explicit list of task states instead of relying on titus master for active vs. archived tasks (#2471)
 - fix(core): Log when results are being added to a task (#2470)
 - cleanup(aws): Avoid scheduling `ReconcileClassicLinkSecurityGroupsAgent` (#2466)
 - feat(jobs): allow jobs to be canceled (#2462)
 - fix(provider/kubernetes): validate correct delete coords (#2467)
 - feat(provider/kubernetes): v2 cache & surface events per-resource (#2464)
 - feat(web): Include `capacity` in `ServerGroupViewModel` (#2455)
 - chore(build): add debug flag to clouddriver build (#2460)
 - fix(titus): stopped titus jobs should be marked as failures (#2461)
 - fix(provider/kubernetes): v2 Avoid NPE for unsupported k8s kinds (#2456)
 - feat(core): allow per account ui skins (#2459)
 - fix(entitytags): Handle a `*` entityRefAccount (#2458)
 - fix(provider/kubernetes): v2 Fix support for rolebindings (#2457)
 - feat(titus): titus cloud provider
 - feat(titus): titus cloud provider
 - titus: return awsAccount for serverGroups endpoint
 - titus: fix casing of loadbalancingEnabled
 - titus: surface awsAccount and host ip
 - Titus: return instance ip, cluster and server group in instance details for generating insight links
 - titus: fixing compilation issues
 - Titus: code reformatting and copyright headers
 - titus: refactor channel instantiation code
 - titus: remove netflix grpc protoc gen wrapper
 - titus: add a a docker section in buildInfo so we can build more detailed links
 - titus: fix issue where sometimes region is null on health key lookup
 - Titus: improve logging and retry on UNAVAILABLE
 - Titus: fix instance sidebar not showing for titus
 - Titus: conditionally look up instance health only if a target group is provided for clusters view
 - Titus: undo healthcheck lookup on clusters
 - Titus: fix health check format for target groups
 - Titus: removes v2 REST based API
 - ignore retrofit package for nebula publish
 - Titus: fix aws account lookup to return null if account is not found
 - Titus: return alb health in cluster and instance lookup
 - fix(titus): fix policy creation on titus deploys
 - Titus: also return targetGroups for clusters
 - Titus: restore awsVpcId method
 - Titus: index target groups
 - utilize updateAutoScalingPolicy command for Titus policies
 - Titus: use shaded grpc netty
 - Titus: set metatron sigs in both attributes and job labels
 - Titus: fix issue where clone will set inService to previous server group's inService flag instead of false
 - delete entity tags on Titus job deletion
 - Titus: fix retry logic to now really only retry on find and gets
 - Titus: bump deadline to 60 seconds
 - Titus: correctly index migration policy
 - Titus: migration policy changes
 - Titus: default instanceId to id as this field no longer exists in v3 engine
 - Titus: record migration policy set in description
 - Titus: return done jobs for getJob method for v3 engine
 - try/catch enable/disable autoscaling processes on Titus server group ops
 - include jobId when disabling titus server groups
 - Revert "Titus: return done jobs for getJob method for v3 engine"
 - Titus: return done jobs for getJob method for v3 engine
 - (titus): toggle scaling enabled on enable/disable; guard against async policy upserts
 - restore titus gRPC definitions
 - fix NPE in target group logic
 - Titus - fix issue where image name was set incorrectly on clone
 - fix(titus clone): remove env variables if empty on clone
 - do not default titus scaling policy unit to None
 - Titus - clean up deploy handler
 - Titus - support for adding a alb target group
 - handle target tracking policies on titus clone
 - Titus: change retry interceptor behavior to retry all the time on find and get methods and not on other methods like set or create
 - titus: fix indexing and posting of efs volumes
 - handle titus batch deployment jobUri field
 - include jobUri in titus deploy output
 - Never trust what IntelliJ tells you are unused imports
 - Adjust titus retry intervals
 - perf(titus): Optimize out the per-security group lookups
 - feat(titus): Force cache refresh support for server groups
 - Re-enabling titus clone behavior (take 2)
 - Re-enabling titus clone behavior
 - Temporarily revert the titus clone behavior
 - Titus - ensure soft and hard contraints are unique when copying from source job
 - titus - handle clone operation correctly
 - Ensure that titus server groups have non-null 'instances' collection
 - titus - add titus spectator metrics interceptor to autoscaling client
 - titus - adds titus account and region to netflix grpc interceptor calls
 - titus - adds the spectator metrics interceptor
 - perf(titus): Support for loading minimal clusters
 - titus - defaults inService to true when not set as grpc does not allow nulls
 - fix tests
 - (titus) enable autoscaling on a per-account/per-region basis
 - Titus - support useSourceCapacity
 - Titus - change v3 retry to 10 minutes
 - titus: set target tracking unit to "None" if not specified
 - Titus - expose job and task endpoints for v3
 - titus - adds a flag to set autoscaling enabled
 - (titus): allow autoscaling wherever eureka is configured
 - include status in titus scaling policy details
 - check for eureka before creating autoscaling client, do not copy deleting policies
 - include id in titus scaling policies
 - Titus - sets capacity group for v3 grpc
 - Support eviction of stale Titus server group keys
 - fix test
 - implement autoscaling crud for titus
 - titus - retry on deadline exceeded or unavailable
 - titus - allow users to specify migration policy
 - titus - fix incorrect constructor for v3
 - titus - read from eureka
 - titus - first step of retry interceptor - just moves the deadline into an interceptor
 - titus - adds an interceptor that pumps metrics into spectator from grpc v3
 - titus - add targetHealthyDeployPercentage to enableDisable operation
 - Titus - add a deadline to connection requests in v3 grpc and bump api-definition versions
 - titus - set apiversion at the regional level rather than at the account level
 - Fix Run Job Stage for Titus v3
 - Fix Run Job Stage for Titus v3
 - titus grpc load balancing
 - titus: v3 api hard constraints and zone field
 - Titus: only poll once for every job and map v3 states to v2
 - Titus: add requiredGroupMembership to devvpc
 - Titus v3: get api url from config file
 - Titus v3: handle missing titus health states
 - Titus v3: fix find job by name and set instance count to desired
 - Titus v3 api - fix serialization issue with job description request
 - Titus v3 api - fix broken tests, change field name to apiVersion
 - Titus v3 api - additional changes to map response from grpc to old format
 - Bump to 1.6722 (restrict perf changes to aws)
 - Implement TitusClusterProvider.getCluster()
 - Titus API - bring up to date with latest version
 - Titus v3 api - metatron creds
 - fully remove OnDemand from TitusClusterCachingAgent instead of stubbing it out
 - Make destroy titus job noop on non-existent job
 - enable v3 api for clouddriver.yml
 - Merge branches 'master' and 'titus-v3' of ssh://stash.corp.netflix.com:7999/spkr/clouddriver-nflx into titus-v3
 - Titus v3 api integration
 - titus: return task.host for host ip
 - 1.614.0 & fix titus instance retrieval
 - 1.614.0 - include awsAccount, stack and region when indexing titus instances
 - fix: test dependencies for clouddriver-titus
 - 1.611.0 move titus into clouddriver-nflx
 - chore(aws): Increasing logs for scaling policies in case of AWS failure (#2453)
 - fix(provider/kubernetes): v2 Use artifact replacers for DaemonSets (#2445)
 - fix(artifacts): Fix broken Docker image name matching and add tests (#2442)
 - fix(provider/kubernetes): v2 Fix artifact replacers for StatefulSets (#2452)
 - fix(cache): remove hardcoded scanSize (#2451)
 - fix(cache): Remove anonymous noop `SearchableProvider` (#2450)
 - fix(aws): Only log warrning if `lastModified` not present and non-empty (#2449)
 - chore: add missing copyright headers (#2446)
 - fix(reservations): Enable the additional allocation attributes for `v3` (#2444)
 - fix(docker): logs the registry address when a tag cannot be loaded (#2441)
 - feat(provider/aws) Fetch image information based on image id (#2285)
 - test(aws): add validateCapacity test cases
 - fix(aws): support for null values in validateCapacity
 - fix(entitytags): Filter out any entity tags that have not been modified (#2438)
 - fix(provider/kubernetes): tolerate per namespace failures (#2435)
 - feat(reservations): Backport 'v4' to 'v3' (#2436)
 - feat(aws): add support for resizing ASGs with optional min/max/desired (#2430)
 - fix(provider/kubernetes): tolerate null service account (#2434)
 - chore(provider/kubernetes): remove unused creds code (#2433)
 - fix(amazon/loadBalancer): Allow updating protocol on a listener (#2432)
 - fix(amazon/loadBalancers): Allow setting target group stickiness.enabled to false (#2431)
 - fix(provider/kubernetes): allow empty manifest in doc (#2427)
 - fix(provider/kuberentes): v1 fix scaleTargetRef apiVersion in HPA (#2249) (#2418)
 - feat(provider/kubernetes): v2 allow service account auth (#2429)
 - feat(reservations): V4 reservation report that proportionally covers shortfalls (#2428)
 - fix(reservations): Add support for m5.large multiplier (#2426)
 - fix(artifacts): Make Docker image name matching more accurate (#2412)
 - fix(provider/kubernetes): v2 Fix stability check for DaemonSet (#2421)
 - feat(provider/kubernetes): v2 Add support for RBAC resource kinds (#2419)
 - fix(provider/kubernetes): v2 Fix validation to not require namespace (#2422)
 - chore(build): Parallel gradle (#2423)
 - fix(provider/kubernetes): fix K8s provider outside proxy environment (#2413)
 - fix(provider/kubernetes): v2 Allow ":" in keys (#2420)
 - fix(core): Switch task IDs to be UUIDs (#2415)
 - feat(kubernetes): allow cluster provider to return subclasses of KubernetesV2ServerGroup (#2414)
 - feat(artifacts): Add a "no auth" http artifact account where needed (#2417)
 - feat(artifacts): base64 embedded artifact downloader (#2416)
 - feat(provider/kubernetes): v2 restrict namespaces for uploading manifests (#2410)
 - feat(cache/aws) Add relation between server group->ami and ami->server group
 - fix(provider/openstack) - handle operations on stacks that were deployed with the previous HEAT template. (#2375)
 - fix(elasticsearch): Proactively cleanup indexes when in a bad state (#2411)
 - feat(provider/kubernetes): v2 run validators for deloyment and deletion of manifests (#2409)
 - fix(provider/kubernetes): v2 statefulset services don't need to be unique (#2406) (#2408)
 - fix(dynomite): Cant use evalsha (#2407)
 - fix(provider/kubernetes): v2 statefulset services don't need to be unique (#2406)
 - fix(metrics): typo in kubernetes metric tag (#2405)
 - fix(provider/aws): Correctly handle AmazonAutoScalingExceptions (#2404)
 - chore(provider/aws): Reduce common not found exceptions (#2403)
 - fix(provider/aws): LifecycleNotification JsonMappingException (#2402)
 - feat(cats): Dynomite-friendly ClusteredSortAgentScheduler (#2393)
 - feat(aws): Allow self-referencing security group clone ops (#2371)
 - fix(provider/kubernetes): don't return empty apps in app list (#2400) (#2401)
 - fix(provider/kubernetes): don't return empty apps in app list (#2400)
 - feat(artifacts): versoin is not required to load an aritfact (#2396) (#2399)
 - chore(core) Move image interface from openstack to core (#2373)
 - feat(provider/kuberentes): restrict caching by kind (#2398)
 - feat(aws): allow setting targetType when creating a target group (#2397)
 - feat(artifacts): versoin is not required to load an aritfact (#2396)
 - feat(artifacts): Add support for HTTP artifacts (#2395)
 - fix(provider/aws): Fix groovyism in call to elbv2 region scoped provider call (#2391)
 - feat(redis): Retry logic on failed connections in cache & task repository (#2389)
 - fix(provider/kubernetes): upsertScalingPolicy will patch existing autoscaler instead of re-creating (#1893)
 - fix(provider/gae): Fix artifact deploy switch statement. (#2387)
 - fix(provider/gae): Fix artifact deploy switch statement. (#2388)
 - chore(cherry pick): fix cherry-picked code based on refactor (#2386)
 - feat(provider/kubernetes): v2 Add support for Kubernetes jobs (#2380) (#2385)
 - fix(amazon): support long AMI ID patterns in image lookup (#2381)
 - chore(*): Bump spinnaker-dependencies to 0.142.1 (#2382)
 - fix(eureka): `getInstanceToModify` calculation was incorrect with multiple health statuses (#2384)
 - feat(aws): Track the instanceIds being disabled (#2383)
 - feat(provider/kubernetes): v2 Add support for Kubernetes jobs (#2380)
 - feat(provider/kubernetes): v2 support pv & pvc deploys (#2378)
 - refactor(provider/kubernetes): v2 rename deployer to handler (#2377)
 - fix(aws): Continue if launch configuration already exists (#2376)

### Deck  - fd93a17...e2296bd
 - fix(ssl/apache2): Fix for ports.conf.gen (#5254) (#5277)
 - chore(deck-kayenta): bump package to 0.0.43 (#5217)
 - fix(provider/ecs): enable ecs provider within deck. (#5190) (#5208)
 - fix(provider/kubernetes): hide bake manifest stage from k8s v1 provider (#5206)
 - feat(provider/kubernetes): run job node selector (#5181) (#5191)
 - chore(core): bump package to 0.0.194 (#5178)
 - feat(core): show execution timestamp on hover (#5177)
 - chore(core) fixed imports (#5176)
 - fix(core/jenkins): show long choice params on multiple lines in dropdown (#5175)
 - chore(npmignore): remove yalc workaround for 'history' and 'changes' (#5173)
 - fix(core/execution) fix wrong parameter passed to getExectionsForConfigIds (#5171)
 - feat(redblack): UI option for 'Rollback on Failure' (#5102)
 - feat(bake/manifest): support overrides (#5172)
 - chore(core): bump package to 0.0.193 (#5170)
 - fix(core): fix awaiting judgment overflow, prevent flicker on active stages (#5169)
 - chore(core): bump package to 0.0.192 (#5168)
 - fix(core/canary): do not transform lazy canaries; smarter stage hydration (#5167)
 - chore(amazon/titus): bump packages to 90, 10 (#5166)
 - chore(core): bump package to 0.0.191 (#5165)
 - fix(core): fix execution rendering on lazy executions (#5164)
 - fix(core): Fix pipeline configurer for new cache format (#5163)
 - fix(amazon): ask if reboot should consider amazon health only (#5156)
 - refactor(*): De-angularize caches (#5161)
 - fix(core): extract target accounts from config (#5155)
 - feat(kubernetes) remove upper limit for CPU Target on resize (#5159)
 - remove a welcome message (#5158)
 - chore(titus): Bump to 0.0.9
 - chore(core): Bump to 0.0.190
 - chore(amazon): Bump to 0.0.89
 - refactor(titus/loadBalancer): Refactor to use AmazonLoadBalancersTag
 - refactor(core/entityTag): Convert clusterTargetBuilder.service to plain JS
 - refactor(core/naming): Convert angular naming.service to plain NameUtils
 - chore(core): bump package to 0.0.189 (#5153)
 - fix(core): fix manual judgment render, active execution refresh (#5152)
 - chore(core): bump package to 0.0.188 (#5150)
 - refactor(core): lazy load execution details (#5141)
 - fix(core/pipeline): Fix rendering of execution graphs after tslint --fix (#5148)
 - fix(core/pipeline): Fix rendering of pipeline graphs after tslint --fix (#5147)
 - fix(core/nav): use 'undefined' instead of 'null' for empty select value (#5146)
 - fix(eslint): enable es2017 parsing to allow trailing commas in function declaration arguments (#5144)
 - fix(core/wizard): Fix header icons at smaller widths
 - chore(tslint): manually fix lint errors that don't have --fix
 - chore(tslint): ❯ npx tslint --fix -p tsconfig.json
 - chore(tslint): Add prettier-tslint rules, manually fix lint errors that don't have --fix
 - Just Use Prettier™
 - chore(prettier): add prettier git hook
 - feat(bake/manifest): bake manifest stage config (#5128)
 - chore(core): bump package to 0.0.187 (#5140)
 - fix(core): skin selection (#5139)
 - feat(provider/appengine): add halyard feature flag for container image url deployments (#5126)
 - chore(docker): grant www-data access to apache logs (#5135)
 - chore(titus): bump package to 0.0.8 (#5138)
 - chore(core): bump package to 0.0.186 (#5137)
 - fix(core): make nav category headings clickable in Firefox (#5136)
 - fix(titus/loadBalancer): Fix load balancer icon (#5134)
 - fix(titus/deploy): Show loading spinner for target groups in deploy config (#5133)
 - fix(build): turn off minification for deck-kayenta (#5132)
 - chore(docker): make sure apache dirs exist (#5131)
 - chore(docker): correct permissions on lib and run (#5130)
 - chore(docker): allow staged files to be edited by www-data (#5129)
 - chore(docker): change ownership while still root user (#5127)
 - chore(docker): fix ownership of docker directory (#5125)
 - fix(core): do not cache non-json responses in local storage (#5124)
 - chore(deck-kayenta): bump package to 0.0.40 (#5123)
 - fix(artifacts): deleting expected artifact removes stale references (#5107)
 - chore(titus): bump package to 0.0.7 (#5122)
 - chore(canary-v2): Add placeholder config for kayenta. (#5120)
 - chore(core): bump package to 0.0.185 (#5121)
 - fix(core): remove :focus styling on nav dropdowns (#5119)
 - fix(core): toggle disabled to false if needed on application refresh (#5114)
 - fix(titus): keep instance/job title on same line as icon (#5118)
 - chore(docker): run as www-data rather than spinnaker (#5117)
 - fix(titus): fix alignment of migration config fields (#5116)
 - chore(docker): deck has a special base image... (#5115)
 - chore(docker): alpine-specific adduser (#5113)
 - chore(amazon/titus): bump packages to 0.0.88, 0.0.6 (#5112)
 - feat(aws): offer rollback when user enables an older server group (#5109)
 - chore(docker): run as spinnaker (#5111)
 - feat(appengine): support publishing appengine module as an npm package (#5110)
 - feat(provider/kubernetes): pipeline manifest events (#5096)
 - feat(provider/gce): Support nested health checks in autoHealingPolicy. (#5108)
 - feat(artifacts) Implement S3 artifacts (#5099)
 - chore(kayenta): Upgrade to version 0.0.38 (#5106)
 - chore(core): Bump to 0.0.184 (#5104)
 - feat(core): Add Pager UI for finding and paging application owners (#5103)
 - chore(titus): bump package version to 0.0.5 (#5101)
 - feat(titus): Support rollback of disabled server groups (parity with aws) (#5100)
 - feat(webhooks): support artifact production (#5090)
 - chore(amazon): bump package to 0.0.87 (#5097)
 - chore(core): bump package to 0.0.183 (#5095)
 - refactor(core): allow dataSources to specify a required dataSource (#5094)
 - style(core/navigation): remove dead zones from nav dropdown (#5087)
 - Updating to use auto-generated files from icomoon.app (#5086)
 - fix(core/search): dont throw when SETTINGS.defaultProviders is null-ish (#5093)
 - feat(provider/kubernetes): events in manifest details (#5092)
 - chore(kubernetes): bump package to 0.0.7 (#5091)
 - chore(core): bump to 0.0.182 (#5089)
 - feat(core/pagerDuty): Add pagerDuty feature to enable/disable adding pd keys (#5088)
 - feat(aws): Support rollback of a disabled server group (#5077)
 - feat(kubernetes): use v2 load balancer and security group transformers (#5085)
 - fix(artifacts): Set the name field on the default github artifacts (#5083)
 - chore(core): bump package to 0.0.181 (#5084)
 - feat(core): move from provider version UI implementation to skins (#5080)
 - chore(kubernetes): bump package to 0.0.6 (#5082)
 - chore(kayenta): bump kayenta to 0.0.35 (#5081)
 - chore(core): bump package to 0.0.180 (#5079)
 - fix(core): set initial state of skipWaitCustomText on wait stage (#5078)
 - Add the ability to enable infrastructureStages (#5073)
 - chore(core/amazon): bump packages to 0.0.179, 0.0.86 (#5076)
 - fix(core): fix submenu alignment, sync running badges (#5075)
 - fix(amazon): show launch configuration for empty server groups (#5074)
 - feat(core): allow custom warning when users skip wait stage (#5069)
 - fix(provider/kubernetes): v2 incorrectly showing runjob stages (#5072)
 - chore(core): bump package to 0.0.178 (#5071)
 - feat(core): navigate to first item in nav dropdown by default on click (#5070)
 - chore(core): bump package to 0.0.177 (#5068)
 - style(core): add running-count style to third-level nav badge (#5067)
 - chore(core): bump package to 0.0.176 (#5066)
 - style(core): tweak navigation icon sizes/placement (#5065)
 - chore(core): bump package to 0.0.175 (#5064)
 - feat(core): implement categorized navigation (#5063)
 - chore(karma): do not use jenkins reporter (#5062)
 - refactor(core): move app refresher, pager duty buttons to components (#5061)
 - fix(core-package): Add explicit do-not-ignores to .npmignore so yalc doesn't exclude them (#5060)
 - chore(package): use --mode=production in gradle build (used by jenkins)
 - chore(package): minify package bundles in production mode only
 - feat(core): add categories for application data sources (#5058)
 - chore(core): bump package to 0.0.174 (#5057)
 - chore(titus): bump package version to 0.0.4 (#5056)
 - Oneliners (#5055)
 - feat(titus): Support specifying a percentage of instances to relaunch (#5054)
 - fix(titus): restore load balancer tag icon (#5053)
 - fix(core): avoid overflow on Applications actions menu (#5051)
 - fix(core/securityGroups): ensure security groups view renders when ready (#5052)
 - fix(core): load jQuery before Angular (#5050)
 - fix(core): avoid excessive rerender on popovers with templates (#5049)
 - chore(webpack): Print webpack mode (production/development) to console (#5047)
 - fix(core/entityTag): Fix notifications popover re-mounting during each render.
 - fix(halyard): Do not assume latest halyard changes. (#5046)
 - chore(kayenta): Upgrade kayenta to v0.0.33. (#5043)
 - fix(titus): handle strategy change (#5044)
 - chore(core/amazon): bump core to 0.0.173, amazon to 0.0.85 (#5042)
 - chore(karma): remove mocha reporter in favor of dots
 - fix(*): update icons to font-awesome 5 equivalents (#5040)
 - style(core): override icon for sitemap (#5038)
 - fix(core/pipeline): Update cancel execution icon for FA-v5.
 - chore(halyard): Initial support for configuring deck for kayenta via halyard. (#5037)
 - chore(karma): fix lint errors in tests
 - chore(karma): use 'dots' reporter for less clutter
 - chore(karma): Update karma config for webpack 4
 - refactor(webpack): rename webpack.common.js to webpack.config.js
 - chore(webpack): update webpack configurations for webpack 4
 - chore(webpack): webpack 4.x libs and karma updates
 - chore(core): bump package to 0.0.172 (#5035)
 - fix(executions): fontawesome renamed repeat icon to redo (#5034)
 - chore(core/amazon): bump core to 0.0.171, amazon to 0.0.84 (#5033)
 - fix(artifacts): remove unreachable error markup (#5031)
 - fix(core): do not render application list if all are filtered out (#5032)
 - chore(core): upgrade to font-awesome 5 (#5029)
 - fix(bake): Fix JavaScript error on bake stage load (#5027)
 - feat(artifacts): Support embedded/base64 artifact type (#5021)
 - fix(tooltip): updating conditional on expression tooltip) (#5028)
 - fix(amazon/loadBalancers): Don't allow to create if duplicate target group names
 - fix(amazon/loadBalancers): Disable editing target group names
 - fix(core/pipeline): Fix closing of "new pipeline" modal using X or Cancel buttons (#5026)
 - chore(amazon): bump package to 0.0.83 (#5024)
 - fix(amazon/serverGroup): Add spelLoadbalancers to IAmazonServerGroupCommand (#5025)
 - chore(core): bump package to 0.0.170 (#5023)
 - refactor(core/task+amazon/common): Reactify UserVerification and AwsModalFooter (#5015)
 - feat(start.sh): improve nvm handling in start.sh (#5018)
 - fix(amazon/deploy): Do not destroy SpEL based load balancers in deploy config (#5016)
 - fix(amazon/securityGroup): ingress selector would try to eagerly load vpcs so set empty default (#5017)
 - fix(amazon/serverGroups): Unable to open "Edit Scheduled Actions" modal (#5020)
 - chore(titus): bump package to 0.2
 - feat(provider/kubernetes): trim runJob logs from executions until explicitly requested (#5014)
 - fix(titus): fix linting and linking errors to open source titus module again
 - preparing to oss titus again
 - titus - change Urls from spinnaker api to titus UI
 - titus: set useprovider for loadbalancers
 - titus - expose target group health check
 - titus - adds clearer message about only using ip type load balancers
 - titus - show target groups on clone command
 - titus - clean up account and region changed subjects for load balancer selector
 - titus - show load balancer tag for titus clusters
 - titus - alb UI support
 - do not track titus policies by id
 - Titus auto scaling - link to configbin
 - Titus: links to titus ui are now more specific
 - Titus: fix incorrect links
 - Titus: change titus ui links to new ui and remove stack from titus-ssh as this is no longer needed
 - amazon to 65, less gross disabled message on titus details
 - Titus: enable the migration policy field
 - fix(titus/deploy): Stop clobbering imageId if it is provided in the config. Fixes SPIN-3219.
 - catch not found server group promise
 - fix track by on target tracking policy, correct update/create link
 - fix target tracking upsert
 - clean up titus server group wizard
 - use upper camel on titus step policy editing
 - add GA to insight links
 - Titus - adds efsRelativeMount field
 - add ConfigBin link
 - handle additional titus accounts
 - Now with more semicolon.
 - feat(titus): Rollback UI
 - convert find ami execution details to react
 - remove predefined metrics from titus scaling config
 - titus - gpu field
 - Fix lint warnings
 - enable titus autoscaling per-region
 - capacity dialogs from aws to titus
 - refactor(provider/titus): Remove duplicate execution details
 - catch modal rejections
 - add titus target tracking policy support
 - proxy titus api links through spinnaker
 - add titus autoscaling support
 - Show results of run job properties file
 - Removed colors.less and added support from custom variables
 - refactor(*): Remove angular-loader in favor of using `.name` explicitly
 - feat(titus): adding timeout override to run job stage
 - Fixed color variables to use styleguide color vars
 - convert UI to struts
 - adds terminate and shrink to titus sidebar
 - (titus) - fix issue where capacity group does not show on the sidebar if min == max
 - discovery application might not be same as application
 - add Discovery Info links to instance details
 - fix(titus): append insight menus to body
 - add custom, redblack strategies to titus
 - updates for new uirouter
 - fix(entityTags): Render alerts in netflix-aws and titus details
 - initial cutover from oss
 - fix(amazon/loadBalancers): Remove cert and ssl policies when submitting http listener (#5011)
 - fix(amazon/loadBalancers): Show target group sticky status appropriately (#5010)
 - fix(core): count down remaining time in wait stage details (#5009)
 - feat(kubernetes): Expose use source capacity for deploy (#2543) (#5008)
 - chore(amazon): bump package to 0.0.82 (#5007)
 - chore(core): bump package to 0.0.169 (#5006)
 - feat(core): Support warning message type for stages (#5005)
 - fix(core): render remaining wait on wait stage popover (#5004)
 - fix(core): ensure stage comments are in string format (#5003)
 - fix(core): allow expressions in pipeline stage application field (#5002)
 - feat(amazon/serverGroup): Change default capacity constraint to off (#5001)
 - fix(core/help): Allow target="_blank" in links in help popover (#5000)
 - fix(core): fix build link to parent pipeline (#4999)
 - feat(core): omit unsupported cloud provider data from clusters (#4998)
 - refactor(amazon/core): move targetHealthyPercentage component to core (#4997)
 - fix(core): fix linting (#4996)
 - chore(core): bump package to 0.0.168 (#4995)
 - feat(core): adds ability to define a dockerInsight link (#4994)
 - feat(core/presentation): Provide `hidePopover()` prop to popover contents component (#4992)
 - chore(core): bump package to 0.0.167 (#4991)
 - fix(core): avoid rendering wizard pages before they have registered (#4990)
 - feat(core/pipeline): Allow overriding of default timeout for "Script stage" from UI (#4989)
 - chore(*): Bump core and amazon (#4987)
 - feat(artifacts): Enable GCE deploy stage to deploy an artifact (#4986)
 - feat(core): allow markdown in stage-level comments (#4988)
 - chore(*): update to react 16.2 (#4984)
 - feat(core): support markdown in tooltips and popovers (#4985)
 - feat(core/triggers): More relevant info when selecting execution (#4983)
 - fix(triggers): Pass image as an artifact when using Docker trigger (#4968)
 - chore(core): bump package to 0.0.165 (#4982)
 - fix(core): omit colon if there is no server group sequence (#4976)
 - feat(rollback): Support rollback on failure for Rolling Red/Black (#4981)
 - feat(core/serverGroup): Use flexbox instead of bootstrap grid to layout running tasks (#4980)
 - fix(core/presentation): Increase specificity of flex-pull-right within flex-container-h (#4979)
 - feat(core/presentation): Add margin selector for flex-container-h layout (#4978)
 - fix(core/entityTag): Fix background colors of notification categories
 - fix(pipeline_templates): prevent [object Object] when object's defaultValue is provided (#4975)
 - chore(core): bump package to 0.0.164 (#4974)
 - fix(provider/ecs): Fixed issue with AZ not appearing in deploy parameters (#4961)
 - feat(core): expose spel evaluator in spinnaker/core (#4972)
 - feat(core): remove 'N/A' if no server group sequence exists (#4973)
 - fixed a spinner that was off (#4965)
 - fix(core/overrideRegistry): fix lint error
 - chore(core): bump package to 0.0.163
 - fix(core/overrideRegistry): Copy static class properties to Overridable component
 - chore(amazon): bump package to 0.0.80
 - chore(core): bump package to 0.0.162
 - fix(core/application): fix scrolling of applications list (#4966)
 - feat(core/serverGroup): Refactor to smaller components and make them Overridable (#4941)
 - feat(goats): goats
 - chore(kubernetes): bump package version (#4963)
 - fix(kubernetes): fix import (#4962)
 - fix(core): Fix dismissal of new server group template modal (#4954)
 - chore(core): Bump to 0.0.161 (#4960)
 - chore(amazon): Bump to 0.0.79 (#4959)
 - fix(core/pagerduty): Fix tests for new service interface (#4957)
 - chore(core/pagerDuty): Add status to IPagerDutyService (#4956)
 - fix(amazon/loadBalancer): Validate target group name when region/account changes (#4955)
 - chore(core): bump package to 0.0.160 (#4953)
 - feat(core): allow custom warning when users skip an execution window (#4952)
 - fix(provider/kubernetes): v1 namespace lookup (#4950) (#4951)
 - fix(provider/kubernetes): v1 namespace lookup (#4950)
 - feat(core): allow parameters to be hidden based on other params (#4948)
 - chore(amazon): Bump to 0.0.78
 - chore(core): bump to 0.0.159
 - chore(*): Prepare for upgrading to react 16 (#4947)
 - fix(provider/gce): Fix call to list backends (#4946)
 - feat(provider/kubernetes): link to resources next to YAML in deploy pipeline (#4945)
 - fix(provider/kubernetes): update stage's kind when controller's rawKind changes (#4940) (#4944)
 - fix(provider/kubernetes): update stage's kind when controller's rawKind changes (#4940)
 - fix(artifacts): Fix no expected artifacts message (#4943)
 - fix(core/search): Get rid of useless prompt, hide 'Recently Viewed' header if no history (#4937)
 - fix(provider/gce): Fix call to list backends
 - chore(amazon): Bump to 0.0.77
 - chore(amazon/serverGroup): Export server group config interfaces
 - fix(provider/kubernetes): deploy manifests array nests itself on save (#4921) (#4935)
 - fix(core): correct category on recently viewed analytics events (#4933)
 - fix(artifacts): Fix no expected artifacts message
 - fix(provider/kubernetes): Show warnings for dashes in app names (#4927)
 - fix(package): Manually bump lockfile entry for fsevents@^1.0.0 (#4934)
 - chore(core): bump package to 0.0.158
 - fix(core/instance): Fix instance details loading on first click
 - feat(core/notification): Add a quick one-off slack channel affordance (#4930)
 - chore(amazon): Bump to 0.0.76 (#4929)
 - chore(core): Bump to 0.0.157 (#4928)
 - fix(core/pipelines): force refresh pipeline configs after save completes (#4925)
 - feat(provider/kubernetes): delineate default artifacts in executions (#4924)
 - fix(core/instance): Update InstanceDetails when props change
 - Making sure styleguide renders in deck (#4922)
 - chore(core): add analytics to recently viewed items (#4920)
 - fix(provider/kubernetes): deploy manifests array nests itself on save (#4921)
 - feat(aws): allow setting the target type for target groups (#4915)
 - feat(amazon/loadBalancers): Disable create button until security groups are refreshed
 - feat(core/wizard): Support a waiting state for sections
 - fix(amazon/loadBalancers): Stop showing stale data in load balancer edit dialog (#4917)
 - chore(core): bump package to 0.0.156 (#4919)
 - feat(pubsub): specify run-as-user for pubsub triggers (#4918)
 - chore(core): bump package to 0.0.155 (#4914)
 - style(core): wrap Overridable expressions in parens (#4913)
 - fix(artifacts): removing artifact from 'produces artifacts' throws exception (#4911)
 - feat(core/overrideRegistry): Pass the original component to the overriding component as a prop (#4910)
 - style(core): postioned the entity notifications to the right (#4907)
 - feat(core/cluster): Allow task matchers to be configured from deck-customized (#4909)
 - feat(core/pipelines): allow user-friendly labels on pipeline parameters (#4912)
 - chore(settings): alphabetize features in settings.js
 - feat(core) add a help menu with links to docs and community (#4903)
 - chore(*): Update typescript and tslint dependencies to latest (#4905)
 - chore(core): Bump package to 0.0.154 (#4904)
 - refactor(core): Remove formsy; replace-ish with formik (#4902)
 - fix(provider/google): add resize stage timeout override (#4901)
 - chore(core): bump package to 0.0.153 (#4900)
 - fix(core/pipelines): force data refresh on pipeline creation (#4897)
 - style(core): replace nano spinners on inline buttons (#4898)
 - fix(core): allow grouping by time on cancelled executions (#4896)
 - feat(kubernetes): Surfacing timeout override for Deploy (Manifest) (#4871) (#4895)
 - style(all): Updated references to styleguide package (#4869)
 - Revert "fix(core/pipeline): Fix pipeline config - add stage that uses BaseProviderStageCtrl (#4885) (#4891)" (#4894)
 - fix(openstack): add missing semicolon (#4893)
 - chore(core): bump package to 0.0.152 (#4892)
 - fix(provider/openstack) - Do not replace security group application when editing. (#4886)
 - fix(core/reactShims): Fix first render when receiving props before $scope is set (#4890)
 - fix(core/pipeline): Fix pipeline config - add stage that uses BaseProviderStageCtrl (#4885) (#4891)
 - fix(core/pipeline): Fix pipeline config - add stage that uses BaseProviderStageCtrl (#4885)
 - fix(core/overrideRegistry): Use account name (not id) when choosing overriding component (#4889)
 - chore(amazon): bump package to 0.0.75 (#4888)
 - chore(core): bump package to 0.0.151 (#4887)
 - feat(core/executions): allow pipeline rerun with same parameters (#4872)
 - feat(core): update permissions help text to include warning about cache delay (#4883)
 - fix(core/presentation): Fix react modal wrapper (#4884)
 - chore(kayenta): bump kayenta module version (#4878)
 - fix(provider/kubernetes): add type to manifestController resolver funcs (#4882)
 - fix(provider/kubernetes): add type to manifestController resolver funcs (#4881)
 - fix(provider/kubernetes): delete action broken for all but server groups (#4879)
 - fix(provider/kubernetes): delete action broken for all but server groups (#4880)
 - refactor(core/presentation): Change how ReactModal works (#4874)
 - chore(kubernetes): bump package version (#4877)
 - fix(core/entityTag): Fix error message after successful entity tag updates (#4873)
 - fix(artifacts): show artifact execution config (#4870) (#4876)
 - fix(artifacts): show artifact execution config (#4870)
 - feat(kubernetes): Surfacing timeout override for Deploy (Manifest) (#4871)
 - fix(core/overrideRegistry): Handle Override component synchronously, when possible (#4868)
 - fix(core/healthCounts): Propagate css class to health counts in either case (#4867)
 - feat(core/angular): Improve error message when angular module dep is undefined. Add overload to $q for normalizing a real promise into a fake angularjs promise. (#4866)
 - fix(provider/kubernetes): exclude manifest stages from v1 (#4863)
 - fix(amazon): make security groups in server group details links (#4862)
 - chore(core): bump package to 0.0.150 (#4864)
 - feat(amazon/loadBalancer): Export react load balancer components from `@spinnaker/amazon` (#4865)
 - feat(pubsub): pubsub providers list pulled from settings (#4861)

### Echo  - be7fd9e...4a9f9c0
 - refactor(test): retrofitStubs available to other modules (#252)
 - reafactor(eventMonitor): create method for emitting metrics for overriding (#251)
 - feat(notifications): Adding an initial swabbie template for email (#247)
 - chore(docker): run as spinnaker (#249)
 - fix(logging): Remove unnecessary log output (#248)
 - feat(artifacts): Docker registry event artifact parser (#246)
 - feat(trigger): allow pubsub monitor to be overridden (#245)
 - refactor(pubsub): split into core and provider specific modules (#244)
 - fix(pubsub): use app default credentials when possible (#243)
 - feat(artifacts): Support artifacts filtering in Travis triggers (#240)
 - chore(build): Parallel gradle tasks (#242)
 - feat(email/slack): allow custom messages per notification (#241)
 - fix(stage notifications): consuming updated orca payload (#238) (#239)
 - fix(stage notifications): consuming updated orca payload (#238)
 - fix(pubsub): changing amazon subscription name (#237)
 - fix(sqs): worker startup conditional on enabled (#236)

### Fiat  - 2a7603f...bac75ed
 - chore(docker): alpine-specific adduser (#221)
 - chore(docker): run as spinnaker (#220)
 - chore(build): Gradle parallel tasks (#219)
 - create the right directory (#218)
 - fix(admin): Admin parameter in accounts not being passed properly. (#217)

### Front50  - 1a61c32...bed259c
 - refactor(s3) - checks for non empty string before using it as an endpoint (#308)
 - feat(log): log whenever an object load returns a stale value (#307)
 - chore(docker): alpine-specific adduser (#306)
 - fix(serviceAccounts): Delete service account front Fiat when deleted from Front50
 - chore(build): Gradle parallel tasks (#304)
 - refactor(pipelineTemplates): Use shared strings (#303)
 - fix(pipelineTemplates): Validate Pipelines with Templates (#302)

### Gate  - edd0ad3...18965e0
 - feat(authz): Add ability to disable Fiat session filter. (#534) (#544)
 - chore(docker): alpine-specific adduser (#524)
 - chore(docker): run as spinnaker (#523)
 - feat(core): expose skin on account model (#522)
 - fix(entitytags): Explicitly `toLowerCase` when looking up entity tags by id (#521)
 - fix(web): allow account details to be referenced in server group insight links (#520)
 - feat(canary-v2): Add kayenta config. (#519)
 - fix(credentials): Remove hystrix from `CredentialsService` (#518)
 - fix(credentials): Swap `LoadingCache` for `@Scheduled` (#516)
 - feat(core): add expand query param to execution history (#515)
 - feat(reservations): Support passing arbitrary reservation report filters (#514)
 - chore(build): Gradle parallel tasks (#513)
 - feature(pagerduty): Add pagerduty service (#512)
 - fix(provider/gce): Allow short queries with explicit flag (#511)

### Igor  - 3d9468a...2a9532c
 - fix(safeguards) exceeding upper poll threshold messages are errors (#264)
 - fix(travis/packagecloud): Fix escaping in the packagecloud regex (#260)
 - fix(travis): Fix a code path that didn't fetch logs when it should (#259)
 - chore(travis): Convert Groovy to Java
 - chore(igor): Update Spinnaker dependencies version to 0.147.0
 - fix(travis): Fix a regression that cause getBuilds() to be slow (#239)
 - fix(docker): Redis key migration: Only migrate V1 keys (#256)
 - fix(docker): Fix igor startup with dockerRegistry enabled (#258)
 - fix(docker): Fix igor startup with dockerRegistry disabled (#257)
 - refactor(safeguard): Setting default high value on item threshold (#254)
 - feat(docker): Fallback value for item circuit breaker in Docker poller (#255)
 - chore(docker): alpine-specific adduser (#253)
 - chore(docker): run as spinnaker (#252)
 - fix(polling): Non-reachable endpoint for fast forwarding a poller (#251)
 - fix(docker): Remove parallel stream on docker polling (#250)
 - fix(health): non fatal healthindicator
 - fix(polling): Regression causes send event logic to be inverted (#248)
 - feat(jenkins): extension point for http request interceptor
 - fix(docker): Correctly set fastForward flag at poll interval start (#246)
 - fix(docker): s/registry/repository (#245)
 - fix(docker): Fixing NPE when an image in list is null (#244)
 - chore(docker): Adding additional logging around docker monitor ops (#243)
 - fix(polling): Fix igor startup with no active ci accounts (#242)
 - fix(polling): Fast-forward flag correctly influencing send events (#241)
 - fix(docker): Fixing Groovyism: Defer problems to runtime, thats fine (#240)
 - feat(jenkins): Adding interface to supply custom OkHttpClient (#232)
 - fix(bitbucket): Fix NPE caused by non-network retrofit errors (#229)
 - refactor(jenkins): mostly cosmetic changes (#238)
 - chore(docker): Removing totally unreasonable log statement (#237)
 - fix(dynomite): Fixing dyno scan usage (#236)
 - refactor(docker): Incrementally migrate keys on each scan result (#235)
 - fix(polling): Correctly reset failed items gauge on success (#233)
 - chore(build): Gradle parallel tasks (#231)
 - feat(web): Adding admin reindex api (#226)
 - feat(docker): Redis key migration (#227)
 - feat(travis): Include artifacts in build event for completed builds (#223)
 - refactor(monitor): Add new item circuit breaker on polling monitors (#225)

### Kayenta  - fd99dfa
No Changes

### Orca  - d0ba41e...324eb37
 - fix(pipeline/expressions): reference trigger as map, not object (#2253) (#2258)
 - fix(provider/kubernetes): fix lookup of non-namespaced manis (#2249) (#2250)
 - fix(provider/kubernetes): use thread local yaml parser (#2198) (#2216)
 - fix(pipeline_template): Don't share snakeyaml instance across threads (#2193) (#2197)
 - fix(kubernetes) - fixes WaitForClusterDisable timeout for kubernetes during redblack (#2176) (#2179)
 - feat(provider/kubernetes): Await all manifests to become stable/fail (#2151)
 - feat(bake/manifest): propegate "overrides" value (#2148)
 - feat(clouddriver): wait for instances down when server group is disabled (#2124)
 - feat(pipelines): remove context/outputs/tasks on /pipelines?expand=false calls (#2137)
 - fix(stage): Allow startTimeExpiry to be a string (#2147)
 - fix(front50): Log an appropriate error if unable to fetch pipelines (#2146)
 - fix(front50): Wait for pipeline to be updated before completing stage (#2145)
 - refactor(core): Alter startTimeTtl -> exiry, fix serialization (#2143)
 - fix(ttl): Avoid a NPE if `ttl` is not specified on a deploy config (#2144)
 - fix(queue): reschedule child pipeline when parent canceled (#2139)
 - feat(script): allow the job used by script stage to be overriden (#2142)
 - chore(highlander): Instrument cluster-wide tasks. (#2141)
 - feat(ttl): Support for ttl'ing server groups (#2111)
 - fix(expressions): Better handling for composite spel (#2135)
 - feat(bake/manifest): bake manfest stage (#2128)
 - fix(queue): Upgrade keiko, keep deprecated attribute around (#2136)
 - fix(redis): Actually persist startTimeTtl (#2134)
 - feat(titus): preferSourceCapacity support (#2132)
 - refactor(expressions): Remove deprecated versioning code (#2131)
 - fix(queue): Override RunTask message ack timeout (#2129)
 - fix(queue): Pass original task status along for `CANCELED` (#2130)
 - fix(queue): `FAILED_CONTINUE` tasks should not halt stage execution (#2127)
 - fix(provider/kubernetes): cancel timed-out run job pods (#2093)
 - fix(core): spotted that RollbackClusterStage was parallelizing regions!
 - chore(kayenta): use StageGraphBuilder.append where it's simpler
 - chore(kayenta): made it easier to define a linear chain of synthetics
 - chore(kayenta): use new API for synthetic stages
 - revert(queue): `FAILED_CONTINUE` tasks should halt stage execution (#2126)
 - Fix reading of converted template header (#2064)
 - feat(artifacts): Orca sends an an artifact to clouddriver (#2121)
 - chore(queue): removed unused throttle time attribute
 - Revert "Revert "chore(*): Remove queue traffic shaping codebase""
 - fix(script): cancel Jenkins job associated with script if stage fails
 - chore(canary-v2): Add placeholder config to orca.yml. (#2119)
 - fix(core): trigger check for waiting pipelines even if pipeline is already complete
 - fix(orca): handle missing trigger json
 - fix(core): don't cancel waiting pipelines as well as starting them
 - chore(docker): alpine-specific adduser (#2117)
 - chore(docker): run as spinnaker (#2116)
 - fix(core): null-check statuses read from DB
 - fix(targets): resolve target ASG with multiple dynamic target stages
 - feat(clouddriver): make restoreMin only restore the min size (#2059)
 - fix(queue): `FAILED_CONTINUE` tasks should not halt stage execution (#2112)
 - refactor(pipelines): handle queueing pipelines with Keiko
 - feat(dry run): find image by cluster / tag run for real in dry run
 - cleanup(logging): rm unnecessary printStackTrace
 - feat(rolling push): respect specified order of terminations (#2108)
 - fix(clouddriver): determine target server group can be any ancestor
 - feat(provider/kubernetes): allow jobs/pods to fail (#2100)
 - feat(pipeline_template): min and max filters (#2107)
 - chore(canary-v2): Replace region with location. (#2106)
 - fix(after stages): handle stage with pre-planned after stages
 - chore(keiko): Upgrade Keiko
 - chore(*): Removing Dynomite (#2102)
 - Revert "chore(*): Remove queue traffic shaping codebase"
 - chore(*): Remove queue traffic shaping codebase
 - feat(webhooks): support webhook artifacts (#2096)
 - fix(after stages): make sure anything downstream of a no-op stage runs
 - fix(after stages): prevent after stages re-running
 - fix(docs): Fix minor typo in README file (#2092)
 - refactor(core): Change type to Instant (#2091)
 - fix(queue): Fix execution type based clouddriver routing (#2089)
 - fix(after stages): Allow for just-in-time planning of after stages
 - feat(execution): Adding start time TTLs to executions and stages
 - feat(orca-kayenta): move canaryExecutionRequest and application to top-level (#2084)
 - chore(tests): fix test that was slow and making real network connections to a host we don't own
 - chore(dependencies): upgrade kotlin
 - fix(dry run): Avoid potential NPE
 - fix(expressions): Apparently the `getMessage()` of an exception can be null
 - feat(metric) execution deserialization errors (#2080)
 - fix(triggers): Git triggers must have a non-null hash
 - fix(spel): Revert "fix(spel): Operate on evaluated stage in handlers. (#1972)" (#2075)
 - fix(pipeline-templates): Fix trigger merging when hydrating templates. (#2081)
 - fix(clouddriver): Fix NPE while searching for ancestor stage (#2037)
 - fix(resize): Support both 'asgName' and 'serverGroupName' (#2071)
 - fix(rollback): Ensure that rollback stages are injected correctly (#2072)
 - chore(debug): Log what went wrong if we fail to plan after stages (#2077)
 - refactor(persistence): Unify Jedis/Dynomite Repos (#2070)
 - fix(canary): fix canary stages
 - chore(tests): added an extra verification to on failure stage tests
 - feat(rollback): Support rollbacks for red/black when deploy fails (#2067)
 - fix(core): fix target server groups stages when running as synthetics
 - chore(*): redis delegate for activeExecutionMonitor (#2068)
 - fix(core): prevent stages evaluating target server groups twice
 - feat(canary-v2): Add kayenta config. (#2066)
 - chore(core) old pipeline cleanup via redis delegate (#2065)
 - chore(*): Wiring up kork redis/dynomite client factories (#2009)
 - feat(core): strip logs from runjob executions and add expand param (#2061)
 - fix(queue): requeue running tasks from CancelStage (#2060)
 - Kayenta tests (#2056)
 - refactor(after stages): Always dynamically build after stages
 - fix(kayenta): handle exceptions from Kayenta (#2058)
 - fix(canary-v2): Fix issue with serialization of duration string. (#2057)
 - refactor(kayenta): Use strongly typed classes to represent Kayenta stage config (#2040)
 - fix(triggers): Git trigger should have a hash property
 - fix(rollback): Guard against entity tags not being enabled (#2054)
 - fix(API): don't allow running tasks to be deleted
 - fix(API): don't allow running pipelines to be deleted
 - debug(scale down): added execution id to debug logging
 - fix(artifacts): Don't replace artifacts in trigger after resolution (#2042)
 - fix(artifacts): Fix deploying of specified image for gce (#2051)
 - feat(artifacts): Use artifacts for Bake->Deploy GCE stages (#2049)
 - feat(core) expose stage to getDynamicBackoffPeriod (#2047)
 - fix(trigger): better error message when triggering a disabled pipeline (#1933)
 - fix(echo): send different phase in notification when stage is skipped (#2046)
 - debug(scale down): add some logging to try to identify why scale down doesn't always happen
 - perf(*): Speeding up the build, fixing transient test errors (#2044)
 - fix(rollback): Better support for automatic rollbacks on RRB deploys (#2043)
 - fix(persistence): enable dynomite repository tests (#2041)
 - chore(provider/kubernetes): de-capitalize cache refresh type (#2038)
 - fix(titus): set default security groups for run job stage (#2036)
 - fix(core): Custom deserializer for old requisiteStageRefId (#2028)
 - refactor(metrics): Use an Atlas timer instead of gauges to track lag
 - fix(queue): NPE on RunTaskHandler on startTime (#2034)
 - chore(kotlin): Kotlin 1.2.30
 - refactor(tests): split up test helpers into multiple modules
 - feat(queue): Bump keiko to 2.5.0 (#2031)
 - feat(metrics): median queue lag
 - chore(tests): AssertJ instead of Hamkrest
 - feat(metrics): graph max lag
 - fix(metrics): get more accurate queue lag metric
 - Revert "feat(kayenta): Adopt Kayenta code for ongoing development"
 - add execution id to ExecutionSerializationException message (#2026)
 - chore(build): add debug flag to orca build (#2020)
 - feat(metrics): graph queue lag
 - feat(queue): Fill executor every poll cycle (#2023)
 - feat(kayenta): Adopt Kayenta code for ongoing development
 - refactor(artifacts): Move artifact binding to orca-core (#2021)
 - fix(core): Correctly handle artifact matching with multiple packages
 - fix(spel): Revert "fix(spel): Operate on evaluated stage in handlers. (#1972)" (#2019)
 - fix(tagging): NPE if image has no tags
 - feat(queue): upgrade keiko to 2.3.0 (#2015)
 - fix(orca-webhook): Disable JsonPath cache (#1853)
 - feat(clouddriver): Improved support for partially disabling a server group (#2014)
 - fix(core): Fix auth propagation from manual judgment stages (#2013)
 - fix(clouddriver): Getting task from clouddriver master when timed out attempts to replicas (#2011)
 - fix(provider/amazon): restore finding imageId from trigger properties (#2012)
 - fix(core): Only record original capacity for server group if it exists (#2010)
 - feat(persistence): Dynomite ExecutionRepository (#2006)
 - feat(aws): Support for pinning min=desired capacity on source server group (#1997)
 - fix(notifications): sending required fields to echo (#2007) (#2008)
 - fix(notifications): sending required fields to echo (#2007)
 - chore(bakery): remove baseOs and baseLabel enums (#2003)
 - fix(queue): Ensure that refId generation accounts for existing stages (#2004)
 - fix(artifacts): fix parsing buildInfo when artifacts missing (#2005)
 - fix(artifacts): fix unconverted artifacts on triggers (#2001) (#2002)
 - fix(artifacts): fix unconverted artifacts on triggers (#2001)
 - fix(stage): npe no longer thrown when stage name is not present (#1999) (#2000)
 - fix(stage): npe no longer thrown when stage name is not present (#1999)
 - fix(spel): Support new return types in SpEL expressions
 - fix(queue): search for attributes in root queue package (#1996)
 - fix(queue): Use a simpler queue serialization migrator (#1995)
 - fix(pipelines): improve error msg if running disabled pipeline (#1991)

### Rosco  - 266bf0d...f86b041
 - chore(packer) - upgrade packer binary to 1.2.2 (#244)
 - fix(bake/manifests): delete templates when request finishes (#246)
 - feat(bake/manifest): support helm `--set` overrides (#247)
 - feat(core): add shutdown hook to avoid killing running jobs (#245)
 - feat(artifacts): Update GCE bake to send image reference (#241)
 - feat(bake/manifests): Rudimentary bake manifest support (#243)
 - fix(bake): Fix broken AWS Windows bake script (#238)
 - chore(docker): update packer (#242)
 - chore(docker): alpine-specific adduser (#240)
 - chore(docker): run as spinnaker (#239)
 - chore(build): add debug flag to rosco build (#236)
