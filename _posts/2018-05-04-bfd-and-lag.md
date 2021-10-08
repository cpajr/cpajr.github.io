---
title: 'Combining BFD and LAG'
---

As is common with most network implementations, you try to leverage elements that will provide you the best design.  This include using link aggregation (LAG) or, in extreme cases, bi-directional forwarding detection (BFD).  In my network we implement BFD to accommodate needs to converge in less than 50ms.  So, it made complete sense to use both LAG and BFD together.  I'm sure some of you are guessing that this combination did not work well.

In a particular use case, before placing production traffic on it, we completed some testing and were finding some unexpected results.  Specifically, it appeared that BFD was reacting more quickly than LACP.  Makes complete sense -- per our configuration, BFD will react within 30ms; whereas, LACP will generally react within 100ms or more.  With BFD triggering our protocols, it would cause the entire link to be unusable, and would remain that way for almost 30 seconds or more in some cases.  In my network, 30 seconds is too long.  What further confirmed this discovery, we changed the multiplier on BFD from 3 to 5 and found that it no longer affected traffic with a failure of a link in the LAG.  Basically, we pushed out the timers on BFD to be longer than LACP.  

After doing some investigation, including a discussion with TAC, I found the following gem in my vendor’s software release notes:

>BFD…is not supported on ports that are in an active-active LAG group.  This configuration is not blocked; however, it is unsupported and will not function correctly.  

Well, gee, thanks for the heads up.  Needless to say, our design has been changed to exlude BFD across a LAG.  
