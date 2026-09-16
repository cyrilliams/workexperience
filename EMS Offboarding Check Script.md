# EMS Off-boarding Check Script

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




















