---
# layout: post
title:  "Dynamic content"
date:   2024-08-06 21:55:52 +0100
categories: html
---

<script>
   	function daysSince(start) {
		writePlural(elapsedDays(start), "day");
   	}

   	function weeksAndDaysSince(start) {
   		const totalDays = elapsedDays(start);
   		const weeks = Math.floor(totalDays / 7);
   		const days = totalDays - 7 * weeks;
   		if (weeks != 0) {
   			writePlural(weeks, "week");
			document.write(" ");
		}
		writePlural(days, "day");
   	}

   	function writePlural(count, unit) {
		document.write(`${count} ${unit}`);
		if (count != 1) {
			document.write("s");
		}
   	}

	function elapsedDays(start) {
	   	const startDate = Date.parse(start);
   		const now = Date.now();
   		return Math.floor((now - startDate) / 60 / 60 / 24 / 1000);
   	}

   	function daysSinceComplaint() {
   		daysSince("10 Jun 2024 GMT");
   	}
</script>

Can inline-Javascript work in a Jekyll-based scenario? <script>document.write("Yes")</script>

Regarding porting a number between providers, Ofcom says "customer should receive compensation if the process takes more than 24 hours". Porting a number within the same provider feels like it should be a much simpler process, and yet here we are, <script>weeksAndDaysSince("3 Jun 2024")</script> since I had working service.
It has been <script>weeksAndDaysSince("10 Jun 2024")</script> since I registered my complaint.


|Status| |
|-|-|
|__Calling__||
|Make a call|Works. Receiver sees the call as coming from my number|
|Receiving calls|The network doesn't attempt to connect the call, reporting *"That number has not been recognised"*|
|__SMS__||
|Send|Cannot send - *"Message delivery failure"*|
|Receive|Cannot receive|
|__Data__||
|Works||

|Progress| |
|-|-|
|Calls to support line|10+, totalling 5+ hours|
|Email to the CEO|No response|
|Signed-for letter to the CEO|No response|
|Closed support tickets|3+<br/>I can only count the tickets that have been identified to me. Presumably there are others that I was never given the identity of.|
|Open ticket|<script>daysSince("2 Aug 2024")</script> old, no signs of progress|


