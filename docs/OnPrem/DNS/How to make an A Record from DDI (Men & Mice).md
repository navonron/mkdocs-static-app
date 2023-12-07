
In this page there is an explanation on how to create an A record for a LTM pool (can also be used for GTM pools).

## First Phase

1. Log into [https://ddi.intel.com](https://ddi.intel.com/)
2. Click on → New DDI request → Internal.

	![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%201.png)

3. Specify the DNS record name e.g. `libi-kafka.iil.intel.com` (.iil.intel.com being the DNS zone of the address)

	![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%202.png)

4. Setting requests fields
	1. Type: Alias (A or AAAA)
	2. Address Type: Domain Member
	3. IP Address i.e. the VIP IP Address in HighWire e.g 10.184.69.151
	4. Corp Domain: the site of the A Record e.g [GER.corp.intel.com](http://ger.corp.intel.com/) (Europe)

5. A warning will pop saying that the IP address is used today (which is ok since it's linked to HighWire generates A record), click on "Continue".

	![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%203.png)

6. A new request will be created with:
	1. PTR deletion of the HighWire generated record.
	2. CNAME with your desired A record name.
	3. TXT of the Corp Domain you chose.
	![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%207.png)
1. Click on "Submit Request" and wait for the request to be applied.

## Second Phase

After you request has been approve (might take a while because a manual approval is needed) follow steps 1-3 again, then click  on 'continue'.

A windows will pop with the current TXT record you created in the first ticket.

1. Click on the line of the TXT record and click on Modify.
2. When the below popup pops click on "Convert"

![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%204.png)

After  clicking on "Convert", A new request will be shown with 3 lines:

1. Delete the "ger" TXT record.
2. Delete the CNAME record with your desired A record name
3. Create line with your desired record name as A record.

	![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%205.png)

4. Click on Submit to submit the ticket and wait for it to be approved (or send another email to the DNS team same as above.
5. Done!
6. Now you should see the A record appear in HighWire like so:

## Third Phase

Same as phase 1-2, the DNS team will have to approve you request manually, which might take time again.

After the request is approved and fully processed, go to HighWire and your DNS record in the Load Balancer should be updated to your desired name:

![](How%20to%20make%20an%20A%20Record%20from%20DDI%20(Men%20&%20Mice)%206.png)
