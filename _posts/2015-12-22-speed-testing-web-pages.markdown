---
layout: post
title:  "Speed Testing Web Pages"
date:   2015-12-22
categories: projects
image: /img/dev-tools.png
---

Sometimes there are rare cases in which the Chrome DevTools aren't quite what you need. I recently had to perform a micro-optimization that wasn't really measureable in the network tab.

##### The Problem

Before applying the fix I jotted down a few load times and, when I compared them to the post-fix times, the data didn't make any sense. The only way to see if my fix had made any impact at all was to collect dozens of load times and average them. While Chrome's network tab is quite detailed, it can't help me perform multiple benchmarks.

##### The Solution

Looking at some of the available options made my stomach turn. Optimizely wouldn't work because the test needed to be run locally, writing a benchmarking app with PhantomJS or Electron would take too long, and refreshing manually is for peasants. Then I realized all the data I needed could be accessed using the window.performance API. All I'd need to do was store the data somewhere and refresh x number of times. That, my friends, is how I came up with this little number:

{% highlight js %}
window.runPerformanceTest = function() {
	var testsToRun = localStorage.getItem('testsToRun');

	var testNumber = parseInt(localStorage.getItem('testNumber'));
	if (testNumber) {
		testNumber += 1;
		if (testNumber > testsToRun) {
			console.log('Test Complete');
			localStorage.setItem('runPerformanceTest', false);
			return false;
		}
	} else {
		testNumber = 1;
	}
	localStorage.setItem('testNumber', testNumber);
	console.log('Test ' + testNumber + '/' + testsToRun);

	var testData = JSON.parse(localStorage.getItem('testData'));
	if (testData) {
		testData.push(window.performance.timing);
	} else {
		testData = [];
		testData.push(window.performance.timing);
	}
	localStorage.setItem('testData', JSON.stringify(testData));
	location.reload();
};

window.resetPerformanceTest = function() {
	localStorage.removeItem('testNumber');
	localStorage.removeItem('testData');
	localStorage.removeItem('testsToRun');
};

window.startPerformanceTest = function() {
	window.resetPerformanceTest();
	var testsToRun = localStorage.getItem('testsToRun');
	if (!testsToRun) {
		testsToRun = window.prompt('Enter number of tests to run:', '10');
		if (testsToRun == null) return;
		localStorage.setItem('testsToRun', testsToRun);
	}
	localStorage.setItem('runPerformanceTest', true);
	location.reload();
};

$(window).load(function() {
	if (localStorage.getItem('runPerformanceTest') == 'true') {
		setTimeout(function() {
			window.runPerformanceTest();
		}, 500);
	}
});
{% endhighlight %}

The script waits until the page has fully loaded and then stashes the window.performance.timing object in localStorage. It will refresh the page and repeat testing until the desired number of tests have been run. The data can be collected from the key called "testData" inside DevTools > Resources > Local Storage.

The test can be started by running startPerformanceTest() from the console, but that's too difficult. Let's make a JavaScript bookmarklet that starts the test. Paste the following into a new bookmark URL:

{% highlight js %}
javascript:(function(){ window.startPerformanceTest(); })();
{% endhighlight %}

Clicking the bookmarklet will bring up the prompt and begin testing.

##### The Result

I ran 50 tests pre and post-fix to get reliable data. After converting the JSON to CSV at convertcsv.com, I pasted it into a Google Sheet and viewed the two sets on a graph. The result: a 5% speed boost from the fix.
