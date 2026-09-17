# Off-boarding Check Script

In this doc, we'll go over a script I wrote at work that checks 3 things: Disabled users that have not been moved to an OU, disabled users in the disabled OU that still have security groups, and licenses that are still applied to disables users in Entra.

## The Problem

The help-desk has been getting busier and unfortunately have missed a couple steps in the off-boarding procedures. This can cause potential security gaps if users who may be disgruntled have access to anything in any shape or form. It also helps save the company money by confirming that users do not have licensees if they are not with the company, such as E5 and CoPilot licenses.




# The Script


<img width="852" height="656" alt="image" src="https://github.com/user-attachments/assets/7055c2aa-c3b6-4ddf-9625-71c02a160c8e" />


First we'll do some configuration.

We'll want to import the Active Directory Module, establish our DC since we are using on prem Domain Controllers, I also have a variable for excluded OU's and excluded users.




## Find Disables Users NOT in Disabled OU

<img width="986" height="799" alt="image" src="https://github.com/user-attachments/assets/fe4fa800-2288-45e3-a5eb-c73c26c67d20" />

We'll put disabled users into the $DisabledUsers variable and then put the ones that are not in the disabled OU, into $DisabledOutsideOU.

<img width="526" height="75" alt="image" src="https://github.com/user-attachments/assets/cbc738ad-3065-45d1-989e-dceb445280ef" />

This part finds all disabled users.

<img width="698" height="472" alt="image" src="https://github.com/user-attachments/assets/e411495c-381c-4ab5-9b66-15981a610510" />

This part takes all the users in $DisablesUsers and matches them to filters such as if they're in the $ExcludedOU and $ExcludedUsers

Then if takes those results and grabs certain fields like the display name, SAM account name, UPN, and distinguished name.




## Find Users in Disabled OU That Have Groups Memberships

<img width="894" height="647" alt="image" src="https://github.com/user-attachments/assets/00c513c8-aab0-45bc-b610-d7b63baf3712" />

<img width="525" height="154" alt="image" src="https://github.com/user-attachments/assets/9a720144-9a55-4417-b6c9-27fa117f599e" />

This part of the script creates a new variable, $DisabledWithGroups (but does not put anything into it yet), and searches for users in the disables OU.


<img width="878" height="467" alt="image" src="https://github.com/user-attachments/assets/f004a66b-934b-435a-8ff9-ff975c137709" />

Next, we have this part go through each of those users and excluded selected users and exclude the 'Domain Users' group, since that applies to all users.

Then, if the user has groups while in the disabled OU, it adds it to the $DisabledWithGroups variable and stores the groups and Display/Sam name.




## Find Disabled with Open Licenses

<img width="847" height="697" alt="image" src="https://github.com/user-attachments/assets/3442bd89-ab5e-4aae-91a5-a1d839dff4e9" />


Next, we have the part that uses MgGraph to find disbaled users with licenses.

This runs a Get-MgUser and adds all users who are disabled and have active licenses into $DisabledEntraUsers.




## Email Body

<img width="825" height="885" alt="image" src="https://github.com/user-attachments/assets/88888a28-818f-4b60-a961-7fad7271000b" />


Next, we have the body of the email. It puts the results of all the variables into plain text and put it in an easy to read format.

<img width="786" height="635" alt="image" src="https://github.com/user-attachments/assets/0e5fce03-b2ac-46e0-b2be-dbdcf2e2d8db" />


The last part is to specify the subject, format type and to and from address.

Then send the email as the selected email and disconnect from MgGraph.



## MgGraph

To use MgGraph, we first need a couple things. We need to register an application, create a certificate and get permissions of the user we want to send as:

<img width="664" height="412" alt="image" src="https://github.com/user-attachments/assets/36f6e8f4-821e-43c3-83af-e8015918173d" />

### Register Application

We'll go to: **Entra > App registrations > All apps > New registration**

Name and take note of the client/app ID and tenant ID:

<img width="1618" height="762" alt="image" src="https://github.com/user-attachments/assets/45d1edd2-5123-482b-997a-1595ededae40" />

### Create Certificate

We'll create our certificate:

<img width="1525" height="327" alt="image" src="https://github.com/user-attachments/assets/91a5345a-e35e-43fe-bd7e-60c3b88a9f98" />


Export our certificate, so we can then upload into Entra:

<img width="1506" height="295" alt="image" src="https://github.com/user-attachments/assets/39c2051d-1a7d-41a0-b448-e2d25c9092cf" />

In Entra, upload our certificate in the app registration.


### Grant API Permissions

Next, we'll add certain permissions. Our API will need to be able to send mail as a user, read licenses and read all users. We'll use application permissions.

<img width="1118" height="436" alt="image" src="https://github.com/user-attachments/assets/a6c1b926-1169-4053-9d8a-2575ebe907ff" />

*You'll also need to grant admin consent or get someone to grant admin consent*

Then you can go back into the script and verify that you have pasted the ID's into the script for when you use ``Connect-MgGraph``. Now the script can be authenticated with Entra.



## Results

After that, verify everything works and check out the results:

We got the email:

<img width="1222" height="796" alt="image" src="https://github.com/user-attachments/assets/5702957a-1379-4ddf-8557-aa3fc8ee6164" />

We can verify in AD that this user is disabled and in fact, NOT in the disabled OU:

<img width="1080" height="672" alt="image" src="https://github.com/user-attachments/assets/944b7d21-bd54-4759-912f-2bf8e09b915e" />

Next, we can look at users who are in the disabled users who have groups still:

<img width="1172" height="552" alt="image" src="https://github.com/user-attachments/assets/69327a35-a492-49df-a16e-46556a0cdd25" />


We can verify this as well:

<img width="1080" height="672" alt="image" src="https://github.com/user-attachments/assets/175af23a-8842-4dfd-92ea-ca53ec7be67e" />


Lastly in the email, we can see users who are disabled in Entra who still have licenses:

<img width="633" height="605" alt="image" src="https://github.com/user-attachments/assets/59ddbb16-9484-455f-a4b6-4eeff8fbe943" />

We can now check Entra and verify that this is the case:

<img width="890" height="835" alt="image" src="https://github.com/user-attachments/assets/4e72a1cb-5211-414c-99b7-7a7cbc97f34d" />


This saves the company money and closes security gaps within off-boarded users

