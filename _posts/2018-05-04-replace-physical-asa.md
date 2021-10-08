---
title: 'Replacing Physical ASA in an HA Configuration'
---

Over a year ago, Cisco released a [field notice](https://www.cisco.com/c/en/us/support/docs/field-notices/642/fn64228.html) regarding a critical clock flaw in several of their ASA platforms.  Of course, I had several ASAs that were affected.

Finally, after almost a year of waiting, I received a pair of ASAs to replace in my environment.  I decided to complete this work instead of having my technicians complete the work.  I found several [posts](https://www.reddit.com/r/networking/comments/6971mk/advice_readding_a_failed_asa_member_to_ha_pair/) online that explain how to accomplish this.  I decided to not copy the entire configuration to the replacement pairs.  Instead, I just placed the appropriate failover configuration on each ASA and planned that the already active unit to copy the configuration over.  

Now, here is where I made my mistake.  I had the secondary ASA as _Active_, and would replace the primary first.  I pulled out the old ASA and replaced it with the new.  But, before turning on the new ASA, I did not connect the _FAILOVER_ link or any other connections.  The new primary ASA booted and showed itself as active.  It was only now that I connected the _FAILOVER_ link.  Once I completed this connection, the new primary ASA, with a blank configuration, pushed its information to the secondary (old) ASA.  Both ASAs were in an almost blank configuration. 

Thankfully, I had the original configuration available.  I dropped it into the primary (new) ASA and re-established connectivity.  

### Lesson Learned

My original intent and approach was correct except that I allowed the (new) primary ASA boot without the _FAILOVER_ link connected.  Had I done this I would not have had any problems.   
