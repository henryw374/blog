


the main thing on the horizon is the new Temporal api for js. 
when that's in browsers, I think that'd be preferred over the current js 
java.time impl (js-joda). not sure how that'll play out but hopefully tick 
protocols could be extended to that so it'll 'just work' for the majority 
of the existing tick api.

cljc.java-time could maybe be made to work for Temporal but even if it 
could, it looks like a lot more work ... so less likely to happen
