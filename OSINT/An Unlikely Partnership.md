# An Unlikely Partnership
Category: OSINT
Points: 100
Solves: 511

### Solution
```It appears that the Long Island Subway Authority (LISA) has made a strategic business partnership with a surprise influencer! See if you can figure out who.```

After solving [Hip With the Youth](https://github.com/EnchLolz/UIUCTF-2024/blob/main/OSINT/Hip%20WIth%20the%20Youth.md), we should be on LISA's Threads account.

![LISA Threads Account](/images/AnUnlikelyPartnerShipLISA_LinkedinLink.png)

Next, there is a LinkedIn account to navigate to.

![LISA LinkedIn Account](/images/AnUnlikelyPartnerShipLISA_LinkedIn_account.png)

After perusing through its profile, the about me, activity, and experience sections, we reach the skills section.

![LISA Skills](/images/AnUnlikelyPartnerShipLISA_skills.png)

Under "Transportation" there is an endorsement.

![LISA Endorsement](/images/AnUnlikelyPartnerShipLISA_endorsements.png)

Clicking on the endorsement shows the LinkedIn profile for UIUC Chan, the unlikely partner.

![UIUC Chan Account](/images/AnUnlikelyPartnerShipUIUC-Chan_LinkedIn_account.png)

Opening up UIUC Chan's profile, we get the flag in her bio.

### Unintended Solution

When trying to solve `Hip With the Youth`, I googled "Long Island Subway Authority" to see if any of LISA's accounts would show up.

![The Google Search](/images/AnUnlikelyPartnerShipCheeseSol.png)

Only 2 results came back, but the 2nd was the LinkedIn profile for UIUC Chan. I had just accidentally solved the second challenge before solving the first.

### Flag

```uiuctf{0M160D_U1UCCH4N_15_MY_F4V0r173_129301}```