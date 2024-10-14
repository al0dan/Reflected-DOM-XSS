# Reflected-DOM-XSS
hello this writeup is for reflected DOM XSS!!
by: al0dan

so let's get started, as we can see looks like you every day website
![Alt text](image1.png)

lets hit it with a alpha-numeric search string i will use "al0dan"

![Alt text](image2.png)
our search string is shown back, fully, on the web page now we need to find where in the dom our search string appears and how it's processed this will help us figure out how to solve the lab

right-click on the search string that came back and pick "inspect" from the menu. this will open the dom browser with the search string highlighted

![Alt text](image.png)

`<script src="/resources/js/searchResults.js"></script>
<script>search('search-results')</script>
<section class="blog-header">
    <h1>0 search results for 'al0dan'</h1>
    <hr>
</section>
`

see our reflected search string in the `<h1>` tag? also notice the `<script>` tags above it these two scripts are handling our search string and then returning it inside the `<h1>` tag but how exactly are they doing this?

the `src=/resources/js/searchResults.js` is an external javascript file think of it like sending your laundry to a dry cleaner instead of washing it at home the web page is processing our string off-site and then sending the result back to the page using `search('search-results')`

but we still don't know exactly how our string is being processed. to figure that out go to the 'network' tab in your dev tools

![Alt text](image3.png)

refresh the blog page to view all the different connections the page is making. this will show you everything being loaded, including scripts, images, and external resources

![Alt text](image4.png)

remember the two <script> tags from earlier? the first one <script src="/resources/js/searchResults.js"></script>, tells us where to start looking

click on searchResults.js a new window will open with several tabs you'll start on the 'headers' tab but we need to check the 'response'

click on 'response'

`
function search(path) {
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
};
if (this.readyState== 4 && this.status == 200) { eval('var searchResultsObj = this.responseText); displaySearchResults (searchResultsObj);
}
xhr.open("GET", path+window.location.search);
xhr.send();
function
displaySearchResults (searchResultsObj) {
var blogHeader = document.getElementsByClassName("blog-header")[0]; var blogList = document.getElementsByClassName("blog-list")[0]; var searchTerm = searchResultsObj.searchTerm
var searchResults = searchResultsObj.results
`
see the processes of our search string

the javascript is simple and easy

the object xhr was created, and an HTTP `GET` request was sent to:

`("GET", path + window.location.search)`

or

`('search-results' + '?search=al0dan')`


![Alt text](image5.png)


clicking on search-results?search=al00dan and checking the 'response' tab you'll see the response in JSON format our results are also enclosed in double quotes


![Alt text](image6.png)

we need to experiment with our search string to see how different characters affect our json response and now is the time to use burp sutie

![Alt text](image7.png)

here we can see both the request and the response from search-results?search=al0dan in burp sutie

from this point right-click inside the request box and select ‘send to repeater’ then go to the repeater tab

on the left side of burp repeater, you can modify your search string to attempt to 'break out' of the response since our searchTerm results are in double quotes we will start by adding a single double quote to the end of our search string along with our alert() function
`al0dan"-alert(1)`
see what happends

![Alt text](image8.png)

the backslash that appears is an escape character used to indicate to ignore the next character not allowing it from terminating the searchTerm

as you can see the double quote is still returned on the page

![Alt text](image9.png)

what if we add our own escape character before the double quote like this

al0dan\"-alert(1)

see how our search string is processed in burp sutie

![Alt text](image10.png)

now our double quote has terminated our search string and our -alert(1) has been successfully broken out!

it’s still not a valid payload because we have the closing double quote and curly brace meant to close the javascript object

to create a usable payload we need to manually close the javascript object by adding a closing curly brace } to our payload then we can comment out everything following it with //

`al0dan\"-alert(1)}//`

enter this payload into burp repeater and hit send and see what happends

![Alt text](image11.png)

woohoo as you can see at worked thanks for reading

![Alt text](image12.png)
