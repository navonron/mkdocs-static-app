## How to identify that the issue is with iBI Service?

All iBI service errors should have the name `daas-<GUID>` in their name, where `GUID` is the "Instance Id" of the service.

Example Ticket: `Kafka Consumer daas-aa49fde3-947a-42da-a7ea-52109dd0d6bf has data issue on site iil`

We can see the the consumer group begins with daas-*GUID*, therefore it's an iBI service.

## How to fix this:
For the Lag to be fixed, we need to restart the problematic iBI Service:

1. Take the GUID from the ticket that's been opened (for example for the consumer group `daas-aa49fde3-947a-42da-a7ea-52109dd0d6bf` the GUID would be `aa49fde3-947a-42da-a7ea-52109dd0d6bf`).

3. Open iBI Management Studio on your PC and in the "Saved Collection" (1) tab search for "Servers List" (2)

	![](Consumer%20Lag%20BKM%20-%20iBI%201.jpeg)<br>

3. Click on the "Server List" (1) button and a new Query named "Server List" (2) with the value of `ibi directory --diagnostics` should appear. Click on "Execute" (3)

	![](Consumer%20Lag%20BKM%20-%20iBI%202.jpeg)<br>
	
4. A table with the "Results" will appear with the list of all the services (2). Use the search box to search for the GUID of the service from Step 1.
	![](Consumer%20Lag%20BKM%20-%20iBI%203.jpeg)<br>
	
5. After search for the GUID of the Service (1) you should see all it's details like Site (2), MachineName (3), InstanceId (4), InstanceName (5)

	![](Consumer%20Lag%20BKM%20-%20iBI%204.jpeg)<br>

6. Login to the `MachineName` from ASG and restart the service `InstanceName`.

7. The problem should be fixed and the ticket should be closed in the next 15 minutes.

If this does not resolve the issue, please contact the OnCall and the DevOps team.