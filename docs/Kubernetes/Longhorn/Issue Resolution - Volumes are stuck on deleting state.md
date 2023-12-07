This issue was observed in the GER Rancher clusters ("ger-pda-cluster") in the Longhorn app installation.

The issue began after an upgrade to the Longhorn app from version 1.2.0 to version 1.2.3 .

The DevOps team faced issues while trying to delete the Kafka cluster's volumes.

As we can see in the following screenshot all of the Longhorn volume after the upgrade were stuck in a 'Deleting' state for a long period of time.

Prior attempts to fix the issue included:

1. Deleting the entire all the apps and deployments that were connected to Longhorn.
2. Deleting the link between the Rancher PV and PVC and the reference to Longhorn.
3. Deleting the PV using kubectl (with force flags).


After all the above attempts the Longhorn volumes still stuck in the same "deleting" state as can be seen below:

![](Issue%20Resolution%20-%20Volumes%20are%20stuck%20on%20deleting%20state%201.png)

The Longhorn support was provided with a support bundle per their request.

They found out that these volumes were created with the older Longhorn engine image (v1.2.0) and will not be able to finish the deletion process untill the old version(v1.2.0) is redeployed.

**Longhorn-manager example log**
```
2022-02-15T15:40:28.265097591Z time="2022-02-15T15:40:28Z" level=warning msg="Error syncing Longhorn engine" controller=longhorn-engine engine=longhorn-system/pvc-5d3cb765-024c-474d-af10-65d00e36674c-e-e47665f3 error="fail to sync engine for longhorn-system/pvc-5d3cb765-024c-474d-af10-65d00e36674c-e-e47665f3: unable to get engine image longhornio/longhorn-engine:v1.2.0: engineimage.longhorn.io \"ei-0f7c4304\" not found" node=iapp906
```

In order to provide necessary logs for Longhorn, you need the click Generate Support Bundle at the bottom left corner of the Longhorn UI page. 

A zip file will be downloaded and there under the logs directory look for a directory called longhorn-manager-XXXX there you will find a log file that contain the log message with missing Longhorn engine version of those volumes.

The old engine image used by these volumes was upgrade to v1.2.3 therefore was replaced by the new version and cease to exist.

The Longhorn team's instructions were:

1. Go to the Longhorn web UI (requires logging in the Rancher).
2. To redeploy an old engine image (in this case` longhornio/longhorn-engine:v1.2.0`)
    1. In the navigation bar at the top of the page click on: Settings → Engine Image → Deploy Engine Image → type the required version → click OK
3. Wait for the volume deletion process to be done.
4. Then delete the old engine image again to prevent issues if it's not needed.

![](Issue%20Resolution%20-%20Volumes%20are%20stuck%20on%20deleting%20state%202.png)

Eventually the volumes were deleted quickly.

However, this matter means that in future upgrade this issue may occur again with the current volumes, and the same instructions will be required again for the volumes deletion process.